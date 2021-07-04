### 3. ros publishers

- ROS publishing can be either synchronous or asynchronous:
  - Synchronous publishing means that a publisher will attempt to  publish to a topic but may be blocked if that topic is being published  to by a different publisher. In this situation, the second publisher is  blocked until the first publisher has serialized all messages to a  buffer and the buffer has written the messages to each of the topic's  subscribers. This is the default behavior of a `rospy.Publisher` if the `queue_size` parameter is not used or set to `None`.
  - Asynchronous publishing means that a publisher can store messages in a queue until the messages can be sent. If the number of messages  published exceeds the size of the queue, the oldest messages are  dropped. The queue size can be set using the `queue_size` parameter.

### 5. simple mover: the code

```python
#!/usr/bin/env python

import math
import rospy
from std_msgs.msg import Float64

def mover():
    pub_j1 = rospy.Publisher('/simple_arm/joint_1_position_controller/command',
                             Float64, queue_size=10)
    pub_j2 = rospy.Publisher('/simple_arm/joint_2_position_controller/command',
                             Float64, queue_size=10)
    rospy.init_node('arm_mover')
    rate = rospy.Rate(10)
    start_time = 0

    while not start_time:
        start_time = rospy.Time.now().to_sec()

    while not rospy.is_shutdown():
        elapsed = rospy.Time.now().to_sec() - start_time
        pub_j1.publish(math.sin(2*math.pi*0.1*elapsed)*(math.pi/2))
        pub_j2.publish(math.sin(2*math.pi*0.1*elapsed)*(math.pi/2))
        rate.sleep()

if __name__ == '__main__':
    try:
        mover()
    except rospy.ROSInterruptException:
        pass
```

- When using ROS with simulated time (as we are doing here), `rospy.Time.now()` will initially return 0, until the first message has been received on the `/clock` topic. This is why `start_time` is set and polled continuously until a nonzero value is returned (more information [here](http://wiki.ros.org/rospy/Overview/Time)). 
- If the name variable is set to “**main**”, indicating that this script is being executed directly, the `mover()` function will be called. The try/except blocks here are significant as rospy uses exceptions extensively. 

### 6. ROS services

```python
service = rospy.Service('service_name', serviceClassName, handler)
```

- 使う側：

  ```python
  service_proxy = rospy.ServiceProxy('service_name', serviceClassName)
  
  msg = serviceClassNameRequest()
  #update msg attributes here to have correct data
  response = service_proxy(msg)
  ```

### 7. arm mover

- Like `simple_mover`, it is responsible for commanding the arm  to move. However, instead of simply commanding the arm to follow a  predetermined trajectory, the `arm_mover` node **provides** the service `move_arm`, which **allows other nodes** in the system to **send** `movement_commands`.

- ```python
  #!/usr/bin/env python
  
  import math
  import rospy
  from std_msgs.msg import Float64
  from sensor_msgs.msg import JointState
  from simple_arm.srv import *
  
  def at_goal(pos_j1, goal_j1, pos_j2, goal_j2):
      tolerance = .05
      result = abs(pos_j1 - goal_j1) <= abs(tolerance)
      result = result and abs(pos_j2 - goal_j2) <= abs(tolerance)
      return result
  
  def clamp_at_boundaries(requested_j1, requested_j2):
      clamped_j1 = requested_j1
      clamped_j2 = requested_j2
  
      min_j1 = rospy.get_param('~min_joint_1_angle', 0)
      max_j1 = rospy.get_param('~max_joint_1_angle', 2*math.pi)
      min_j2 = rospy.get_param('~min_joint_2_angle', 0)
      max_j2 = rospy.get_param('~max_joint_2_angle', 2*math.pi)
  
      if not min_j1 <= requested_j1 <= max_j1:
          clamped_j1 = min(max(requested_j1, min_j1), max_j1)
          rospy.logwarn('j1 is out of bounds, valid range (%s,%s), clamping to: %s',
                        min_j1, max_j1, clamped_j1)
  
      if not min_j2 <= requested_j2 <= max_j2:
          clamped_j2 = min(max(requested_j2, min_j2), max_j2)
          rospy.logwarn('j2 is out of bounds, valid range (%s,%s), clamping to: %s',
                        min_j2, max_j2, clamped_j2)
  
      return clamped_j1, clamped_j2
  
  def move_arm(pos_j1, pos_j2):
      time_elapsed = rospy.Time.now()
      j1_publisher.publish(pos_j1)
      j2_publisher.publish(pos_j2)
  
      while True:
          joint_state = rospy.wait_for_message('/simple_arm/joint_states', JointState)
          if at_goal(joint_state.position[0], pos_j1, joint_state.position[1], pos_j2):
              time_elapsed = joint_state.header.stamp - time_elapsed
              break
  
      return time_elapsed
  
  def handle_safe_move_request(req):
      rospy.loginfo('GoToPositionRequest Received - j1:%s, j2:%s',
                     req.joint_1, req.joint_2)
      clamp_j1, clamp_j2 = clamp_at_boundaries(req.joint_1, req.joint_2)
      time_elapsed = move_arm(clamp_j1, clamp_j2)
  
      return GoToPositionResponse(time_elapsed)
  
  def mover_service():
      rospy.init_node('arm_mover')
      service = rospy.Service('~safe_move', GoToPosition, handle_safe_move_request)
      rospy.spin()
  
  if __name__ == '__main__':
      j1_publisher = rospy.Publisher('/simple_arm/joint_1_position_controller/command',
                                     Float64, queue_size=10)
      j2_publisher = rospy.Publisher('/simple_arm/joint_2_position_controller/command',
                                     Float64, queue_size=10)
  
      try:
          mover_service()
      except rospy.ROSInterruptException:
          pass
  ```

  - `JointState` messages are published to the `/simple_arm/joint_states` topic, and are used for monitoring the position of the arm.
  - Note: `move_arm()` is blocking, and will not return until the arm  has finished its movement. Incoming messages cannot be processed, and no other useful work can be done in the python script while the arm is  performing it’s movement command.  While this poses no real problem for  this example, it is a practice that should generally be avoided. One  great way to avoid blocking the thread of execution would be to use [Action](http://wiki.ros.org/actionlib). Here’s some [informative documentation](http://wiki.ros.org/ROS/Patterns/Communication#Communication_via_Topics_vs_Services_vs_X) describing when it’s best to use a Topic versus a Service, versus an Action.

### 9. arm mover: launch and interact

```xml
  <!-- The arm mover node -->
  <node name="arm_mover" type="arm_mover" pkg="simple_arm">
    <rosparam>
      min_joint_1_angle: 0
      max_joint_1_angle: 1.57
      min_joint_2_angle: 0
      max_joint_2_angle: 1.0
    </rosparam>
  </node>
```

- `<rosparam>`を使ってみよう。

- `rosservice list`.

- To view the camera image stream, you can use the command `rqt_image_view` (you can learn more about rqt and the associated tools [here](http://wiki.ros.org/rqt)):

  ```bash
  $ rqt_image_view /rgb_camera/image_raw
  ```

- ```sh
  $ rosservice call /arm_mover/safe_move "joint_1: 1.57
  joint_2: 1.57"
  ```

### 11. look away

- The `look_away` node will subscribe to the `/rgb_camera/image_raw` topic, which has image data from the camera mounted on the end of the  robotic arm. Whenever the camera is pointed towards an uninteresting  image - in this case, an image with uniform color - the callback  function will move the arm to something more interesting.

- ```python
  def __init__(self):
      rospy.init_node('look_away')
      self.last_position = None
      self.arm_moving = False
  
      rospy.wait_for_message('/simple_arm/joint_states', JointState)
      rospy.wait_for_message('/rgb_camera/image_raw', Image)
  
      self.sub1 = rospy.Subscriber('/simple_arm/joint_states', 
                                   JointState, self.joint_states_callback)
      self.sub2 = rospy.Subscriber('/rgb_camera/image_raw', 
                                   Image, self.look_away_callback)
      self.safe_move = rospy.ServiceProxy('/arm_mover/safe_move', 
                                   GoToPosition)
      rospy.spin()
  ```

  - add `wait_for_message` to the look_away node before subscribing to the topics. This ensures that the callbacks are not called before the gazebo simulation (publishing these topics) is fully initialized.

### 12. look away: the code

```python
#!/usr/bin/env python

import math
import rospy
from sensor_msgs.msg import Image, JointState
from simple_arm.srv import *


class LookAway(object):
    def __init__(self):
        rospy.init_node('look_away')

        self.sub1 = rospy.Subscriber('/simple_arm/joint_states', 
                                     JointState, self.joint_states_callback)
        self.sub2 = rospy.Subscriber("rgb_camera/image_raw", 
                                     Image, self.look_away_callback)
        self.safe_move = rospy.ServiceProxy('/arm_mover/safe_move', 
                                     GoToPosition)

        self.last_position = None
        self.arm_moving = False

        rospy.spin()

    def uniform_image(self, image):
        return all(value == image[0] for value in image)

    def coord_equal(self, coord_1, coord_2):
        if coord_1 is None or coord_2 is None:
            return False
        tolerance = .0005
        result = abs(coord_1[0] - coord_2[0]) <= abs(tolerance)
        result = result and abs(coord_1[1] - coord_2[1]) <= abs(tolerance)
        return result

    def joint_states_callback(self, data):
        if self.coord_equal(data.position, self.last_position):
            self.arm_moving = False
        else:
            self.last_position = data.position
            self.arm_moving = True

    def look_away_callback(self, data):
        if not self.arm_moving and self.uniform_image(data.data):
            try:
                rospy.wait_for_service('/arm_mover/safe_move')
                msg = GoToPositionRequest()
                msg.joint_1 = 1.57
                msg.joint_2 = 1.57
                response = self.safe_move(msg)

                rospy.logwarn("Camera detecting uniform image. \
                               Elapsed time to look at something nicer:\n%s", 
                               response)

            except rospy.ServiceException, e:
                rospy.logwarn("Service call failed: %s", e)



if __name__ == '__main__':
    try: 
        LookAway()
    except rospy.ROSInterruptException:
        pass
```

- The first subscriber, `self.sub1`, subscribes to the `/simple_arm/joint_states` topic. The node is written to check the camera only when the arm is not moving, and by subscribing to `/simple_arm/joint_states`, changes in the position of the arm can be tracked.

### 14. logging

```python
rospy.logdebug(...)
rospy.loginfo(...)
rospy.logwarn(...)
rospy.logerr(...)
rospy.logfatal(...)
```

|          | Debug | Info | Warn | Error | Fatal |
| :------: | :---: | :--: | :--: | :---: | ----- |
|  stdout  |       |  X   |      |       |       |
|  stderr  |       |      |  X   |   X   | X     |
| log file |   X   |  X   |  X   |   X   | X     |
| /rosout  |       |  X   |  X   |   X   | X     |

- `rostopic echo /rosout | grep insert_search_expression_here > path_to_output/output.txt`

- For example, if you'd like to allow logdebug messages to be written to `/rosout`, that can be done as follows:

  ```python
  rospy.init_node('my_node', log_level=rospy.DEBUG)
  ```

- launch option: 

  |            | `stdout` | `stderr`       |
  | ---------- | -------- | -------------- |
  | `"screen"` | screen   | screen         |
  | `"log"`    | log      | screen and log |


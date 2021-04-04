#### Do your curvature values make sense?

![](img/screen-shot-2017-01-28-at-5.06.11-pm.png)

- If you're reporting 10km or 0.1km, you know there might be something wrong with your calculations.

#### Tracking

```python
# Define a class to receive the characteristics of each line detection
class Line():
    def __init__(self):
        # was the line detected in the last iteration?
        self.detected = False  
        # x values of the last n fits of the line
        self.recent_xfitted = [] 
        #average x values of the fitted line over the last n iterations
        self.bestx = None     
        #polynomial coefficients averaged over the last n iterations
        self.best_fit = None  
        #polynomial coefficients for the most recent fit
        self.current_fit = [np.array([False])]  
        #radius of curvature of the line in some units
        self.radius_of_curvature = None 
        #distance in meters of vehicle center from the line
        self.line_base_pos = None 
        #difference in fit coefficients between last and new fits
        self.diffs = np.array([0,0,0], dtype='float') 
        #x values for detected line pixels
        self.allx = None  
        #y values for detected line pixels
        self.ally = None  
```

- 履歴を記録。

#### Sanity Check

- To confirm that your detected lane lines are real, you might consider:
  - Checking that they have similar curvature.
  - Checking that they are separated by approximately the right distance horizontally.
  - Checking that they are roughly parallel.

#### Look-ahead filter

- For example, if you fit a polynomial, then for each y position, you have an x position that represents the lane center from the last frame.
  - Search for the new line within +/- some margin around the old line center.

#### Reset

- If your sanity checks reveal that the lane lines you've detected are problematic for some reason, you can simply assume it was a **bad or difficult frame** of video, **retain the previous positions from the frame prior and step to the next frame to search again**.
  - If you **lose the lines for several frames** in a row, you should probably start searching from scratch using a histogram and sliding window, or another method, to re-establish your measurement.

#### Smoothing

- Even when everything is working, your line detections will jump around from frame to frame a bit and it can be preferable to smooth over the last *n* frames of video to obtain a cleaner result.
- Each time you get a new high-confidence measurement, you can append it to the list of recent measurements and then take an average over *n* past measurements to obtain the lane position you want to draw onto the image.

#### Drawing

- warp back:

  ```python
  # Create an image to draw the lines on
  warp_zero = np.zeros_like(warped).astype(np.uint8)
  color_warp = np.dstack((warp_zero, warp_zero, warp_zero))
  
  # Recast the x and y points into usable format for cv2.fillPoly()
  pts_left = np.array([np.transpose(np.vstack([left_fitx, ploty]))])
  pts_right = np.array([np.flipud(np.transpose(np.vstack([right_fitx, ploty])))])
  pts = np.hstack((pts_left, pts_right))
  
  # Draw the lane onto the warped blank image
  cv2.fillPoly(color_warp, np.int_([pts]), (0,255, 0))
  
  # Warp the blank back to original image space using inverse perspective matrix (Minv)
  newwarp = cv2.warpPerspective(color_warp, Minv, (image.shape[1], image.shape[0])) 
  # Combine the result with the original image
  result = cv2.addWeighted(undist, 1, newwarp, 0.3, 0)
  plt.imshow(result)
  ```

![](img/lane-drawn.jpg)


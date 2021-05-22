### 14. cropping images in keras

```python
from keras.models import Sequential, Model
from keras.layers import Cropping2D
import cv2

# set up cropping2D layer
model = Sequential()
model.add(Cropping2D(cropping=((50,20), (0,0)), input_shape=(160,320,3)))
...
```

- The example above crops:
  - 50 rows pixels from the top of the image
  - 20 rows pixels from the bottom of the image
  - 0 columns of pixels from the left of the image
  - 0 columns of pixels from the right of the image
- Keras provides the [Cropping2D layer](https://keras.io/layers/convolutional/#cropping2d) for image cropping within the model. **This is relatively fast, because  the model is parallelized on the GPU, so many images are cropped  simultaneously. By contrast, image cropping outside the model on the CPU is relatively slow**. Also, by adding the cropping layer, the model will automatically crop the input images when making predictions in `drive.py`. 

### 18. generators

- A generator is like a [coroutine](https://en.wikipedia.org/wiki/Coroutine), a process that can run separately from another main routine, which makes it a useful Python function. Instead of using `return`, the generator uses `yield`, which still returns the desired output values but saves the current  values of all the generator's variables. When the generator is called a  second time it re-starts right after the `yield` statement, with all its variables set to the same values as before.

```python
import os
import csv

samples = []
with open('./driving_log.csv') as csvfile:
    reader = csv.reader(csvfile)
    for line in reader:
        samples.append(line)

from sklearn.model_selection import train_test_split
train_samples, validation_samples = train_test_split(samples, test_size=0.2)

import cv2
import numpy as np
import sklearn

def generator(samples, batch_size=32):
    num_samples = len(samples)
    while 1: # Loop forever so the generator never terminates
        shuffle(samples)
        for offset in range(0, num_samples, batch_size):
            batch_samples = samples[offset:offset+batch_size]

            images = []
            angles = []
            for batch_sample in batch_samples:
                name = './IMG/'+batch_sample[0].split('/')[-1]
                center_image = cv2.imread(name)
                center_angle = float(batch_sample[3])
                images.append(center_image)
                angles.append(center_angle)

            # trim image to only see section with road
            X_train = np.array(images)
            y_train = np.array(angles)
            yield sklearn.utils.shuffle(X_train, y_train)

# Set our batch size
batch_size=32

# compile and train the model using the generator function
train_generator = generator(train_samples, batch_size=batch_size)
validation_generator = generator(validation_samples, batch_size=batch_size)

ch, row, col = 3, 80, 320  # Trimmed image format

model = Sequential()
# Preprocess incoming data, centered around zero with small standard deviation 
model.add(Lambda(lambda x: x/127.5 - 1.,
        input_shape=(ch, row, col),
        output_shape=(ch, row, col)))
model.add(... finish defining the rest of your model architecture here ...)

model.compile(loss='mse', optimizer='adam')
model.fit_generator(train_generator, /
            steps_per_epoch=ceil(len(train_samples)/batch_size), /
            validation_data=validation_generator, /
            validation_steps=ceil(len(validation_samples)/batch_size), /
            epochs=5, verbose=1)
```
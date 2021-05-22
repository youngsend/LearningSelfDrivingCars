### 6. kalman filter intuition

![](img/2021-05-09-20-50-47-kalman-filter.png)

![](img/2021-05-14-19-25-53-kalman-filter.png)

- The K matrix, often called the Kalman filter gain, **combines** the **uncertainty of where we think we are** P′ with the **uncertainty of our sensor measurement** R.
  - If our sensor measurements are very uncertain (R is high relative to  P'), then the Kalman filter will give more weight to where we think we  are: x′. 
  - If where we think we are is uncertain (P' is high relative to R), the  Kalman filter will put more weight on the sensor measurement: z. 

### 10. process covariance matrix

![](img/2021-05-16-16-14-01-process-covariance-matrix.png)

![](img/2021-05-16-16-17-15-process-covariance-matrix.png)

![](img/2021-05-16-16-19-30-process-covariance-matrix.png)

![](img/2021-05-16-16-20-38-process-covariance-matrix.png)

### 15. radar measurements

![](img/2021-05-16-16-25-15-radar-measurement.png)

![](img/2021-05-16-16-26-28-radar-measurement-function.png)

### 17. extended kalman filter

![](img/2021-05-16-16-28-51-first-order-taylor-expansion.png)

### 19. jacobian matrix

![](img/2021-05-16-16-08-17-radar-jacobian.png)

### 21. ekf algorithm generalization

![](img/2021-05-16-17-14-19-kf-to-ekf.png)

### 22. sensor fusion general flow

![](img/2021-05-16-17-22-08-sensor-fusion-general-flow.png)

### 23. evaluating kf performance

![](img/2021-05-16-17-23-55-root-mean-squared-error.png)


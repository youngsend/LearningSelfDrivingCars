### 6. kalman filter intuition

![](img/2021-05-09-20-50-47-kalman-filter.png)

![](img/2021-05-14-19-25-53-kalman-filter.png)

- The K matrix, often called the Kalman filter gain, **combines** the **uncertainty of where we think we are** P′ with the **uncertainty of our sensor measurement** R.
  - If our sensor measurements are very uncertain (R is high relative to  P'), then the Kalman filter will give more weight to where we think we  are: x′. 
  - If where we think we are is uncertain (P' is high relative to R), the  Kalman filter will put more weight on the sensor measurement: z. 


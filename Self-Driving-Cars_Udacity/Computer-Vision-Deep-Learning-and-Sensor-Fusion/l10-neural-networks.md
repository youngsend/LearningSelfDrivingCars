### 12. Perceptrons

![](img/2021-04-09-23-25-04-perceptron.png)

### 14. Why "Neural Networks"?

![](img/2021-04-10-12-53-14-dendrites-axon.png)

### 20. Discrete vs Continuous

![](img/2021-04-10-18-13-02-discrete-to-continuous.png)

- gradient descentを使うために。ここの横軸はWx+bだ。

![](img/2021-04-10-18-17-51-step-to-sigmoid.png)

### 31. Perceptron vs Gradient descent

![](img/2021-04-10-21-12-59-perceptron-gradient-descent.png)

- perceptron algorithmだと、correctly classified pointの場合は何もしない。gradient descent algorithmだと、境界線を自分から遠くさせる。

### 35. Neural Network Architecture

![](img/2021-04-10-21-26-04-how-neural-network-built.png)

![](img/2021-04-10-21-29-21-how-neural-network-built.png)

![](img/2021-04-10-21-34-38-neural-network-architecture.png)

![](img/2021-04-10-21-35-52-neural-network-architecture.png)

![](img/2021-04-10-21-37-24-neural-network-architecture.png)

- multiclass classification model: catの確率、dogの確率、birdの確率。

![](img/2021-04-10-21-39-54-more-hidden-layers.png)

### 37. Multilayer Perceptrons

- It's possible to get the transpose of an array like so `arr.T`, but **for a 1D array, the transpose will return a row vector**. Instead, **use `arr[:,None]` to create a column vector**:

  ```python
  print(features)
  > array([ 0.49671415, -0.1382643 ,  0.64768854])
  
  print(features.T)
  > array([ 0.49671415, -0.1382643 ,  0.64768854])
  
  print(features[:, None])
  > array([[ 0.49671415],
         [-0.1382643 ],
         [ 0.64768854]])
  ```

- Alternatively, you can **create arrays with two dimensions**. Then, you can use `arr.T` to get the column vector.

  ```python
  np.array(features, ndmin=2)
  > array([[ 0.49671415, -0.1382643 ,  0.64768854]])
  
  np.array(features, ndmin=2).T
  > array([[ 0.49671415],
         [-0.1382643 ],
         [ 0.64768854]])
  ```

- vectorとmatrixの掛け算はnp.dotでできる。
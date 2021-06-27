### 13. twiddle

![](img/2021-06-27-22-49-14-pid-parameter-tuning.png)

```python
def twiddle(tol=0.2): 
    p = [0, 0, 0]
    dp = [1, 1, 1]
    robot = make_robot()
    x_trajectory, y_trajectory, best_err = run(robot, p)

    it = 0
    while sum(dp) > tol:
        print("Iteration {}, best error = {}".format(it, best_err))
        for i in range(len(p)):
            p[i] += dp[i]
            robot = make_robot()
            x_trajectory, y_trajectory, err = run(robot, p)

            if err < best_err:
                best_err = err
                dp[i] *= 1.1
            else:
                p[i] -= 2 * dp[i]
                robot = make_robot()
                x_trajectory, y_trajectory, err = run(robot, p)

                if err < best_err:
                    best_err = err
                    dp[i] *= 1.1
                else:
                    p[i] += dp[i]
                    dp[i] *= 0.9
        it += 1
    return p
```

- `run()`を実行する前に、`make_robot()`が必要。

### 17. controlに関する論文

- Model Predictive Control (MPC): https://arxiv.org/abs/1812.02071
- Reinforcement Learning-based: https://arxiv.org/abs/1810.12778
- Behavior Cloning: https://arxiv.org/abs/1812.03079


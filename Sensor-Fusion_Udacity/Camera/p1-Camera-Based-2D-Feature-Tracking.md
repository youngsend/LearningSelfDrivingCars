##  2. The Data Buffer

### MP.1 Data Buffer Optimization

- When `dataBuffer.size() < dataBufferSize`, push back current frame.
- When `dataBuffer.size() >= dataBufferSize`.
  - the newest element (current frame) is put in `dataBuffer[imgIndex % dataBufferSize]`.
  - the second newest element (if exists) is in `dataBuffer[imgIndex - 1 % dataBufferSize]`.

## 3. Keypoint Detection

### MP.2 Keypoint Detection

- I referred to the following document which list all detector types. 
  - https://docs.opencv.org/4.1.0/d0/d13/classcv_1_1Feature2D.html
- I just use the default arguments for their `create()` functions.
  - Since they have the same `detect(image, keypoints)` api, I put this call out of the if-else statements for creating detectors.
- `SHITOMASI`: ![](img/shi-tomasi-detector-2020-08-22-10-33-58.png)
- `HARRIS`: ![](img/harris-detector-2020-08-22-10-37-56.png)
- `FAST`: ![](img/fast-detector-2020-08-22-10-40-31.png)
- `BRISK`: ![](img/brisk-detector-2020-08-22-10-42-35.png)
- `ORB`: ![](img/orb-detector-2020-08-22-10-44-19.png)
- `AKAZE`: ![](img/akaze-detector-2020-08-22-10-45-42.png)
- `SIFT`: ![](img/sift-detector-2020-08-22-10-47-09.png)

### MP.3 Keypoint Removal

- I used the `cv::Keypoint::contains()` to judge whether to erase a keypoint.
- `SIFT` cropped: ![](img/sift-keypoints-cropped-2020-08-22-12-26-29.png)
- `ORB` cropped: ![](img/orb-keypoints-cropped-2020-08-22-12-28-28.png)

## 4. Descriptor Extraction & Matching

### MP.4 Keypoint Descriptors

- I referred to the same document as that in *Keypoint Detection*.
  - use the default argument values to create extractors.

### MP.5 Descriptor Matching

- `BFMatcher`.

  - If `DES_HOG`, use `cv::NORM_L2`; else, use `cv::NORM_HAMMING`.

- `FLANNBASED` matcher as well as `knnMatch`.

  - I referred to the example in https://docs.opencv.org/4.1.0/d5/d6f/tutorial_feature_flann_matcher.html

- FLANN matcher for binary descriptor.

  - I use `cv::flann::LshIndexParams` to create flann matcher (https://answers.opencv.org/question/59996/flann-error-in-opencv-3/):

    ```c++
    matcher = cv::makePtr<cv::FlannBasedMatcher>(
            cv::makePtr<cv::flann::LshIndexParams>(12,20, 2));
    ```

- `knnMatch`.

  - I first get `std::vector<std::vector<DMatch>>` with inner vector holding `k` best matches.
  - **If the size of inner vector is 0 or 1, do not perform distance ratio and skip it**.

### MP.6 Descriptor Distance Ratio

- For Lowe's ratio test, I referred to the same example as *Descriptor Matching* above.
- Set `k` to 2.
  - Compare distance `a` between descriptors in the best match and distance `b` between descriptors in the second best match.
  - If `a < ratio*b`, then the best match is a good match; if not, the best match is ambiguous and will not be reserved.

## 5. Performance Evaluation

### MP.7 Performance Evaluation 1: Keypoint number

- The keypoint numbers for ORB are all 500 because I set the `nfeatures` to 500 (which is default value). 

  | AKAZE | BRISK | FAST | HARRIS | ORB  | SHITOMASI | SIFT |
  | ----- | ----- | ---- | ------ | ---- | --------- | ---- |
  | 1351  | 2757  | 1824 | 339    | 500  | 1370      | 1437 |
  | 1327  | 2777  | 1832 | 286    | 500  | 1301      | 1371 |
  | 1311  | 2741  | 1810 | 349    | 500  | 1361      | 1381 |
  | 1351  | 2735  | 1817 | 356    | 500  | 1358      | 1336 |
  | 1360  | 2757  | 1793 | 521    | 500  | 1333      | 1303 |
  | 1347  | 2695  | 1796 | 2611   | 500  | 1284      | 1370 |
  | 1363  | 2715  | 1788 | 200    | 500  | 1322      | 1396 |
  | 1331  | 2628  | 1695 | 806    | 500  | 1366      | 1382 |
  | 1357  | 2639  | 1749 | 572    | 500  | 1389      | 1462 |
  | 1331  | 2672  | 1770 | 1471   | 500  | 1339      | 1422 |

- Use the first frame to compare the mean and standard deviation of keypoints' neighborhood sizes (diameter). 

  |      | AKAZE   | BRISK   | FAST | HARRIS | ORB     | SHITOMASI | SIFT    |
  | ---- | ------- | ------- | ---- | ------ | ------- | --------- | ------- |
  | mean | 7.43816 | 19.0813 | 7    | 6      | 53.9247 | 4         | 5.24563 |
  | std  | 3.19362 | 12.2054 | 0    | 0      | 23.3767 | 0         | 7.26947 |

  - `FAST, HARRIS, SHITOMASI` use fixed-size neighborhood.
  - `ORB`'s keypoint is the biggest! `BRISK`'s keypoint is also big.

### MP.8 Performance Evaluation 2: Match number

- There are 17 pairs. `AKAZE` detector can only be paired with `AKAZE` descriptor. `SIFT` detector can only be paired with `SIFT` descriptor.

  | AKAZE_AKAZE | BRISK_BRIEF | BRISK_FREAK | BRISK_ORB | FAST_BRIEF | FAST_FREAK | FAST_ORB | HARRIS_BRIEF | HARRIS_FREAK | HARRIS_ORB | ORB_BRIEF | ORB_FREAK | ORB_ORB | SHITOMASI_BRIEF | SHITOMASI_FREAK | SHITOMASI_ORB | SIFT_SIFT |
  | ----------- | ----------- | ----------- | --------- | ---------- | ---------- | -------- | ------------ | ------------ | ---------- | --------- | --------- | ------- | --------------- | --------------- | ------------- | --------- |
  | 128         | 138         | 114         | 94        | 88         | 64         | 96       | 25           | 22           | 20         | 37        | 39        | 40      | 96              | 66              | 86            | 82        |
  | 128         | 166         | 121         | 107       | 102        | 80         | 102      | 18           | 18           | 18         | 38        | 33        | 57      | 93              | 66              | 84            | 81        |
  | 125         | 129         | 113         | 88        | 88         | 65         | 87       | 29           | 21           | 32         | 37        | 37        | 49      | 92              | 64              | 87            | 86        |
  | 117         | 141         | 118         | 97        | 102        | 79         | 94       | 27           | 20           | 30         | 53        | 40        | 54      | 89              | 63              | 91            | 94        |
  | 121         | 148         | 103         | 85        | 94         | 61         | 87       | 27           | 24           | 32         | 42        | 33        | 57      | 92              | 62              | 87            | 90        |
  | 132         | 155         | 129         | 114       | 98         | 76         | 100      | 62           | 51           | 70         | 64        | 40        | 68      | 93              | 64              | 76            | 81        |
  | 137         | 158         | 135         | 112       | 112        | 83         | 101      | 15           | 13           | 17         | 58        | 41        | 71      | 85              | 61              | 81            | 82        |
  | 140         | 161         | 129         | 114       | 107        | 77         | 96       | 51           | 57           | 60         | 62        | 39        | 62      | 91              | 65              | 88            | 102       |
  | 144         | 148         | 131         | 122       | 92         | 82         | 99       | 52           | 42           | 52         | 59        | 44        | 72      | 85              | 63              | 88            | 104       |

### MP.9 Performance Evaluation 3: Detection time, Extraction time, Best Three.

- The detection time and extraction time for 17 pairs are as follows. For each combination, the first row is detection time (ms), and the second row is extraction time (ms).

  |     AKAZE_AKAZE | 38.7427  | 38.2321  | 36.3333  | 35.5254  | 37.3495  | 36.6156  | 36.5284  | 37.1584  | 35.9962  | 36.5343  |
  | --------------: | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- | -------- |
  |                 | 31.4759  | 30.7055  | 30.601   | 30.3501  | 29.6717  | 30.4214  | 30.5262  | 29.1384  | 31.1382  | 29.1641  |
  |     BRISK_BRIEF | 27.2606  | 26.7661  | 25.9327  | 25.78    | 25.7357  | 25.8668  | 25.6021  | 25.1917  | 25.8177  | 25.6536  |
  |                 | 0.994511 | 0.680178 | 0.718774 | 0.695496 | 0.697484 | 0.611736 | 0.646015 | 0.732436 | 0.639876 | 0.643969 |
  |     BRISK_FREAK | 26.6502  | 26.1077  | 25.7845  | 25.5655  | 25.6272  | 25.8712  | 25.8257  | 25.2817  | 25.3669  | 25.5679  |
  |                 | 25.2602  | 24.8548  | 24.1233  | 23.8612  | 23.8004  | 23.7601  | 23.5952  | 23.5429  | 23.6687  | 23.816   |
  |       BRISK_ORB | 27.0071  | 26.6958  | 27.1932  | 26.0772  | 26.4985  | 25.6043  | 25.8448  | 25.7299  | 25.4235  | 25.5504  |
  |                 | 3.14627  | 2.82552  | 2.67054  | 2.55846  | 2.59779  | 2.74703  | 2.68526  | 2.66448  | 2.62372  | 2.73198  |
  |      FAST_BRIEF | 0.645737 | 0.509338 | 0.497907 | 0.483429 | 0.480548 | 0.538022 | 0.524486 | 0.574459 | 0.549386 | 0.532323 |
  |                 | 0.825445 | 0.678456 | 0.694334 | 0.627688 | 0.633605 | 0.489878 | 0.730159 | 0.662427 | 0.652259 | 0.366599 |
  |      FAST_FREAK | 0.532173 | 0.576425 | 0.525257 | 0.558657 | 0.519704 | 0.574418 | 0.53711  | 0.484339 | 0.531315 | 0.585267 |
  |                 | 25.477   | 25.1477  | 23.6529  | 23.2822  | 23.4439  | 23.3171  | 24.2989  | 23.2882  | 23.2888  | 23.3712  |
  |        FAST_ORB | 0.641324 | 0.58629  | 0.48272  | 0.481995 | 0.612733 | 0.588343 | 0.564407 | 0.573542 | 0.559881 | 0.563466 |
  |                 | 0.975024 | 0.766512 | 0.681414 | 0.697949 | 0.59559  | 0.623771 | 0.781369 | 0.702771 | 0.714609 | 0.633099 |
  |    HARRIS_BRIEF | 8.69923  | 8.22619  | 7.55066  | 7.72795  | 8.58701  | 7.07095  | 6.05297  | 6.67944  | 6.20335  | 6.19633  |
  |                 | 0.287341 | 0.725901 | 0.681275 | 0.579329 | 0.268605 | 0.766296 | 0.164438 | 0.387226 | 0.394741 | 0.626369 |
  |    HARRIS_FREAK | 8.59718  | 7.65963  | 6.34976  | 6.24293  | 6.04683  | 6.31194  | 6.06846  | 6.09231  | 6.19634  | 6.23447  |
  |                 | 24.3818  | 23.6405  | 23.3196  | 22.8401  | 22.8119  | 23.882   | 22.8163  | 23.1199  | 22.9042  | 23.3053  |
  |      HARRIS_ORB | 8.54078  | 7.83292  | 7.62399  | 7.80544  | 7.83478  | 6.31752  | 6.52296  | 8.05092  | 7.78686  | 7.71318  |
  |                 | 0.621957 | 0.795494 | 0.800353 | 0.779487 | 0.689113 | 0.830389 | 0.643667 | 0.774505 | 0.79543  | 0.837704 |
  |       ORB_BRIEF | 4.64585  | 4.44485  | 4.45883  | 4.16095  | 4.38711  | 4.57756  | 4.10269  | 3.99672  | 4.28447  | 4.08026  |
  |                 | 0.769553 | 0.27816  | 0.324718 | 0.366085 | 0.393862 | 0.395448 | 0.381862 | 0.391029 | 0.339228 | 0.397774 |
  |       ORB_FREAK | 4.81567  | 4.54347  | 4.04312  | 4.05634  | 4.1161   | 4.16066  | 3.98135  | 4.18235  | 3.90484  | 4.02725  |
  |                 | 25.2881  | 24.4486  | 23.4548  | 23.2291  | 23.0473  | 23.4178  | 23.1628  | 23.2823  | 23.216   | 23.3229  |
  |         ORB_ORB | 4.64338  | 4.34531  | 4.03283  | 4.17905  | 4.26     | 3.88189  | 4.06186  | 3.94275  | 4.01131  | 4.00856  |
  |                 | 2.7971   | 2.3933   | 2.63375  | 3.22037  | 2.88126  | 2.75474  | 2.60193  | 2.63447  | 2.44849  | 2.47675  |
  | SHITOMASI_BRIEF | 9.19049  | 8.15029  | 8.9984   | 8.53341  | 7.89912  | 6.07855  | 6.01322  | 6.3472   | 7.94732  | 6.72722  |
  |                 | 0.660995 | 0.745268 | 0.733581 | 0.827919 | 0.426922 | 0.383765 | 0.382819 | 0.390547 | 0.436727 | 0.719449 |
  | SHITOMASI_FREAK | 9.25601  | 7.92618  | 6.44384  | 5.94758  | 6.03575  | 6.19561  | 6.02038  | 6.19748  | 6.5343   | 6.05263  |
  |                 | 24.7259  | 23.8865  | 23.2595  | 23.1169  | 22.9955  | 23.1028  | 23.3331  | 23.1762  | 22.9968  | 23.0338  |
  |   SHITOMASI_ORB | 9.50781  | 8.59923  | 8.42044  | 8.31375  | 8.79126  | 8.52503  | 8.3251   | 8.31297  | 8.60219  | 8.14241  |
  |                 | 0.820154 | 0.798389 | 0.736929 | 0.726288 | 0.746016 | 0.782126 | 0.727202 | 0.716626 | 0.708267 | 0.669184 |
  |       SIFT_SIFT | 53.0619  | 49.3807  | 50.0712  | 49.1134  | 51.5435  | 50.7647  | 43.3271  | 49.3318  | 48.8474  | 49.3506  |
  |                 | 44.5715  | 43.6893  | 43.3666  | 43.453   | 44.1839  | 44.3605  | 44.2557  | 44.3112  | 43.9839  | 43.6504  |

1. `SIFT`, `AKAZE`, `FREAK` extraction, `BRISK` detection spends more than 20ms which may not be affordable for real time object detection. So I only consider the remaining 8 combinations: `FAST_BRIEF, FAST_ORB, HARRIS_BRIEF, HARRIS_ORB, ORB_BRIEF, ORB_ORB, SHITOMASI_BRIEF, SHITOMASI_ORB`.
2. Combinations using `HARRIS` detection have too few matches (less than 20) which may be not robust enough. So I do not use `HARRIS` detector. And now I only consider the remaining 6 combinations: `FAST_BRIEF`, `FAST_ORB`, `ORB_BRIEF`, `ORB_ORB`, `SHITOMASI_BRIEF`, `SHITOMASI_ORB`.
3. Again, `SHITOMASI` detector needs almost double of the time needed by `ORB` detector. So I only consider the remaining 4 combinations: `FAST_BRIEF, FAST_ORB, ORB_BRIEF, ORB_ORB`.
4. My conclusion: for time consideration, I **recommend `FAST_BRIEF` and `FAST_ORB`**. Between `ORB_BRIEF` and `ORB_ORB`,  I think both are OK. And I **recommend `ORB_ORB`** for the following 4 small reasons.
   1. Apparently, `ORB` detector and `ORB` extractor are designed to be used together.
   2. At some frames, when `ORB_BRIEF` has only less than 40 matches, `ORB_ORB` still has about 50 matches (except the 1st frame), which shows more robustness.
   3. The time for `ORB` extraction does not exceed the `ORB` detection time.
   4. `ORB_SLAM2` uses `ORB` extractor.
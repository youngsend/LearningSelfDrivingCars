### 1. Implementation steps for the 2D CFAR process 

- Create two `zeros` with the same size of `RDM`, to hold thresholds and comparison results:

  ```matlab
  threshold_cfar = zeros(size(RDM));
  signal_cfar = zeros(size(RDM));
  ```

- Convert `RDM` from db to pow, to calculate mean of training cells. Also calculate the number of training cells.

  ```matlab
  RDM_pow = db2pow(RDM);
  train_cell_number = (2*(Gr+Tr)+1) * (2*(Gd+Td)+1) - ...
      (2*Gr+1) * (2*Gd+1);
  ```

- Calculate threshold in for loop. 

  ```matlab
  for i = Gr+Tr+1:Nr/2-Gr-Tr
      for j = Gd+Td+1:Nd-Gd-Td
          noise_level = sum(RDM_pow(i-(Gr+Tr):i+(Gr+Tr),...
              j-(Gd+Td):j+(Gd+Td)), 'all') - ...
              sum(RDM_pow(i-Gr:i+Gr, j-Gd:j+Gd), 'all');
          threshold_cfar(i,j) = pow2db(noise_level / train_cell_number) + offset;
      end
  end
  ```

  - Due to size of training and guard cells, start and end indices for range and doppler dimensions are `Gr+Tr+1:Nr/2-Gr-Tr` and `Gd+Td+1:Nd-Gd-Td` respectively.
    - Therefore, **the margin of `threshold_cfar` remains 0**.
  - To calculate sum within training cells, I subtract **sums of two rectangles** `(i-(Gr+Tr):i+(Gr+Tr), j-(Gd+Td):j+(Gd+Td)) ` and `(i-Gr:i+Gr, j-Gd:j+Gd)`.
  - Before adding offset, convert mean value back to db.

- The final step is explained in #3.

### 3. Steps taken to suppress the non-thresholded cells at the edges

- Only use values within  `(Gr+Tr+1:Nr/2-Gr-Tr, Gd+Td+1:Nd-Gd-Td)`  to compare with `threshold_cfar`.

  ```matlab
  % leave margin area to 0
  signal_cfar(Gr+Tr+1:Nr/2-Gr-Tr, Gd+Td+1:Nd-Gd-Td) = ...
      RDM(Gr+Tr+1:Nr/2-Gr-Tr, Gd+Td+1:Nd-Gd-Td);
  
  % compare signal to threshold
  signal_cfar = double(signal_cfar > threshold_cfar);
  ```

  - Since both `signal_cfar` and `threshold_cfar` margins are 0, they remain 0 after comparison.

### 2. Selection of Training, Guard cells and offset

- My selection are:

  ```matlab
  %Select the number of Training Cells in both the dimensions.
  Tr = 12;
  Td = 10;
  
  %Select the number of Guard Cells in both dimensions around the Cell under 
  %test (CUT) for accurate estimation
  Gr = 6;
  Gd = 6;
  
  % offset the threshold by SNR value in dB
  offset=10;
  ```

  - For `offset`, I modified it so that there are no peaks far away from the initial velocity (`50m/s`), at Doppler axis.
  - For training and guard cell numbers, I count the number of high db cells (from range doppler map) in both dimensions to decide the initial guess.
  - After that, I modified them to decrease the width of peaks (currently 14 at Doppler axis) in 2D CFAR results.

# Optimizing GPU Settings \(P2, P3, and G3 Instances\)<a name="optimize_gpu"></a>

There are several GPU setting optimizations that you can perform to achieve the best performance on P2, P3, and G3 instances\. By default, the NVIDIA driver uses an autoboost feature, which varies the GPU clock speeds\. By disabling the autoboost feature and setting the GPU clock speeds to their maximum frequency, you can consistently achieve the maximum performance with your GPU instances\.

**To optimize GPU settings**

1. Open a PowerShell window and navigate to the NVIDIA installation folder\.

   ```
   cd "C:\Program Files\NVIDIA Corporation\NVSMI"
   ```

1. Disable the autoboost feature for all GPUs on the instance\.

   ```
   .\nvidia-smi --auto-boost-default=0
   ```
**Note**  
GPUs on P3 instances do not support autoboost\.

1. Set all GPU clock speeds to their maximum frequency\. Use the memory and graphics clock speeds specified in the following commands\.
**Note**  
Some versions of the NVIDIA driver do not allow setting application clock speed and throw a `"Setting applications clocks is not supported for GPU …"` error, which you can ignore\.
   + P2 instances:

     ```
     .\nvidia-smi -ac "2505,875"
     ```
   + P3 instances:

     ```
     .\nvidia-smi -ac "877,1530"
     ```
   + G3 instances:

     ```
     .\nvidia-smi -ac "2505,1177"
     ```
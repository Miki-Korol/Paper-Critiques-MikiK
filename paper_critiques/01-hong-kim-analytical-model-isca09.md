# An Analytical Model for a GPU Architecture with Memory-level and Thread-level Parallelism Awareness

**Paper:** Sunpyo Hong, Hyesoon Kim · *ISCA 2009* · [Link](https://doi.org/10.1145/1555815.1555775)  
**Critique by:** Miki Korol · 10/11/26  
*Written for CS 254A: Advanced GPU Microarchitecture, UCLA*

---

This paper proposes a model which allegedly accurately predicts GPU performance or multithreaded program performance using only static time available information. The main idea is to reference two metrics: CWP (the number of warps that the SM can execute during a memory warp waiting period \+ 1\) and MWP (the number of simultaneous warps that can access memory from the time the first warp is issued with a memory instruction until the warp is done servicing all of the threads) to calculate app execution cycles. CWP is mainly used to determine if a program's execution cycles will be dominated by memory access time or computation cost which can be seen in the independent equations (23) and (24) which both calculate app execution time depending on the magnitude comparison of MWP and CWP.

The paper tackles the challenge of finding the real bottlenecks in GPU application execution and creating a useful model with relevant metrics to closely predict execution times with fairly simple static time information.

Some other challenges the paper addresses are the effects of coalesced vs uncoalesced memory accesses, synchronization, and a dynamic number of instructions are modeled.

The model itself is constructed with hardware constraints in mind as well as program specific modifications as well. I will analyze the execution cycles for an application for the case when CWP \>= MWP because that is said to be the general case. In that case they model the application execution cycles as (Mem\_Cycles \* N/MWP \+ Comp\_cycles/\#Mem\_insts \* (MWP-1)) \* \#Rep. As you can see since memory is the bottleneck it is the only component of this model.

Mem\_Cycles \* N/MWP is the actual memory latency where Mem\_Cycles is the memory latency for one warp and it is multiplied by N warps. However, The memory warp parallelism allows MWP warps to group their latency therefore dividing N warps into N/MWP Groups. Comp\_cycles is the issue\_cycles times total\_insts so dividing it by Mem\_insts gives you the cycle count in between successive memory instructions. (MWP-1) memory instructions with delays must be issued before the memory warp parallelism is full and no further memory warps can be issued. This all details the memory latency and other memory delays for one wave then it is all multiplied by the \#Rep which is like the number of waves to complete the application.

The challenge of taking into account memory coalescing is simply solved by using a separate intrinsic latency for each type of memory load and factoring in the ratio of memory coalesced vs non coalesced memory loads. This can be seen in the calculation of Mem\_Cycles (equation 18 in the paper) from the equation above: Mem\_Cycles \= Mem\_L\_Uncoal \* \#Uncoal\_Mem\_Insts \+ Mem\_L\_Coal \* \#Coal\_Mem\_Insts

Synchronization is accounted for by adding a Synch\_cost delay term which in short is dependant only on the delay between the first and last memory loads issued times the number of stalls times the blocks because each block will have its own stall times the waves in the program resulting in a simply calculatable additive delay. The paper mentions that when instruction count for a program becomes large these sync stalls tend to become negligible

The paper uses several different GPUs \[8800GTX, Quadro FX5600, 8800GT, GTX280\] to compare the model predictions to real execution times. They use cudaEventRecord API to measure the GPU Kernel execution time. Several different types of benchmarks are measured on all of the GPUs. Each benchmark is modeled and measured at a wide range (\~32-420+) of threads per block and the results for each benchmark were plotted to compare the model to experimental values.

The model does accurately fit the benchmarks tested on with geometric mean of errors on benchmarks of 5.4% and 13.3% on GPU computing applications.

This paper seems to be the first to take the approach of modeling execution cost at compile time by using static information. Assuming the model is accurate, this would allow GPGPU programmers to take advantage of the information the model gives and assume some insights into their bottlenecks and possibly change their code or use a different kernel etc.

The model takes into account a large amount of static variables and carefully selects important metrics highlighting the common memory bottleneck which allows them to formulate an accurate prediction of execution time. The results seem to be strong with low error rates on the benchmarks they tested.

One issue which they mentioned is the lack of variable length memory access times AKA they didn't account for different cache levels and cache misses too. Since they assumed that the memory calls were all to DRAM memory and they all hit, they used a constant for the round trip latency Mem\_LD. This allowed them to make more assumptions like a fixed departure delay and Mem\_LD resulted in the simple synchronisation calculation which would not hold with variable length memory latencies. Also the calculation for MWP would be harder as some of the parallel accesses could be delayed due to the cache misses and such. These limitations might cause this simplified model to be inaccurate on more complex programs and architectures.

It challenged the modeling field a lot creating a new methodology, however the applicability for more complex systems like modern GPU architectures with caches could cause such a model with assumptions to lose accuracy and become useless.

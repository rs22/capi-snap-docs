## Optimization

After the first correct version of the AFU is flashed on the FPGA and tested, there usually comes a disappointment: Without explicit optimization, very few algorithms perform better on the FPGA than on the CPU. FPGAs support due to their generic structure much lower clock frequencies in comparison to CPUs. The value of an FPGA solution lies in the fact, that problems can be solved with parallel hardware that implements exactly the required functionality in few clock cycles, instead of using sequential general purpose instructions.

As it is generally not obvious, in what way an algorithm can be most efficiently parallelized, it is the task of the AFU developer to annotate the HLS code sufficiently to point out parallelization possibilities and remove obstacles such as false dependencies.

Another important consideration is whether one particular parallelization (that multiplies the required hardware resources by the respective factor) really improves the focused overall performance indicator. In a throughput oriented scenario like the Blowfish AFU it seems sensible to use multiple instances of the encrypt function to encrypt multiple blocks in parallel. This measure is however only useful, if the data blocks can be transferred from host memory with a sufficient rate. Otherwise the throughput is limited by the host memory bus and a far more useful strategy would be to improve throughput by implementing intelligent buffering and prefetching strategies.

The best starting point for optimization can often be found by experimenting with the real hardware. Thus our first step to optimize the Blowfish AFU was to perform two series of throughput measurements. The first only read and wrote back data blocks without encrypting them but keeping the same access patterns. In the second series the AFU did not interact with host memory at all but only performed the requested number of encryptions on a dummy data block. Based on this information it could be determined that no more than 16 parallel encrypt instances can bring any performance benefit because the memory throughput was about 16 times as large as the encrypt throughput.

[!IMG throughput plots]

[! improve encrypt throughput]
    [! parallel implementation of encrypt, characteristics and dependencies]
    [! read only port duplication with multiple arrays and ARRAY_PARTITION]

[! improve memory throughput, 4k buffering]

[! results and conclusion, comparison to CPU solution] 
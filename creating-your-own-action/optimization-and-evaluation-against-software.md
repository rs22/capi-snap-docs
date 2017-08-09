## Optimization

After the first correct version of the AFU is flashed on the FPGA and tested, there usually comes a disappointment: Without explicit optimization, very few algorithms perform better on the FPGA than on the CPU. FPGAs support due to their generic structure much lower clock frequencies in comparison to CPUs. The value of an FPGA solution lies in the fact, that problems can be solved with parallel hardware that implements exactly the required functionality in few clock cycles, instead of using sequential general purpose instructions.

As it is generally not obvious, in what way an algorithm can be most efficiently parallelized, it is the task of the AFU developer to annotate the HLS code sufficiently to point out parallelization possibilities and remove obstacles such as false dependencies.


### Performance Analysis

Another important consideration is whether one particular parallelization (that multiplies the required hardware resources by the respective factor) really improves the focused overall performance indicator. In a throughput oriented scenario like the Blowfish AFU it seems sensible to use multiple instances of the encrypt function to encrypt multiple blocks in parallel. This measure is however only useful, if the data blocks can be transferred from host memory with a sufficient rate. Otherwise the throughput is limited by the host memory bus and a far more useful strategy would be to improve memory throughput by implementing intelligent buffering and prefetching strategies.

The best starting point for optimizations can often be found by experimenting with the real hardware. Thus our first step to optimize the Blowfish AFU was to perform two series of throughput measurements. The first only read and wrote back data blocks without encrypting them but keeping the same access patterns. In the second series the AFU did not interact with host memory at all but only performed the requested number of encryptions on a dummy data block. Based on this information as visualized in the plot below, we determined that the encrypt function was the current bottleneck and that a speedup of up to 16 times will benefit the overall performance.

![](/assets/throughputCombined.svg)


### Improving Encrypt Performance

To speed up the encryption process, it is necessary to use parallel instances of the encrypt function on separate blocks of data. Because of the internal block structure of the Blowfish cipher, most operations during the encryption of one particular block depend directly on the results of its predecessors, which makes improving the single block encrypt performance by parallelization impossible.

Encrypting multiple blocks in parallel imposes however a severe limitation on the feasible cipher modes. All chaining modes make the encryption of a block dependent on the result for the previous block, which collides with parallel block encryption unless several different data streams are to be encrypted concurrently. This leaves ECB and CTR mode. While ECB has severe security limitations, CTR mode provides a sufficient level of security.
Therefore the parallel encryption of data blocks - though limiting - is still a valid paradigm.

In order to effectively utilize multiple instances of the encrypt function, care must be taken to coordinate access to resources shared by all instances, i.e. the S and P arrays.
With no key initialization operation pending, these arrays are read only resources, so that no inconsistent states can arise from multiple instances interacting with the arrays in parallel. However the read access patterns among the instances are problematic: While the P array accesses are the same for every instance given a synchrinized invocation, the four S array accesses performed by the `bf_f()` subroutine are data dependent, forbidding any prediction about common or regular access patterns.

In this situation it is necessary to understand how arrays are implemented on the FPGA hardware: One or more Block RAM resources are instantiated to create a contiguous memory space large enough to hold the array, which is later mapped into that space. Each Block RAM has two access ports, so that in one clock cycle two independent memory operations can be performed.

More than two parallel instances of `bf_f()` require more than two memory access ports to the S array. This particular read-only-case permits to overcome this limitation by maintaining a sufficient (\#instances/2) number of copies. This is however only useful, if each copy of the array is mapped into its particular set of Block RAMs so that no two copies share one Block RAM. To acheive this, HLS provides an annotation to control how arrays are mapped to Block RAMs:

```
//in action_blowfish.H
typedef ap_uint<BF_S_DATA_W> bf_S_t[BF_S_CPYCNT][BF_S_ARYCNT][BF_S_ENTCNT];

//in hls_blowfish.cpp
static bf_S_t g_S;

//in hls_blowfish.cpp:hls_action()
#pragma HLS ARRAY_PARTITION variable=g_S complete dim=1
```

This example shows, how a third dimension is added to the originally two-dimensional S array to provide space for its copies. The `#pragma`, which must be placed inside a function (preferably the entry point `hls_action()`), partitions `g_S` completely along the copy dimension ensuring that each copy resides in a separate Block RAM.

With an appropriate number of read ports, it is now necessary that the generated hardware structures utilize them efficiently. This involves explicit scheduling, which instance of `bf_f()` reads from which copy of the S array.

[! improve encrypt throughput]
    [! parallel implementation of encrypt, characteristics and dependencies]
    [! read only port duplication with multiple arrays and ARRAY_PARTITION]

[! improve memory throughput, 4k buffering]

[! results and conclusion, comparison to CPU solution] 
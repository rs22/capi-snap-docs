## Hardware Development

As mentioned earlier, SNAP supports hardware development with Vivado HLS \(subsequently called HLS\), that translates C/C++ code to VHDL or Verilog.

The translation involves tracing the dependencies between sequential statements and identifying parallelizable groups, resulting in the automatic generation of a state machine producing the same results as the original C code. This enables software developers to quickly port existing algorithms to a hardware implementation. For some easily parallelizable algorithms this might be enough to acheive substantial speedups, as HLS recognizes the optimization potential and generates parallel hardware. In the general case however, the formulation of an algorithm that is optimized for a sequential execution model is not a favourable candidate for automatic optimization, as HLS has hardly the same domain knowledge of a hardware developer, who knows or guesses how best to restructure the algorithm to distribute the workload well among the available resources. Therefore HLS provides a set of annotations \(`#pragma HLS ...`\) that allow a developer to direct the hardware generation process in specific ways to give a more fine grained control over the generated hardware. In some specific cases it might even be necessary to restructure the C code by hand in order to make HLS generate the intended hardware structures.

A basic SNAP AFU project consists of two files in the hardware (`hw`) subdirectory of the action repository: One hardware specific header file, and the C implementation. In our case these are `action_blowfish.H` and `hls_blowfish.cpp`. Note the capital `H` in the header filetype. This convention is used to distinguish the hardware specific from the common header file `action_blowfish.h`. The latter contains the definition oft the job structure and is thus included in both the hardware and software implementation.


### Porting the Algorithm to HLS

The first step of the AFU development is the implementation of the actual algorithm in HLS. Depending on the style and complexity of an existing software implementation, it might be more or less easy to adapt it to the limitations of HLS' subset of C/C++: Dynamic memory allocation and some pointer operations are not supported and algorithms relying on dynamic data structures need to be reorganized to fit into the more static nature of a hardware implementation.

Blowfish however operates on blocks of a fixed size and needs only a fixed number of intermediate results, which makes the port to hardware easy. The [Wikipedia article](https://en.wikipedia.org/wiki/Blowfish_%28cipher%29) provides a compact pseudocode representation, that needs only minor adaptions to serve as a first step to the hardware implementation:

\[!CODE\]

When porting pseudo or existing C code to HLS, care should be taken when choosing data types: In software development it does not usually make a difference in terms of perfomance, if much wider data types than necessary are used, provided they do not exceed the underlying machine's native register width. When specifying hardware however, a data type's bit width has a significant impact on the size and performance of a design.
The generated hardware is exactly large enough to process the specified bit count. During the translation process it is generally impossible to determine the range of values a variable might take at runtime so that the worst case must be assumed. Therefore it is advisable to specify the bit width of a variable as tightly as possible. To enable a finer control, the template types `ap\_uint` and `ap\_int` can be parameterized to represent any integral bit width.

This can be seen in the code above. Instead of using the usual `int` for loop counters the `ap\_uint<5>` type was selected. That is the smallest unsigned type that can represent the number 16 and causes only a 5 bit instead of a 32 bit adder to be implemented.


### Testbench I

The hardware implementation (`hls_blowfish.cpp`) can define a `main()` function, that should be non-synthesizable. It will be the entry point when debugging and should contain a testbench for the HLS code.

In a later stage of the development, it will set up a complete SNAP environment to execute the AFU code in software just as if it was invoked in hardware. For now it is sufficient to write test cases for the algorithm to debug the implementation and assure its correctness before the SNAP integration can begin. The example below shows how the encrypt function could be tested.

```
#ifdef NO_SYNTH
int main()
{
    bf_halfBlock_t left = 0xda7a, right = 0xb10c;
    printf("encrypt(0x%08x, 0x%08x) -> ", left, right);
    bf_encrypt(left, right);
    printf("0x%08x, 0x%08x\n", left, right);
}
#endif
```

### Integration with the SNAP Framework

The next step on the way to a working Blowfish-AFU is to adapt its structure to the SNAP framework. SNAP expects a module named `hls_action` and instantiates it as the AFU user design when composing the final design on the FPGA.
HLS translates all functions to separate VHDL modules unless they are inlined. Consequently the AFU code must contain a function named `hls_action()` that will be the entry point to the AFU logic.
The arguments of this function represent different parts of the environment that SNAP offers to an AFU: They include pointers to host memory, a pointer to the job structure and if enabled a pointer to the cards local memory. (See `USE_DRAM=TRUE` with `make config`). `hls_action()` is *called* each time the job interface of the PSL indicates that a job should be started.

All AFUs must be able to introduce themselves to the host with their action code and hardware revision. There is a list of assigned action codes in the SNAP repository named `ActionTypes.md`. A new action code is registered by a successful merge request that adds the new assignment to that list. There is however a large range of codes reserved for experimental use which - though with the risk of collisions - should be ideal for the private development of a new AFU. The hardware revision is at the discretion of the developer and should only be increased if significant changes in the host visible AFU interface or behavior have occurred.

If the host queries this information, `hls_action()` is entered just as if a regular job was started. To distinguish both cases, a flag in the control register is set to indicate that instead of executing a job the AFU should write the respective entries in a configuration memory space, accessible through a pointer argument to `hls_action()`.
The specific code that implements this behavior can be seen below and - as it is not specific to a particular AFU - can be freely reused.

[!CODE action discovery]

To separate this mechanism from the actual logic, the Blowfish AFU calls `process_action()` if the `hls_action()` invocation was not a configuration request. This function is the first AFU specific part and the right place to extract all necessary parameters and commands from the job structure. The Blowfish AFU provides three separate operations: Encryption, decryption and key initialization. They are distinguished by specific values of the `mode` field and use the input and output buffers if applicable. To maintain a clear structure, the operations are implemented in separate functions: `action_setkey()` performs the key initialization, whereas `action_endecrypt()` handles both en- and decryption as they are very similar. These functions contain the required memory access logic to execute the Blowfish algorithm consisting of the `bf_*()` functions efficiently.


### Testbench II

With the SNAP action code in place, the testbench should be changed accordingly, to call `hls_action()` with a correctly set up environment. This includes arrays of the `snap\_membus\_t` type for each bus that is connected to the action module, i.e. two busses to host memory and optionally one to the DRAM, as well as the action and config registers (`action_reg` and `act\_R0\_config\_reg`). The memory arrays must be initialized to contain the data, on which the action will operate. The action register must contain a correctly initialized job structure. The config register contains the flag to distinguish discovery from normal mode and this flag is the only part that needs to be set for testbench purposes.

With all these preparations in place `hls_action()` can be called so that all parts of the action functionality are covered by the testbench. Should that produce incorrect results, breakpoints and variable inspection are effective means to find the bug.


### Using the SNAP Environment

With the memory and register pointers passed to `hls_action()` SNAP already provides everything to access the resouces available to an AFU. By dereferencing and using pointer arithmetic different memory areas can be read and written. After the job structure pointer, that translates to an interface to a job management module provided by SNAP, the most interesting resource to any AFU will be host memory.

When interacting with host memory, there arises a slight incongruity: While a regular bus interface can be expected to be bidirectional, SNAP provides two separate interfaces for host memory access, one for read and one for write operations. This is due to the way Vivado HLS translates bus interfaces, which is not quite compatible with SNAPs PSL interface module.

```
snap\_membus\_t result = din\_gmem\[address >> ADDR\_RIGHT\_SHIFT\];
```

The statement above performs a single read operation from host memory. The result is a `snap\_membus\_t` which represents one word with the native bus width of the underlying PSL interface. The [!?current] width is 512 bit, so only 64 byte aligned accesses to 64 byte blocks of data are possible. The addresses specified by the host software will generally be byte addresses, while the host memory pointer is indexed in multiples of 64 bytes. That necessitates the right shift of the byte address. Care should be taken if the lower bits of the byte address are not 0. In that case the desired part must be extracted from the result according to those lower bits.

Every time a host memory pointer is dereferenced, a transaction on the PSL interface ensues. If large amounts of data need to be transferred, it is more efficient to issue a block operation. The library function 
[!!! hls_memcopy replaced memcopy() call by manual implementation, mention here?]


[!? introduce BRAM, local arrays?]
[!! resolve confusion with job structure location: CAPI = hostmem; SNAP = obscure part of MMIO space?!]

[! arguments of hls_action, Control regs and AXI busses (2Host!, 0-1 DRAM, 0-1 NVMe)]

[! access job struct]

[!REF io performance, 4k buffering -> Opt section]
[!? How to integrate source code, which version?]

### Maintaining a Redundant Software Implementation

Though not necessary for the hardware implementation itself, it is often a good idea to maintain a separate implementation of the AFU functionality in software. Besides being a good reference for testing the hardware implementation correctness, is also serves as a baseline for performance analyses.

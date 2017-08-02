## Hardware Development

As mentioned earlier, SNAP supports hardware development with Vivado HLS \(subsequently called HLS\), that translates C/C++ code to VHDL or Verilog.

The translation involves tracing the dependencies between sequential statements and identifying parallelizable groups, resulting in the automatic generation of a state machine producing the same results as the original C code. This enables software developers to quickly port existing algorithms to a hardware implementation. For some easily parallelizable algorithms this might be enough to acheive substantial speedups, as HLS recognizes the optimization potential and generates parallel hardware. In the general case however, the formulation of an algorithm that is optimized for a sequential execution model is not a favourable candidate for automatic optimization, as HLS has hardly the same domain knowledge of a hardware developer, who knows or guesses how best to restructure the algorithm to distribute the workload well among the available resources. Therefore HLS provides a set of annotations \(`#pragma HLS ...`\) that allow a developer to direct the hardware generation process in specific ways to give a more fine grained control over the generated hardware. In some specific cases it might even be necessary to restructure the C code by hand in order to make HLS generate the intended hardware structures.

### Building a Blowfish AFU From Scratch

The development process will be demonstrated with the Blowfish algorithm, which is a relatively easy to understand symmetric block cipher. The [Wikipedia article](https://en.wikipedia.org/wiki/Blowfish_%28cipher%29) provides a compact pseudocode representation, that needs only minor adaptions to serve as a first step to the hardware implementation:

\[!CODE\]

When porting pseudo or existing C code to HLS, care should be taken when choosing data types: When writing software it does not usually make difference in terms of perfomance, if data types are used that are much wider than necessary, provided they do not exceed the underlying machine's native register width. When specifying hardware however, the bit width of a used data type has a significant impact on the size and performance of a design.
The resulting hardware is exactly large enough to process the specified bit count. During the translation process it is generally impossible to determine the range of values a variable might take at runtime so that the worst case must be assumed. Therefore it is advisable to specify the bit width of a variable as tightly as possible. To enable a finer control, the template types `ap\_uint` and `ap\_int` can be parameterized to represent any integral bit width.

This can be seen in the code above. Instead of using the usual `int` for loop counters the `ap\_uint<5>` type was selected. That is the smallest unsigned type that can represent the number 16 and causes only a 5 bit instead of a 32 bit adder to be implemented.

### Integration with the SNAP Framework

The next step on the way to a working Blowfish-AFU is to adapt its structure to make it compliant with the SNAP framework. SNAP expects a module named `hls_action` and instantiates it as the AFU design when building the final design on the FPGA.
HLS translates all functions to separate VHDL modules unless they are inlined. Consequently the AFU code must contain a function named `hls_action()` that will be the entry point into the AFU logic.
The arguments of this function represent different parts of the environment that SNAP offers to an AFU: They include pointers to host memory, a pointer to the job structure and if enabled a pointer to the cards local memory. (See `USE_DRAM=TRUE` with `make config`). `hls_action()` is *called* each time the job interface of the PSL indicates that a job should be started.

All AFUs must be able to introduce themselves to the host with their action code and hardware revision. There is a list of assigned action codes in the SNAP repository named `ActionTypes.md`. A new action type is registered by a successful merge request that adds the new assignment to that list. There is however a large range of codes reserved for experimental use which - though with the risk of collisions - should be ideal for the private development of a new AFU. The hardware revision is at the discretion of the developer and should only be increased if significant changes in the host visible AFU interface or behavior have occurred.

If the host queries this information, `hls_action()` is entered just as if a regular job was started. To distinguish both cases, a flag in the control register is set to indicate that instead of executing a job the AFU should write the respective entries in a configuration memory space, accessible through a pointer argument to `hls_action()`.
The specific code that implements this behavior can be seen below and - as it is not specific to a particular AFU - can be freely reused.

[!CODE action discovery]

To separate this mechanism from the actual logic, the Blowfish AFU calls `process_action()` if the `hls_action()` invocation was not a configuration request. This function is the first AFU specific part and the right place to extract all necessary parameters and commands from the job structure. The Blowfish AFU provides three separate operations: Encryption, Decryption and Key Initialization.


The Blowfish AFU provides three operations: , that share the buffer addresses and are distinguished by specific values of the mode field. [!REF: process_action()]

[! arguments of hls_action, Control regs and AXI busses (2Host!, 0-1 DRAM, 0-1 NVMe)]

[! access job struct]
[! access host memory]

[!REF io performance, 4k buffering -> Opt section]
[!? How to integrate source code, which version?]


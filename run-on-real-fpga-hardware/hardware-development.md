## Hardware Development

As mentioned earlier, SNAP supports hardware development with Vivado HLS \(subsequently called HLS\), that translates C/C++ code to VHDL or Verilog.

The translation involves tracing the dependencies between sequential statements and identifying parallelizable groups, resulting in the automatic generation of a state machine producing the same results as the original C code. This enables software developers to quickly port existing algorithms to a hardware implementation. For some easily parallelizable algorithms this might be enough to acheive substantial speedups, as HLS recognizes the optimization potential and generates parallel hardware. In the general case however, the formulation of an algorithm that is optimized for a sequential execution model is not a favourable candidate for automatic optimization, as HLS has hardly the same domain knowledge of a hardware developer, who knows or guesses how best to restructure the algorithm to distribute the workload well among the available resources. Therefore HLS provides a set of annotations \(`#pragma HLS ...`\) that allow a developer to direct the hardware generation process in specific ways to give a more fine grained control over the generated hardware. In some specific cases it might even be necessary to restructure the C code by hand in order to make HLS generate the intended hardware structures.

This process will be demonstrated with the Blowfish algorithm. There is a compact pseudocode representation given with the [Wikipedia article](https://en.wikipedia.org/wiki/Blowfish_%28cipher%29), that needs only minor adaptions to serve as a first step to the hardware implementation:

\[!CODE\]

When porting pseudo or existing C code to HLS, care should be taken when choosing data types: While in software \(on common processor architectures\) it does not make any performance difference, whether a 32bit addition operation is used to add 2 numbers that are only 11bit wide, this makes a significant difference when specifying hardware. The resulting hardware is exactly large enough to add the specified number of bits. During the translation process it is generally impossible to determine the range of values a variable might take at runtime so that the worst case must be assumed. As larger bit widths generally cause larger and slower designs, it is advisable to use data types with the least possible width. To enable a finer control, the template types ap_uint and ap_int can be parameterized to represent any integral bit width.

This can be seen in the code above. Instead of using the common int loop counter, the ap\_uint&lt;5&gt; type was used which is the smallest type that can represent the number 16.



The next step to create a Blowfish-AFU is to integrate the algorithm into the action infrastructure. HLS translates all functions to separate modules unless they are inlined. The SNAP framework expects a module named `hls`_`action and instantiates it. The respective C function is consequently the entry point similar to the main() function. Its arguments are pointers of different types, that represent the axi busses that connect the user design once translated with the SNAP and ultimately PSL components. hls`_`action is "called" each time the job interface of the PSL indicates that a job should be started. [!describe fixed init procedure]`




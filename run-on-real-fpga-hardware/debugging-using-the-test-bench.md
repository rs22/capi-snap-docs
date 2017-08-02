## Debugging with the Vivado Testbench

When building an action there often arises the need for debugging. The classic approach in hardware development is to simulate the design on a testbench that applies predefined signal sequences to the design under test and monitors how it reacts. This procedure is perfectly possible with an HLS design as the C code is translated to VHDL modules that act like any user written hardware design. Nevertheless the translation process produces somewhat cryptic names for states and signals. Furthermore the state machine is not easily comprehensible to a developer used to C code and the mapping between states and C statements is not obvious. Therefore it is desirable to run the HLS code like a regular C program in a software debugger to fix errors on that level, always assuming that the translated hardware modules behave exactly as the original C code.

This is possible in the Vivado_HLS IDE that includes a C/C++ Debugger. Only a `main()` function is required, that sets up the environment expected by `hls_action()`. In a SNAP action this includes arrays of the `snap\_membus\_t` type for each bus that is connected to the action module, i.e. two busses to host memory and optionally one bus to the onboard DRAM, as well as the action and config registers (`action_reg` and `act\_R0\_config\_reg`). The memory arrays must be initialized to contain the data, on which the action will operate and the action register must contain a correctly initialized job structure. The config register contains the flag to distinguish discovery from normal mode and this is the only part that needs to be set for testbench purposes.

With all these preparations in place `hls_action()` can be called once or many times so that all parts of the action functionality are covered. Should that not produce the expected results, breakpoints and variable inspection can be used to find the bug.

[! Details for Blowfish testbench]

[! Bugs encountered through endianness confusion, Details regarding endianness in general]

[! Limitations of Testbench, cases that require hardware simulation]


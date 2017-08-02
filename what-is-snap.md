## What is SNAP?

As much as **CAPI** builds the technical foundations for effectively using accelerators, it can be **hard to adopt** due to various reasons:

* Having to write hardware code \(Verilog / VHDL\) and switching from procedural to state-based thinking may be hard for software engineers
* Many different components that have to be managed individually are part of the build, simulation and execution process
* The API leaves a lot of responsibility for the user - for example waiting for the completion of memory reads and writes

Therefore **SNAP**, which is build on top of CAPI, aims to make it **as easy as possible** to use FPGA-based hardware acceleration. It does that by providing two things: a unified, automated build process and a simpler API on top of CAPI.

### Build parts

The SNAP framework defines a unified build process for building, simulation and execution on hardware.

#### Combining Xilinx Vivado with various other tools and components

While Xilinx Vivado is used to synthesize and layout the action for the FPGA or simulation, a lot of external components are needed for building as well. For example the information which action to build and the PSL layouted for the target FPGA. SNAP bundles all neccessary files and settings and contains a structure of Makefiles to automate building based on them.

#### High Level Synthesis support

By incorporating Vivado HLS into the build process,
By incorporating Vivado HLS into the build process, the action behaviour can also be specified in C or C++ and gets automatically converted to hardware code. Even though it is not possible to completely abstract from thinking about pipelining and parrallelization, it removes the need for developers to learn VHDL or Verilog.

#### Support for multiple FPGA cards


SNAP can build for different FPGA cards 

Support for different FPGA cards

PSL checkpoint file

### Framework part

Simpler API

Memory access



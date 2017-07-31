## What is SNAP?

As much as CAPI builds the technical foundations for effectively using accelerators, it can be hard to adopt due to various reasons:

* Having to write **hardware code** \(Verilog / VHDL\) and switching from **procedural to state-based thinking** may be hard for software engineers
* Many different **components** that have to be **managed individually** are part of the build, simulation and execution process
* The **API leaves **a lot of** responsibility** for the user - for example **waiting for** the completion of **memory** reads and writes

Therefore SNAP, which is build on top of CAPI, aims to make it as easy as possible to use FPGA-based hardware acceleration. It does that by providing two things: a unified, automated build process and a simpler API on top of CAPI.

### Build parts

The SNAP framework defines a unified build process for building, simulation and execution on hardware.

#### Combining Xilinx Vivado and various other tools

TODO

---
<img src="assets/brain.png" width="50%">

#### something

asdasdssssssssssss
 This could be a background box.
sadasd

It could look like this... Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet. Lorem ipsum dolor sit amet, consetetur sadipscing elitr, sed diam nonumy eirmod tempor invidunt ut labore et dolore magna aliquyam erat, sed diam voluptua. At vero eos et accusam et justo duo dolores et ea rebum. Stet clita kasd gubergren, no sea takimata sanctus est Lorem ipsum dolor sit amet.

---

#### High Level Synthesis support

The build process includes the possibility to use Vivado HLS to convert C++ code to hardware code. This way developers do not have to learn VHDL or Verilog.

#### Support for multiple FPGA cards

SNAP can build for different FPGA cards

Support for different FPGA cards

PSL checkpoint file

### Framework part

Simpler API

Memory access


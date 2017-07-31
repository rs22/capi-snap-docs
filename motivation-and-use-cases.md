# Motivation and use cases

FPGAs provide the ability to efficiently execute parallel algorithms.

In comparison to other hardware accelerators that provide parallel processing capabilities, FPGAs are very flexible in terms of the possible application scenarios \(compared to on-chip accelerators\) while being orders of magnitude more power-efficient compared to graphics cards \(GPGPU acceleration\) because of their lower clock frequency.

Traditionally however, it was very time consuming and difficult for an application programmer to implement FPGA-based accelerators. The type of 'hardware programming' that is necessary is quite different from well-known programming paradigms used in imperative languages such as C or Java. Not only is a detailed knowledge of the targeted FPGA architecture needed to optimize the VHDL- or Verilog- hardware descriptions; a lot of time is often spent to establish communication channels and management interfaces used by the consuming application.

In order to overcome these issues, IBM provides the CAPI and SNAP frameworks discussed here. They allow a broader group of developers to leverage FPGAs in the following paradigms.

## FPGA-based acceleration paradigms

![](/assets/offload.png)

![](/assets/funnel.png)

![](/assets/ingress.png)

![](/assets/egress.png)


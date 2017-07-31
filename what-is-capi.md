## What is CAPI?

CAPI, the Coherent Accelerator Processor Interface is an interface standard introduced with IBMs POWER8 server architecture. As the name suggests, it enables Accelerators such as FPGAs to be part of the system's coherent cache hierarchy and memory subsystem formerly only available to CPU cores. Consequently, while former technologies mapped the accelerator to its specific IO memory area, where data had to be copied to explicitly, an accelerator connected via CAPI can access the same virtual address space as its controlling process, rendering copy operations redundant in many cases.

As yet PCIe, but independent of underlying bus system \(bandwidth limits\)

### Architecture

\[!IMG: CAPI structure\]

Figure \[!REF\] shows that CAPI consists of several components distributed among the accelerator and host CPU: \[glossar\]

Interface semantics

### Development Paradigms

Build state machines!




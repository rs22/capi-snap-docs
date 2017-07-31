## What is CAPI?

CAPI, the Coherent Accelerator Processor Interface is an interface standard introduced with IBMs POWER8 server architecture. As the name suggests, it enables Accelerators \(i.e. FPGAs\) to be part of the system's coherent cache hierarchy and memory subsystem formerly only available to CPU cores. Consequently, while former technologies mapped the accelerator to its specific IO memory area, where data had to be copied to explicitly, an accelerator connected via CAPI can access the same virtual address space as its controlling process.

As yet, a CAPI accelerator card is connected to the system via a PCIe slot. However CAPI itself is independent of the underlying bus system. Therefore subsequent versions of the POWER architecture might introduce different bus systems with higher bandwidths.

### Architecture

![](/assets/CAPI__block_diagramm.png)

Figure \[!REF\] shows that CAPI consists of several components distributed among the accelerator and host CPU: The logic on the FPGA is split into the application specific AFU \(Accelerator Function Unit\) part, that implements the accelerator hardware proper, and the PSL \(Power Service Layer\). The latter is a fixed design provided by IBM, that implements the interface logic necessary to provide cached virtual host memory access and job management services to the AFU.

The PSL communicates with the host part of the CAPI hardware, the Coherent Accelerator Processor Proxy \(CAPP\) via PCIe. The CAPP is part of the POWER CPU and from the point of view of the memory subsystem it has the same status as a processor core.

The software part of CAPI, running on regular processor cores, consists of a driver in the linux kernel that exposes cxl devices representing an installed CAPI accelerator card. To encapsulate the interaction with raw cxl devices via read/write and ioctl systemcalls, libcxl provides a C API with the same functionality. Any user application, given sufficient privileges to interact with the cxl device, can use functions from and link against libcxl and thereby use the functions implemented by the AFU on any installed accelerator card.

### Development Concepts

Besides using libcxl to initialize and attach a CAPI accelerator to the current process, the hardware design of the AFU constitutes the central part of CAPI development.

The AFU is a piece of logic that was configured into the FPGA and therefore the design of the AFU must be expressed in a hardware description language such as VHDL or Verilog. Such language differ significantly from imperative languages like C in that most statements have concurrent semantics as they translate to separate parts of hardware that operate independently from one another. If an algorithm requires several sequential steps, it must be represented in hardware as a state machine.

The interface between AFU and PSL consists of five semi-independent sets of signals, the Job-, Command-, Response-, Read-Buffer- and Write-Buffer-Interface. The Job-Interface is controlled by the PSL and indicates job control and reset commands from the host to the AFU. The MMIO-Interface exposes a register view of the AFU to the host that can map this view into its virtual memory to control and monitor the AFU. The Command-Interface is controlled by the AFU, which can issue a variety of read or write commands with different side effects on the Cache Hierarchy. The remaining 3 interfaces are controlled by the PSL and play a role in completing pending commands: A read command causes the PSL to return the requested data via the Write-Buffer-Interface and signal the command completion via the Response-Interface. To perform a write operation, the PSL would read the data to be written via the Read-Buffer-Interface once required. To support an efficient implementation of the PSL, there are no ordering or uniqueness criteria on the Buffer-Interfaces. Therefore, one read command might result in several different results to be written via the Write-Buffer-Interface and transactions belonging to different pending commands are not guaranteed to execute in any order.

This complex interface between PSL and AFU enables an efficient operation and communication with the PSL, but imposes higher efforts on an AFU developer: In order to perform a single read operation three concurrent interfaces must be controlled and monitored, requiring additional states in the AFU state machine. An example provides Figure \[!REF\], that depicts the structure and state machine of a simple AFU that adds two numbers stored in host memory and writes back the result.

![](/assets/statemachine.png)


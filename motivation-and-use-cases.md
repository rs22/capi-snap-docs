# Motivation and use cases

FPGAs provide the ability to efficiently execute parallel algorithms.

In comparison to other hardware accelerators that provide parallel processing capabilities, FPGAs are very flexible in terms of the possible application scenarios \(compared to on-chip accelerators\) while being orders of magnitude more power-efficient compared to graphics cards \(GPGPU acceleration\) because of their lower clock frequency.

Traditionally however, it was very time consuming and difficult for an application programmer to implement FPGA-based accelerators. The type of 'hardware programming' that is necessary is quite different from well-known programming paradigms used in imperative languages such as C or Java. Not only is a detailed knowledge of the targeted FPGA architecture needed to optimize the VHDL- or Verilog- hardware descriptions; a lot of time is often spent to establish communication channels and management interfaces used by the consuming application.

In order to overcome these issues, IBM provides the CAPI and SNAP frameworks discussed here. They allow a broader group of developers to leverage FPGAs in the following paradigms.

## FPGA-based acceleration paradigms

### Offloading parallel computation tasks from the CPU to the FPGA

![](/assets/offload.png)

The most traditional use case for accelerators: Real-world benefits are however limited by the effective data bandwidth that the underlying bus \(e.g. PCIe\) supports. Focus on throughput instead of latency \(which the CPU is focused on, with all its caches and prefetchers\). Massively parallelizable application procedures have to be isolated to create heterogenous applications where the sequential part remains on the CPU side. In contrast to CPUs, GPUs and FPGAs are still able to significantly increase their computation power per watt over new hardware generations \(Dennard scaling\) \(actually an Nvidia marketing claim heard during TuK\).

### Transforming data before it is sent to network or storage

![](/assets/egress.png)

Depending on the hardware capabilities of the FPGA \(i.e. integrated network or storage adapters; on-chip non-volatile memory\) this paradigm can be used to reduce load on the CPU by taking over I/O operations such as communication with network devices over certain protocols \(Nvidia talk: The GPU is good at waiting for stuff because it often has lots of otherwise idling threads\). Furthermore it adds the flexibility to transform the data stream \(e.g. encryption or compression\) without incurring any additional load on the CPU.

### Transforming or filtering data received from network or storage

![](/assets/ingress.png)

Basically same as above, just the other way around.

### Aggregating data received from multiple external sources

![](/assets/funnel.png)

This scenario is especially interesting when the combined input bandwidth of all connected external sources is greater than the available bandwidth from the FPGA to the CPU \(otherwise: 'drips into the funnel'\). One application example could be to pre-filter or pre-aggregate incoming sensor data to reduce the amount of data that needs to be transferred via PCIe buses to the CPU.


## Native CAPI Interface

Even though SNAP encapsulates and hides the details of CAPI's native PSL interface, it is both instructive and interesting to learn about its structure and semantics.

As mentioned in the introduction, the PSL interface consists of six parts, the Job-, MMIO-, Command-, Read-Buffer-, Write-Buffer- and Response-Interface. Transactions happen independently on each of these interfaces, so that multiple operations might be in progress simultaneously on different interfaces.

###JobInterface
Via the job interface the AFU lifecycle is controlled by the PSL. This includes resetting the AFU, initiating jobs and monitoring the AFU's state.

A transaction on the job interface is qualified by the valid signal and contains a command that is either RESET or START. To both commands the AFU reacts by asserting the done signal once the requested operation is complete. This might be immediately after assertion of valid, as shown in the first (RESET) transaction in the figure below. Otherwise the AFU can delay done by an arbitrary number of clock cycles thereby blocking the PSL and also any host process waiting for the operation to complete.

If the command is START, it also qualified the address signal, that contains the address of the job structure the AFU is meant to work on. As long as the AFU is working, it has to assert the running signal.

![](/assets/wave_job.svg)


###MMIO Interface
The MMIO (Memory Mapped IO) interface allows the User Design to provide a register based view of the job-independent part of its state and configuration. This view can be mapped into the address space of a host process, which can thus configure the operation of the afu.
The same interface is also used by the host to access the AFU Descriptor space, which contains registers with a standardized format (as specified in section 4 of the Manual) that specify the capabilities of the afu device.

A transaction on the MMIO interface is initiated by the PSL with the assertion of the the valid line. This qualifies the register address, the read/write select, the word/doubleword select and the descriptor/user space select lines. A transaction is finished when the User Design asserts the acknowledge line. For a write transaction the data is presented on the data in signals while  valid is asserted, while a read transaction requires the data to be driven on the data out signals together with the assertion of acknowledge. The figure below illustrates both a read and a write transaction on the MMIO interface. Please note that the time from the recognition of the valid assertion to the assertion of the acknowledge line can be freely chosen by the user design. In the example below this time is one clock cycle for the read operation and zero clock cycles for the write operation.

![](/assets/wave_mmio.svg)


###Command Interface, Response Interface, Buffer Interfaces

Even though the Manual describes them as separate interfaces, the Command, Response and Buffer interfaces are closely coupled in terms of operation semantics. Together they are used by the User Design to exchange data via and manage the cache of the PSL interface. Any such operation is initiated on the Command interface that besides the operation and its parameters (address, ordering criteria, …) establishes a tag value to identify transactions on the other interfaces that are related to this particular operation. This is necessary because multiple operations can be pending at any given time.
If an operation is completed, this fact is reported on the Response interface with a response code indicating success or different types of failures.
Read or write operations are performed with the granularity of one cache line which is equivalent to 128 Bytes/1024 Bits of data. The respective Read Buffer and Write Buffer interfaces are only 512 bits wide so that a complete transfer requires 2 cycles. Each interface indicates by an address line whether the higher or lower portion of the cache line should be accessed by a transaction. There are no restrictions on the order, timing and count of transactions on the Read or Write Buffer interface relating to a single pending command. The same data might be read or written multiple times in which case the data read/written in the most recent transaction is considered valid. It might be noted that the naming of the Read Buffer and Write Buffer interface are chosen from the perspective of the PSL while commands are named according to the User Design perspective. That means, that a write operation requires the User Design to supply the intended data via the Read Buffer interface while a read operation causes the resulting data to be sent via the Write Buffer interface.
On the Command, Response, and Write Buffer interface a transaction spans exactly one clock cycle. The respective valid line qualifies the remaining signals. The Read Buffer interface also allows for a transaction to be started on every clock cycle, but each individual transaction finishes after a specific latency that the User Design requires to supply the requested data.

The signal trace in the figure below illustrates a  possible transaction sequence for a read command. Some less important signals were omitted from the trace. These include the parity lines that accompany most multibit signals to support error detection. The mapping of the signal names in the trace and those in the Manual is listed in the appendix.

![](/assets/wave_com_read.svg)

The figure below shows the progress of a write operation that uses the Read Buffer interface. Please note that the specified read data latency of 1 cycle is not counted from the clock edge of the assertion of the valid line, buf from the edge on that the User Design recognizes its assertion, which is one clock cycle later. Therefore there is a 2 cycle delay between the assertion of the valid line and the driving of the data signals. If the specified read data latency would be 3 (the only other supported value) this delay would increase to 4 cycles.

![](/assets/wave_com_write.svg)

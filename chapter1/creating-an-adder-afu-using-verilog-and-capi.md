# Creating an Adder-AFU using Verilog

* Similar example using VHDL: [https://github.com/ibm-capi/afu-walkthrough](https://github.com/ibm-capi/afu-walkthrough)

* Define job structure for AFU in consumer code

* Initialize project

  * Create a new repository or folder for your application.
    \# We divided this directory into the folders afu \(for the FPGA part\) and consumer \(C application that uses the AFU\)
  * Copy pslse/afu\_driver/verilog/top.v to afu-hello-world/afu/top.v as it will be the wrapper of your AFU.
  * Copy afu.sv and capi.sv from [https://github.com/KennethWilke/capi-parity](https://github.com/KennethWilke/capi-parity) in the same directory where you put top.v. They simplify the interface used by top.v.
    \# capi.sv categorizes the signals of top.v by grouping them into structures
    \# afu.sv is instantiated by top.v and instantiates your custom AFU with the structures defined by capi.sv instead of having all signals individually
    Replace the module name parity\_afu in afu.sv with how you want to name your module.
  * Create &lt;YOUR\_MODULE\_NAME&gt;.sv without any functionality.  
    The include here is necessary, because compile order does depend on alphabetical order and the command line tool xvlog does not analyse dependencies.

    Content of the file:
    
    ```  
    include "capi.sv"  
    import CAPI::\*;

    module &lt;YOUR\_MODULE\_NAME&gt; \(  
      input clock,  
      output timebase\_request,  
      output parity\_enabled,  
      input JobInterfaceInput job\_in,  
      output JobInterfaceOutput job\_out,  
      input CommandInterfaceInput command\_in,  
      output CommandInterfaceOutput command\_out,  
      input BufferInterfaceInput buffer\_in,  
      output BufferInterfaceOutput buffer\_out,  
      input ResponseInterface response,  
      input MMIOInterfaceInput mmio\_in,  
      output MMIOInterfaceOutput mmio\_out\);

    endmodule  
    ```

* Implement modules for MMIO communication and job lifecycle management
* Implement work element as state machine
* Consume the AFU using libcxl

## Signals and interfaces

To create own AFUs it is necessary to understand the signal flow between PSL and FPGA, as it has to be handled by the AFU developer. Therefore Kenneth Wilke implemented capi.sv and afu.sv to group the Signals of the PSL Accelerator Interface according to the interface structure in section 5 \(p. 63\) of IBM’s CAPI user guide.

The translation table  contains the mapping of original signals to the structured interface.

Our implementation is based on this structure and code and provides the modules job\_interface and mmio\_interface as wrappers for the respective interface signals.

### **JobInterface**

**Via the job interfaceTODO**

### MMIO Interface

The MMIO \(Memory Mapped IO\) interface allows the User Design to provide a register based view of the job-independent part of its state and configuration. This view can be mapped into the address space of a host process, which can thus configure the operation of the afu.

The same interface is also used by the host to access the AFU Descriptor space, which contains registers with a standardized format \(as specified in section 4 of the Manual\) that specify the capabilities of the afu device.

A transaction on the MMIO interface is initiated by the PSL with the assertion of the the valid line. This qualifies the register address, the read/write select, the word/doubleword select and the descriptor/user space select lines. A transaction is finished when the User Design asserts the acknowledge line. For a write transaction the data is presented on the data in signals while  valid is asserted, while a read transaction requires the data to be driven on the data out signals together with the assertion of acknowledge. Figure \[ref\] illustrates both a read and a write transaction on the MMIO interface. Please note that the time from the recognition of the valid assertion to the assertion of the acknowledge line can be freely chosen by the user design. In the example below this time is one clock cycle for the read operation and zero clock cycles for the write operation.

### Command Interface, Response Interface, Buffer Interfaces

Even though the Manual describes them as separate interfaces, the Command, Response and Buffer interfaces are closely coupled in terms of operation semantics. Together they are used by the User Design to exchange data via and manage the cache of the PSL interface. Any such operation is initiated on the Command interface that besides the operation and its parameters \(address, ordering criteria, …\) establishes a tag value to identify transactions on the other interfaces that are related to this particular operation. This is necessary because multiple operations can be pending at any given time.

If an operation is completed, this fact is reported on the Response interface with a response code indicating success or different types of failures.

Read or write operations are performed with the granularity of one cache line which is equivalent to 128 Bytes/1024 Bits of data. The respective Read Buffer and Write Buffer interfaces are only 512 bits wide so that a complete transfer requires 2 cycles. Each interface indicates by an address line whether the higher or lower portion of the cache line should be accessed by a transaction. There are no restrictions on the order, timing and count of transactions on the Read or Write Buffer interface relating to a single pending command. The same data might be read or written multiple times in which case the data read/written in the most recent transaction is considered valid. It might be noted that the naming of the Read Buffer and Write Buffer interface are chosen from the perspective of the PSL while commands are named according to the User Design perspective. That means, that a write operation requires the User Design to supply the intended data via the Read Buffer interface while a read operation causes the resulting data to be sent via the Write Buffer interface.

On the Command, Response, and Write Buffer interface a transaction spans exactly one clock cycle. The respective valid line qualifies the remaining signals. The Read Buffer interface also allows for a transaction to be started on every clock cycle, but each individual transaction finishes after a specific latency that the User Design requires to supply the requested data.

The signal trace in figure \[ref\] illustrates a possible transaction sequence for a read command. Some less important signals were omitted from the trace. These include the parity lines that accompany most multibit signals to support error detection. The mapping of the signal names in the trace and those in the Manual is listed in the appendix.

Figure \[ref\] shows the progress of a write operation that uses the Read Buffer interface. Please note that the specified read data latency of 1 cycle is not counted from the clock edge of the assertion of the valid line, buf from the edge on that the User Design recognizes its assertion, which is one clock cycle later. Therefore there is a 2 cycle delay between the assertion of the valid line and the driving of the data signals. If the specified read data latency would be 3 \(the only other supported value\) this delay would increase to 4 cycles.

## Compile the AFU

1. `xsc $PSLSE\_DIR/afu\_driver/src/afu\_driver.c`
2. `cp $PSLSE_DIR/afu_driver/src/libdpi.so .`
3. Or: `ln -s $PSLSE_DIR/afu_driver/src/libdpi.so .`
4. `xvlog --sv \*.sv \*.v`
5. `xelab -timescale 1ns/1ps -svlog $PSLSE_DIR/afu_driver/verilog/top.v -sv_root . -sv_lib libdpi -debug all`
6. Open XSim using `xsim -g work.top`



| Accelerator Control Interface: FPGA input |  |
| :--- | :--- |
| **ha\_pclock** | **clockorclk** |
| **ha\_jval** | **JobInterfaceInput.valid** |
| **ha\_jcom** | **JobInterfaceInput.command** |
| **ha\_jcompar** | **JobInterfaceInput.command\_parity** |
| **ha\_jea** | **JobInterfaceInput.address** |
| **ha\_jeapar** | **JobInterfaceInput.address\_parity** |
|  | **Accelerator Control Interface: FPGA output** |
| **ah\_jrunning** | **JobInterfaceOutput.running** |
| **ah\_jdone** | **JobInterfaceOutput.done** |
| **ah\_jcack** | **JobInterfaceOutput.cack** |
| **ah\_jerror** | **JobInterfaceOutput.error** |
| **ah\_jyield** | **JobInterfaceOutput.yield** |
| **ah\_tbreq** | **NOT MAPPED** |
| **ah\_paren** | **NOT MAPPED** |
|  | **Accelerator Command Interface: FPGA input** |
| **ha\_croom** | **CommandInterfaceInput.room** |
|  | **Accelerator Command Interface: FPGA output** |
| **ah\_cvalid** | **CommandInterfaceOutput.valid** |
| **ah\_ctag** | **CommandInterfaceOutput.tag** |
| **ah\_ctagpar** | **CommandInterfaceOutput.tag\_parity** |
| **ah\_com** | **CommandInterfaceOutput.command** |
| **ah\_compar** | **CommandInterfaceOutput.command\_parity** |
| **ah\_cabt** | **CommandInterfaceOutput.abt** |
| **ah\_cea** | **CommandInterfaceOutput.address** |
| **ah\_ceapar** | **CommandInterfaceOutput.address\_parity** |
| **ah\_cch** | **CommandInterfaceOutput.context\_handle** |
| **ah\_csize** | **CommandInterfaceOutput.size** |
|  | **Accelerator Buffer Interface: FPGA input** |
| **ha\_brvalid** | **BufferInterfaceInput.read\_valid** |
| **ha\_brtag** | **BufferInterfaceInput.read\_tag** |
| **ha\_brtagpar** | **BufferInterfaceInput.read\_tag\_parity** |
| **ha\_brad** | **BufferInterfaceInput.read\_address** |
| **ha\_bwvalid** | **BufferInterfaceInput.write\_valid** |
| **ha\_bwtag** | **BufferInterfaceInput.write\_tag** |
| **ha\_bwtagpar** | **BufferInterfaceInput.write\_tag\_parity** |
| **ha\_bwad** | **BufferInterfaceInput.write\_address** |
| **ha\_bwdata** | **BufferInterfaceInput.write\_data** |
| **ha\_bwpar** | **BufferInterfaceInput.write\_parity** |
|  | **Accelerator Buffer Interface: FPGA output** |
| **ah\_brlat** | **BufferInterfaceOutput.read\_latency** |
| **ah\_brdata** | **BufferInterfaceOutput.read\_data** |
| **ah\_brpar** | **BufferInterfaceOutput.read\_parity** |
|  | **PSL Response Interface: FPGA input** |
| **ha\_rvalid** | **ResponseInterface.valid** |
| **ha\_rtag** | **ResponseInterface.tag** |
| **ha\_rtagpar** | **ResponseInterface.tag\_parity** |
| **ha\_response** | **ResponseInterface.response** |
| **ha\_rcredits** | **ResponseInterface.credits** |
| **ha\_rcachestate** | **ResponseInterface.cache\_state** |
| **ha\_rcachepos** | **ResponseInterface.cache\_pos** |
|  | **Accelerator MMIO Interface: FPGA input** |
| **ha\_mmval** | **MMIOInterfaceInput.valid** |
| **ha\_mmcfg** | **MMIOInterfaceInput.cfg** |
| **ha\_mmrnw** | **MMIOInterfaceInput.read** |
| **ha\_mmdw** | **MMIOInterfaceInput.doubleword** |
| **ha\_mmad** | **MMIOInterfaceInput.address** |
| **ha\_mmadpar** | **MMIOInterfaceInput.address\_parity** |
| **ha\_mmdata** | **MMIOInterfaceInput.data** |
| **ha\_mmdatapar** | **MMIOInterfaceInput.data\_parity** |
|  | **Accelerator MMIO Interface: FPGA input** |
| **ah\_mmack** | **MMIOInterfaceOutput.ack** |
| **ah\_mmdata** | **MMIOInterfaceOutput.data** |
| **ah\_mmdatapar** | **MMIOInterfaceOutput.data\_parity** |




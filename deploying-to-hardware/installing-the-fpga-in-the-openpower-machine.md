## Installing the FPGA in the OpenPower machine

We are not responsible for damage you do to your own hardware.

1. Find a CAPI capable PCIe port which you can use to install the FPGA. For the S824L OpenPower machine we used, this information can be obtained in the 'Internal I/O subsystem' section of its [technical overview document](http://www.redbooks.ibm.com/redpapers/pdfs/redp5139.pdf) \(page 59 of the PDF in this case\).
2. Install the FPGA
3. Attach the [JTAG programmer](https://www.xilinx.com/products/boards-and-kits/hw-usb-ii-g.html) to the FPGA and connect its USB cable to an Intel-based machine that will be used to initialize the FPGA with Xilinx' hw\_server. Its activity LED will only come on after the Xilinx software has successfully established a connection to the card, so don't worry if it stays off once the systems have booted.




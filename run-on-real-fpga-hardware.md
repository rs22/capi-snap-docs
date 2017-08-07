## Running on actual hardware

### Creating a SNAP FPGA image

Once you have successfully tested your AFU in the simulator, it is time to actually deploy it to the FPGA.

To do that -- assuming you've executed `make config` before -- just run `make image` inside the hardware directory. Synthesizing the image is a memory intensive process and can take any time from 50 minutes to a couple of hours depending on the complexity of your action.

If you suspect that the model synthesis may fail due to a timing problem \(i.e. you have defined complex nested statements in your code\), set the `TIMING_LABLIMIT` environment variable to a large negative value \([default](https://github.com/open-power/snap/blob/master/hardware/setup/snap_build.tcl#L29): -250 ps\) before starting the build. This avoids build failures at a late stage in the process, but may produce images that are not suited for production use. However, you might have success trying them out in a lab environment.

Once the build process has successfully finished, you can find the resulting bitstream files in the `build/Image` folder under the `hardware` directory. The version ending in \*.bit can be flashed using the JTAG programmer; the \*.bin file is meant for the capi-flash-script \(see the below sections\).

### Installing the FPGA in the OpenPower machine

We are not responsible for damage you do to your own hardware.

1. Find a CAPI capable PCIe port which you can use to install the FPGA. For the S824L OpenPower machine we used, this information can be obtained in the 'Internal I/O subsystem' section of its [technical overview document](http://www.redbooks.ibm.com/redpapers/pdfs/redp5139.pdf) \(page 59 of the PDF in this case\).
2. Install the FPGA
3. Attach the [JTAG programmer](https://www.xilinx.com/products/boards-and-kits/hw-usb-ii-g.html) to the FPGA and connect its USB cable to an Intel-based machine that will be used to initialize the FPGA with Xilinx' hw\_server. Its activity LED will only come on after the Xilinx software has successfully established a connection to the card, so don't worry if it stays off once the systems have booted.

### Initializing the FPGA using a JTAG Programmer

The official documentation on this topic can be found [here](https://github.com/open-power/snap/blob/master/hardware/doc/Bitstream_flashing.md).

After the FPGA was installed, usually the Linux OS on the OpenPower-machine will not detect it as a CAPI-enabled device. This is because the image that comes pre-installed on the _factory_ partition of the FPGA doesn't support CAPI; instead a suitable image needs to be flashed onto the _user_ partition by using the external USB programmer.

Once this procedure is completed and the FPGA is listed under `/dev/cxl`, new images can be flashed directly from the OpenPower host using the CAPI tooling provided by IBM. This is described in the next section.

If you would like to set up a headless Vivado installation on the Intel machine connected to the JTAG programmer, you can instead choose to install a lighter weight version of Vivado that includes the hardware server tool `hw_server`.

In any case, the process accessing the programmer \(either Vivado or `hw_server`\) might need to run with root privileges. Open Vivado's Hardware Manager and 'open' the target device either on the local server or remotely. Once it has successfully connected to the FPGA, the programmer's activity LED should come on.  
You will now need a device image \(\*.bit\) that includes at least the PSL component -- one way to obtain such an image is to run `make image` for one of the SNAP examples or your own action \(it is not necessary to enable SNAP's `FACTORY_IMAGE` switch\).

Because the user partition's contents will be cleared on each power cycle of the OpenPower machine if the card was not yet detected as a CAPI device, a certain procedure is required to initialize it: The image needs to be flashed _after_ the system was powered on, but _before_ the OS performs the PCI Express walk.

Therefore, set up a ping to the host OS and trigger a reboot \(a normal OS reboot should suffice, although sometimes an ipmi power cycle is necessary\). Once the ping is lost, start the programming process in Vivado. There should be enough time to fully program the card before the host OS has completed booting.

### Programming the FPGA directly from the OpenPower Host

Once the FPGA is detected as a CAPI-device it is a lot easier and faster to flash new images onto it. The tool necessary can be obtained from GitHub:

```
git clone https://github.com/ibm-capi/capi-utils
cd capi-utils
make
make install
```

Afterwards, start the tool like this and follow the instructions:

```
capi-flash-script my_image.bin
```

### Testing the breadth-first search action on hardware

Assuming you have built the bitstream \(using `make image`\) and set up the FPGA as described above, the `hls_bfs` action should by now be programmed onto the card.

You will now need a version of SNAP on the Power8 server:

1. Because now the 'real' version of libcxl is needed \(compared to the 'mock' version used earlier by the PSL Simulation Engine\), install it \(e.g. on Ubuntu\):

   ```
   apt install libcxl-dev
   ```

2. Clone SNAP, build the actions and tooling

   ```
   git clone https://github.com/open-power/snap
   cd snap/software
   make
   cd ../actions
   make
   ```

3. Before you can use the action for the first time, it needs to be detected by SNAP. Use the `snap_maint` tool to trigger this:

   ```
   cd ${SNAP_ROOT}/software/tools
   ./snap_maint -v
   ```

4. Now you can execute the breadth-first search on hardware!

   ```
   cd ${SNAP_ROOT}/actions/hls_bfs/sw
   ./snap_bfs -v
   ```




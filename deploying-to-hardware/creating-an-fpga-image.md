## Creating a SNAP FPGA image

Once you have successfully tested your AFU in the simulation, it is time to actually deploy it to the FPGA.

To do that -- assuming you've executed `make config` before -- just run `make image` inside the hardware directory. Synthesizing the image is a memory intensive process and can take any time from 50 minutes to a couple of hours depending on the complexity of your action.

If you suspect that the model synthesis may fail due to a timing problem \(i.e. you have defined complex nested statements in your code\), set the `TIMING_LABLIMIT` environment variable to a large negative value \([default](https://github.com/open-power/snap/blob/master/hardware/setup/snap_build.tcl#L29): -250 ps\) before starting the build to avoid it to fail at a late stage in the process.

Once the build process has successfully completed, you can find the resulting bitstream files in the build/Image folder under the hardware directory. The version ending in \*.bit can be flashed using the JTAG programmer; the \*.bin file is meant for the capi-flash-script \(see the following sections\).


## Creating a SNAP FPGA image

Once you have successfully tested your AFU in the simulation, it is time to actually deploy it to the FPGA.

To do that -- assuming you've executed `make config` before -- just run `make image` inside the hardware directory. Synthesizing the image is a memory intensive process and can take any time from 50 minutes to a couple of hours depending on the complexity of your action.

So get yourself a coffee. Then take some time to [legitimately slack off](https://xkcd.com/303/). Then go home and come back tomorrow.

If you find out that the model synthesis failed due to a timing problem, set the `TIMING_LABLIMIT` environment variable to a large negative value ([default](https://github.com/open-power/snap/blob/master/hardware/setup/snap_build.tcl#L29): -250 ps). Repeat.
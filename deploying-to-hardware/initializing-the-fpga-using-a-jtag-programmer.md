## Initializing the FPGA using a JTAG Programmer

The official documentation on this topic can be found [here](https://github.com/open-power/snap/blob/master/hardware/doc/Bitstream_flashing.md).

After the FPGA was installed, usually the Linux OS on the OpenPower-machine will not detect it as a CAPI-enabled device. This is because the image that comes pre-installed on the *factory* partition of the FPGA doesn't support CAPI; instead a suitable image needs to be flashed onto the *user* partition by using the external USB programmer.

Once this procedure is completed and the FPGA is available under `/dev/cxl`, new images can be flashed directly from the OpenPower host using the CAPI tooling provided by IBM. This is described in the next chapter.

If you would like to avoid to set up a full Vivado installation on the Intel machine connected to the JTAG programmer, you can instead choose a lighter weight version of Vivado that only includes the hardware server tool `hw_server`.

In any case, the process accessing the programmer might need to run with root privileges. Once it has successfully connected to the FPGA, the programmer's activity LED should come on.

Open Vivado's Hardware Manager and 'open' the target device either on the local server or remotely. You will now need a device image (*.bit) that includes at least the PSL component -- one way to obtain such an image is to run `make image` for one of the SNAP examples (it is not necessary to enable the `FACTORY_IMAGE` switch).

Because the user partition's contents will be cleared on each power cycle of the OpenPower machine if the card was not yet detected as a CAPI device, a certain procedure is required to initialize it: The image needs to be flashed *after* the system has booted, but *before* the OS performs the PCI Express walk.

Therefore, set up a ping to the host OS and trigger a reboot (a normal OS reboot should suffice, although sometimes an ipmi power cycle is necessary). Once you lose the ping, start the programming process in Vivado. There should be enough time to fully program the card before the host OS has completed booting.
# Installing Xilinx Vivado

* We had issues with Version 2017.1 / use 2016.4 instead
* A license is necessary
* We installed the Design Edition, which includes support for the needed FPGAs
* Because a license is \(in our case\) bound to a specific MAC address, Vivado checks your network adapters to verify the license
  * On Linux, we needed to rename our interface to eth0 to make it work
* We also installed Vivado on headless Ubuntu servers used for synthesis and JTAG programming, using X-forwarding and with the help of these instructions: [https://github.com/dirkcgrunwald/wiserzynq-docker/wiki/Vivado-Docker](https://github.com/dirkcgrunwald/wiserzynq-docker/wiki/Vivado-Docker)
* Place the following line into your .bashrc: `source /opt/Xilinx/Vivado/2016.4/bin/settings64.sh`




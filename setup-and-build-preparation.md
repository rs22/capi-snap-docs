## Setup and build preparation

In order to work with SNAP and CAPI, the most important requirement is to install Xilinx Vivado. In our setup we used Ubuntu 16.04, but although not officially supported by the needed Vivado version, Ubuntu 17.04 should also be possible.

### Download and installation

Let's first download, install and configure the software and repositories we need.

#### 1. Xilinx Vivado 

Xilinx Vivado is used to synthesize and layout our actions for the FPGA and contains HLS to convert C++ to hardware code as well as xsim (Vivado Simulator) for simulating without an FPGA. Currently version 2016.4 is the latest supported release of Vivado. As the HL WebPACK Edition may not include support for your FPGA-chip we recommend the HL Design Edition. The free 30-day evaluation should work as well ([Comparison of different Vivado Editions](https://www.xilinx.com/products/design-tools/vivado.html#buy)). You can find the downloads [here](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/2016-4.html).

1. Download and install Xilinx Vivado
2. If not using the evaluation version, obtain a Vivado license-file (.lic)
3. Add Vivado to your path by sourcing its settings file:
```
source /opt/Xilinx/Vivado/2016.4/settings64.sh
```
If you do not want to repeat this after every reboot, do it automatically on startup by appending it to your bashrc:
```
echo "source /opt/Xilinx/Vivado/2016.4/settings64.sh" >> ~/.bashrc
```

#### 2. Libcxl and PSL checkpoint

The libcxl is the library that is used by the host communicate with PSL devices. You can install it using the package manager:

```
sudo apt-get install libcxl-dev
```

On the FPGA the Power Service Layer (PSL) manages the communication with the host. This includes translating memory addresses, handling interrupts and virtualizing AFUs if needed. The PSL will be part of the circuit on the FPGA and is, synthesised and layouted for your specific FPGA, provided as a Vivado checkpoint (.dcp) file. This file contains a snapshot at a certain build step and has to be integrated when building your action. Even when you just want to simulate, you still need a checkpoint file (we recommend the one for the FlashGT card).

Please download the PSL checkpoint for your card [here](https://www-355.ibm.com/systems/power/openpower/tgcmDocumentRepository.xhtml?aliasId=CAPI). 

#### 3. Power Service Layer Simulation Engine

While building for hardware takes a lot of time, building for simulation is comparably fast. Therefore, even if you have a real chip, you want to use simulation of an FPGA in development as well.
Because simulation of hardware is computationally intense, only the actions and not the PSL should be simulated. The PSLSE implements the PSL in software and connects to the (locally hosted) simulation server with the desired action. The host application then communicates to the (locally hosted) PSLSE server instead of an FPGA. 

Clone the PSLSE with
```
git clone https://github.com/ibm-capi/pslse
```

#### 4. SNAP

TODO

You will need the following components for development \(we've tested this setup on Ubuntu 16.04\):

1. Xilinx Vivado in your PATH variable  
   Can be achieved by calling \`source &lt;xilinx\_root&gt;/Vivado/2016.4/settings64.sh\`

2. A Xilinx Vivado license file \(.lic\)  
   Have the environment variable \`XILINXD\_LICENSEFILE\` point to it.

3. The pre-routed PSL design checkpoint file \(\*.dcp\) for the FPGA you are using. Can be obtained from [IBM](https://www-355.ibm.com/systems/power/openpower/tgcmDocumentRepository.xhtml?aliasId=CAPI).  
   Have the environment variable \`PSL\_DCP\` point to it. Depending on the FPGA type, set the `FPGACARD` environment variable to either `FGT` or `KU3`.

4. A version of the PSL Simulation Engine: `git clone https://github.com/ibm-capi/pslse`  
   Have the environment variable \`PSLSE\_ROOT\` point to it.

5. Finally, SNAP itself: `git clone https://github.com/open-power/snap`

### Preparing to build the breadth-first search action

Now, in order to tell SNAP that you would like to build the breadth-first search action (called `hls_bfs` because it's implemented using the C-like Vivado HLS language), set another environment variable called `ACTION_ROOT` to the absolute path of the `actions/hls_bfs` directory inside your local SNAP repository. Follow the same procedure if you want to build other examples or your own actions.

You can now configure SNAP and verify all your environment variables by changing into SNAP's `hardware` directory and calling `source snap_settings`. This script will end by presenting you a summary of the current configuration. If everything looks fine, you can now generate the Vivado project files needed to simulate and build the action by executing `make config` inside the `hardware` directory.

Whenever you change one of the `snap_settings` parameters, you will have to execute `make clean config` to re-generate the project files (be careful with cleaning, however! All previously built model and image files will be lost!)


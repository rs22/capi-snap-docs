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

Now you still need to clone SNAP itself:

```
git clone https://github.com/open-power/snap
```

### Preparing the build environment

Before you can use SNAP for building you have to specify where the components you just installed are. Add Vivado to the path by sourcing the Vivado settings script, point environment variables to the components you just downloaded (PSLSE is optional here). Then source the hardware settings script in the SNAP repository:

```
source /opt/Xilinx/Vivado/2016.4/settings64.sh
export XILINXD_LICENSE_FILE=<pointer to Xilinx license>
export PSL_DCP=<CAPI PSL Checkpoint file (b_route_design.dcp)>
export PSLSE_ROOT=<path to PSLSE>
source <path to snap>/hardware/snap_settings
```

As you have to do this everytime you open a new terminal window in which you want to use SNAP, we recommend writing it into a script. You may also call that script in your `~/.bashrc` which gets executed every time you open a new terminal window. 

When executed (or sourced), `snap_settings` displays the settings for your build:

```
=======================================================
== SNAP SETUP                                        ==
=======================================================
=====Checking Xilinx Vivado:===========================
Path to vivado          is set to: /opt/Xilinx/Vivado/2016.4/bin/vivado
Vivado version          is set to: Vivado v2016.4 (64-bit)
=====CARD variables====================================
FPGACARD                is set to: "FGT"
FPGACHIP                is set to: "xcku060-ffva1156-2-e"
PSL_DCP                 is set to: "/home/balthasar/cards/FGT/portal_20170413/b_route_design.dcp"
=====SNAP PATH variables===============================
SNAP_ROOT               is set to: "/home/balthasar/repos/snap"
ACTION_ROOT             is set to: "/home/balthasar/repos/snap/actions/hdl_example"
=====SNAP simulation variables=========================
SIMULATOR               is set to: "xsim"
=====SNAP function variables===========================
NUM_OF_ACTIONS          is set to: "1"
SDRAM_USED              is set to: "FALSE"
NVME_USED               is set to: "FALSE"
ILA_DEBUG               is set to: "FALSE"
FACTORY_IMAGE           is set to: "FALSE"
```

Depending on which card you use you may have to change `$FPGACARD` and `$FPGACHIP`:

```
export FPGACARD=<FGT or KU3>
export FPGACARD=<your chip identifier>
```

While `xsim` is the default Simulator and  `SNAP_ROOT` is set automatically, `ACTION_ROOT` and the SNAP function Variables have to be choosen based on the action you want to build.


### Preparing to build the breadth-first search action

Now, in order to tell SNAP that you would like to build the breadth-first search action (called `hls_bfs` because it's implemented using the C-like Vivado HLS language), set `ACTION_ROOT` to the absolute path of the `actions/hls_bfs` directory inside your local SNAP repository:

```
export ACTION_ROOT=/home/balthasar/repos/snap/actions/hls_bfs
```

Because `hls_bfs` does not use DRAM or NVMe storage (see [](https://github.com/open-power/snap/tree/master/actions/hls_bfs/doc)), we leave `SDRAM_USED` and `NVME_USED` to `FALSE`.

If you have not done so already, change into the `hardware` directory of your snap repository (`cd $SNAP_ROOT/hardware`). If all the settings are correct (you can check that by calling `./snap_settings`), you can generate the Vivado project files needed to build and simulate the action with:
```
make config
```

You can follow the same procedure if you want to build other examples or your own actions.
Whenever you change one of the `snap_settings` parameters, you will have to execute `make clean config` to re-generate the project files (be careful with cleaning, however, as all previously built model and image files will be lost!)

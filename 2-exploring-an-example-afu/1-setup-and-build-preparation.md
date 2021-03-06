## Setup and build preparation

In order to work with SNAP and CAPI, the most important requirement is to install Xilinx Vivado. In our setup we used Ubuntu 16.04, but although not officially supported by the needed Vivado version, working with Ubuntu 17.04 should also be possible.

### Download and installation

Let's first download, install and configure the software and repositories we need on our development machine.

#### 1. Xilinx Vivado 

Xilinx Vivado is used to synthesize and layout our actions for the FPGA and contains HLS to convert C++ to hardware code as well as xsim (Vivado Simulator) for simulating without an FPGA. Currently version 2016.4 is the latest supported release of Vivado. As the HL WebPACK Edition may not include support for your FPGA-chip we recommend the HL Design Edition. The free 30-day evaluation should work as well ([Comparison of different Vivado Editions](https://www.xilinx.com/products/design-tools/vivado.html#buy)). You can find the downloads [here](https://www.xilinx.com/support/download/index.html/content/xilinx/en/downloadNav/vivado-design-tools/2016-4.html).

1. Download and install Xilinx Vivado
2. If not using the evaluation version, obtain a Vivado license-file (.lic)
3. Add Vivado to your path by sourcing its settings file:
```bash
~ $ source /opt/Xilinx/Vivado/2016.4/settings64.sh
```
If you do not want to repeat this after every reboot, do it automatically on startup by appending it to your bashrc:
```bash
~ $ echo "source /opt/Xilinx/Vivado/2016.4/settings64.sh" >> ~/.bashrc
```

<div class="brainbox"><span>
Because a Vivado license is usually bound to a specific MAC address, Vivado scans your network adapters to verify the license. However, it only looks for interfaces named eth*. As a result, we had to rename our interface to eth0 on Ubuntu to make it work.
</span></div>

#### 2. PSL checkpoint

On the FPGA, the Power Service Layer (PSL) manages the communication with the host. This includes translating memory addresses, handling interrupts and virtualizing AFUs if needed. 

<div class="brainbox"><span>
The PSL will be part of the circuit on the FPGA and is synthesised and layouted for your specific FPGA, provided as a Vivado checkpoint (.dcp) file. This file contains a snapshot at a certain build step and has to be integrated when building your action. Even when you just want to simulate, you still need a checkpoint file (we recommend the one for the FlashGT card).
</span></div>

Please download the PSL checkpoint for your card [here](https://www-355.ibm.com/systems/power/openpower/tgcmDocumentRepository.xhtml?aliasId=CAPI). 

#### 3. Power Service Layer Simulation Engine

PSLSE is optional and only needed for simulation, but while building for hardware takes a lot of time, building for simulation is comparably fast. Therefore you may want to use simulation even if you have a real chip.
Because simulation of hardware is computationally intense, only the actions and not the PSL should be simulated. The PSLSE implements the PSL in software and connects to the (locally hosted) simulation server with the desired action. The host application then communicates to the (locally hosted) PSLSE server instead of an FPGA. 

Clone the PSLSE with
```bash
~ $ git clone https://github.com/ibm-capi/pslse
```

#### 4. SNAP

Now you still need to clone SNAP itself:

```bash
~ $ git clone https://github.com/open-power/snap
```

The repository is split into three folders. The `software` and `hardware` folder contain the structures necessary for building software and hardware, while the `actions` folder contains various examples that are ready to be built. Each action is either prefixed by `hdl_` or `hls_`, indicating whether it is defined in a hardware description language or high level HLS code. Inside each action, the hardware specification lives in `hw` while the host application is implemented in `sw`.

### Preparing the build environment

Before you can use SNAP for building you have to specify where the components you just installed are. Add Vivado to the path by sourcing the Vivado settings script, point environment variables to the components you just downloaded (omitting `PSLSE_ROOT` if you did not download PSLSE). Then source the hardware settings script in the SNAP repository:

```bash
source /opt/Xilinx/Vivado/2016.4/settings64.sh
export XILINXD_LICENSE_FILE=<pointer to Xilinx license>
export PSL_DCP=<CAPI PSL Checkpoint file (b_route_design.dcp)>
export PSLSE_ROOT=<path to PSLSE>
source <path to snap>/hardware/snap_settings
```
<p class="figure-caption">Template for a script to setup the SNAP environment.
</p>

As you have to do this every time you open a new terminal window in which you want to use SNAP, we recommend writing it into a script. You may also call that script in your `~/.bashrc` which gets executed automatically once you open a new terminal window.

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
<p class="figure-caption">Example output of the snap_settings script.
</p>

Depending on which card you use you may have to change `$FPGACARD` and `$FPGACHIP`:

```bash
~ $ export FPGACARD=<FGT or KU3>
~ $ export FPGACARD=<your chip identifier>
```

While `xsim` is the default Simulator and  `SNAP_ROOT` is set automatically, `ACTION_ROOT` and the SNAP function variables (`SDRAM_USED`, `NVME_USED`) have to be chosen based on the action you want to build.

### Selecting the action to build

As an example, let us built one of the provided actions. In order to tell SNAP that you would like to build the breadth-first search action (called `hls_bfs` because it's implemented using the C-like Vivado HLS language), set `ACTION_ROOT` to the absolute path of the `actions/hls_bfs` directory inside your local SNAP repository:

```bash
~ $ export ACTION_ROOT=$SNAP_ROOT/actions/hls_bfs
```

Because `hls_bfs` does not use DRAM or NVMe storage (see the [documentation](https://github.com/open-power/snap/tree/master/actions/hls_bfs/doc)), we leave `SDRAM_USED` and `NVME_USED` to `FALSE`.

Now, if you have not done so already, change into the `hardware` directory of your snap repository (`cd $SNAP_ROOT/hardware`). If all the settings are correct (you can check that by calling `./snap_settings`), you can generate the Vivado project files needed to build and simulate the action with:

```bash
$SNAP_ROOT/hardware $ make config
```

You can follow the same procedure if you want to build other examples or your own actions.
Whenever you change one of the `snap_settings` parameters, you will have to execute `make clean config` to re-generate the project files. Be careful with cleaning, however, as all previously built model and image files will be lost!

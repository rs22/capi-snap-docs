## Setup and build preparation

In order to work with SNAP and CAPI, the most important requirement is to install Xilinx Vivado. Currently version 2016.4 is the latest supported release of Vivado; the free version will not be sufficient to build images for the FPGA chipset currently supported by SNAP.

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

Now, in order to tell SNAP that you would like to build the breadth-first search action (called `hls_bfs` because it's implemented using the C-like Vivado HLS language), set another environment variable called `ACTION_ROOT` to the `actions/hls_bfs` directory inside your local SNAP repository. Follow the same procedure if you want to build other examples or your own actions.

You can now configure SNAP and verify all your environment variables by changing into SNAP's `hardware` directory and calling `source snap_settings`. This script will end by presenting you a summary of the current configuration. If everything looks fine, you can now generate the Vivado project files needed to simulate and build the action by executing `make config` inside the `hardware` directory.


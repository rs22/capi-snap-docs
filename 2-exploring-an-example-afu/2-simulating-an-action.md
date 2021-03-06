## Simulating an action

The simulation step is useful if you want to quickly test if your host application can successfully communicate with the AFU. You won't need access to the POWER/FPGA hardware and can generate a simulation model in a much shorter time than a hardware bitstream.

However, the simulation speed is, understandably, much slower than what the real hardware will achieve. Furthermore, although you can trace the flow of different binary signals in Vivado, simulation often isn't useful for debugging HLS AFUs: The HLS code will be converted into a VHDL/Verilog blocks that are quite hard to match to the HLS code. How to debug HLS code will be explained in the following chapter.

### Simulation explained

SNAP supports several simulators, but in this document we will assume `xsim` which is included in the Xilinx Vivado Suite and therefore available in all setups.

After SNAP is configured, a simulation model of the user design can be built with:

```bash
$SNAP_ROOT/hardware $ make model
```

All HDL sources both from SNAP and the user design are compiled into a simulation model and the simulator configuration is written into a shell script. The simulation of large hardware designs is computationally very expensive. Therefore the resulting model does not include the PSL nor any part of the host side hardware. Nevertheless these components are essential for a user application to access the AFU under test. The solution is the PSLSE \(Power Service Layer Simulation Environment\) server.

<div class="brainbox"><span>
The PSLSE implements a higher level and thus faster model of the internal CAPI components. It provides a special version of the libcxl, that does not attempt to interact with any CAPI device, but connects to the PSLSE server to access the virtual device it provides. In the same way, the PSL is not simulated as a hardware component, but is only a placeholder, whose behavior on the signal level is controlled by the PSLSE server via a network connection.
</span></div>

The necessary setup of the PSLSE server and its connection to the simulator is done automatically by a script called `run_sim`. It creates an environment where the regular libcxl is replaced by the PSLSE version. Thus all applications started in this environment will automatically interact with the simulated AFU. The script supports different modes of operation:

```bash
$SNAP_ROOT/hardware/sim $ ./run_sim -explore
```

```bash
$SNAP_ROOT/hardware/sim $ ./run_sim -app ${SNAP_ROOT}/software/tools/snap_maint -arg '-i 3 -vvv'
```

If it is run without the `-app` parameter, it opens a new terminal session with the correctly set up environment to run applications on the simulated hardware. If `-app` is given, the specified application is executed with the arguments optionally specified by `-arg` in the simulation environment without creating an interactive session. The `-explore` option enables an action exploration step before the specified application or terminal session is started. This step is necessary to discover the configuration of the AFU and allow a user application to correctly identify it. This is equivalent to executing the `snap_maint` tool manually before starting the user application.

### Simulating the breadth-first search AFU

1. Given that you've already run `make config` as described in the previous section, build the simulation image with 
  ```bash
  $SNAP_ROOT/hardware $ make model
  ```
2. Before you can test the model, you will also need to build the consumer application \(that will communicate with the simulator through PSLSE\):
   ```bash
   $SNAP_ROOT/actions $ make
   ```
3. As described above, start the the simulation itself:
   ```bash
   $SNAP_ROOT/hardware/sim $ ./run_sim -explore
   ```
4. A new terminal window will open which you can use to launch the test application. The environment in this terminal window is carefully prepared by `run_sim` which includes the current working directory. Please do not change it as any test application not run from within there can not use the simulated environment.

   ```bash
   /path/to/sim/env $ $SNAP_ROOT/actions/hls_bfs/sw/snap_bfs -v
   ```
   
  If the output ends with
  ```bash
  Visiting node (0): 0, 3, 5, 1, 8, 4, 7, 9, End. Cnt = 8
  ```
  the breadth-first search has successfully completed! You can try different configurations by using the options described in `$ snap_bfs --help`

5. Once you're done testing, close the simulation terminal window to shut down the simulator to avoid it consuming CPU cycles unnecessarily.


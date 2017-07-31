## Simulating an action

SNAP supports several simulators, but in this document we will assume `xsim` which is included in the Xilinx Vivado Suite and therefore available in all setups.

After SNAP is configured, a simulation model of the user design can be built with:

```bash
$SNAP_ROOT/hardware> make model
```

All HDL sources both from SNAP and the user design are compiled into a simulation model and the simulator configuration is written into a shell script. The simulation of large hardware designs is computationally very expensive. Therefore the resulting model does not include the PSL nor any part of the host side hardware. Nevertheless these components are essential for a user application to access the AFU under test. The solution is the PSLSE \(Power Service Layer Simulation Environment\) server, that implements a higher level and thus faster model of the internal CAPI components. It provides a special version of the libcxl, that does not attempt to interact with any cxl device, but connects to the PSLSE server to access the virtual device it provides. In the same way, the PSL is not simulated as a hardware component, but is only a placeholder, whose behavior on the signal level is controlled by the PSLSE server via a network connection.

The necessary setup of the PSLSE server and its connection to the simulator is done automatically by a script called `run_sim`. It creates an environment where the regular libcxl is replaced by the PSLSE version. Thus all applications started in this environment will automatically interact with the simulated AFU. The script supports different modes of operation:

```
${SNAP_ROOT}/hardware/sim> ./run_sim -explore
```

```
${SNAP_ROOT}/hardware/sim> ./run_sim -app ${SNAP_ROOT}/software/tools/snap_maint -arg '-i 3 -vvv'
```

If it is run without the `-app` parameter, it opens a new terminal session with the correctly set up environment to run applications on the simulated hardware. If `-app` is given, the specified application is executed with the arguments optionally specified by `-arg` in the simulation environment without creating an interactive session. The `-explore` option enables an action exploration step before the specified application or terminal session is started. This step is necessary to discover the configuration of the AFU and allow a user application to correctly identify it. This is equivalent to executing the `snap_maint` tool manually before starting the user application.


To focus on what may be new for software developers — i.e. the hardware part — we create a minimal program to test our blowfish AFU. It should encrypt and then decrypt some hardcoded example data with a hardcoded key.

To seperate this from the bigger example that you will also find in our github repository, we create the directory `hls_blowfish/sw_minimal`.

First, we copy a Makefile from one of the examples \(e.g. `snap/actions/hls_bfs/sw/Makefile`\) into our `hls_blowfish/sw_minimal` directory and modify it to build blowfish.

```Makefile
# Finding $SNAP_ROOT
ifndef SNAP_ROOT
# check if we are in sw folder of an action (three directories below snap root)
ifneq ("$(wildcard ../../../ActionTypes.md)","")
SNAP_ROOT=$(abspath ../../../)
else
$(info You are not building your software from the default directory (/path/to/snap/actions/<action_name>/sw) or specified a wrong $$SNAP_ROOT.)
$(error Please source /path/to/snap/hardware/snap_settings.sh or set $$SNAP_ROOT manually.)
endif
endif


snap_blowfish_minimal: snap_blowfish_minimal.o

projs += snap_blowfish_minimal

include $(SNAP_ROOT)/actions/software.mk
```

CODE



To bring it to the structure of a snap example, we need more commmand line options, file reading and printed information. This can be found in \[link\]. Also, software impl. to compare \[link\].


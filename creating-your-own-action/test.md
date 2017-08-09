## Creating a minimal host software

To focus on what may be new for software developers — i.e. the hardware part — we create a minimal program to test our blowfish AFU. It should encrypt and then decrypt some hardcoded example data with a hardcoded key.

To seperate this from the bigger example that you will also find in our github repository, we create the directory `hls_blowfish/sw_minimal`.

### Makefile

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
<p class="figure-caption">Makefile<a href="url">link text</a>
</p>



### Host program

The Makefile builds `hls_blowfish/sw_minimal/snap_blowfish_minimal.c` which we created by copying and modifing `snap/actions/hls_bfs/sw/snap_bfs.c`. In the following paragraphs we explain the most important parts. View the complete code [here]().

The `prepare_blowfish` method takes a pointer to a empty SNAP job and two empty job structs (one for in and on for out) and the parameters for the job. It then initializes the `snap_addr` variables that the AFU will read and write and fills the input job struct with the parameters. This all is used to create the SNAP job.

```c
static void snap_prepare_blowfish(struct snap_job *job,
        uint32_t mode_in,
        uint32_t data_length_in,
        blowfish_job_t *bjob_in,
        blowfish_job_t *bjob_out,
        const void *addr_in,
        uint16_t type_in,
        void *addr_out,
        uint16_t type_out)
{
    //initialize SNAP addresses
    snap_addr_set(&bjob_in->input_data, addr_in, data_length_in,
		  type_in, SNAP_ADDRFLAG_ADDR | SNAP_ADDRFLAG_SRC);

    snap_addr_set(&bjob_in->output_data, addr_out, data_length_in,
		  type_out, SNAP_ADDRFLAG_ADDR | SNAP_ADDRFLAG_DST |
		  SNAP_ADDRFLAG_END );

    // fill job struct
    bjob_in->mode = mode_in;
    bjob_in->data_length = data_length_in;

    // Here sets the 108byte MMIO settings input.
    // We have input parameters.
    snap_job_set(job, bjob_in, sizeof(*bjob_in),
		 bjob_out, sizeof(*bjob_out));
}
```


### Further comments

Our minimal implementation lacks error handling and is inflexible due to hardcoded information. To bring it to the level of a SNAP example, it would need some more features, including commmand line options, file reading and debug output. You can find the code for the real example in the same repository in the `sw` folder in [snap_blowfish.c](https://github.com/ldurdel/hls_blowfish/blob/master/sw/snap_blowfish.c).
If you clone the [repository](https://github.com/ldurdel/hls_blowfish), it can be used like the original examples. This also includes a software implementation of the blowfish algorithm in `hls_blowfish/sw/action_blowfish.c`. Though not necessary for the hardware implementation itself, maintain such a separate implementation of the AFU is often a good idea. Besides being a reference for testing the hardware implementation correctness, is also serves as a baseline for performance analyses.
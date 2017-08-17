## Creating a minimal host software

To focus on what may be new for software developers — i.e. the hardware part — we create a minimal program to test our blowfish AFU. It should encrypt and then decrypt some hardcoded example data with a hardcoded key.

To separate this from the bigger example that you will also find in our GitHub repository, we create the directory `hls_blowfish/sw_minimal`.

### Makefile

First, we copy a Makefile from one of the examples \(e.g. `snap/actions/hls_bfs/sw/Makefile`\) into our `hls_blowfish/sw_minimal` directory and modify it to build blowfish.

```makefile
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
<p class="figure-caption">Makefile, edited for readability. View the original <a href="https://github.com/ldurdel/hls_blowfish/blob/master/sw_minimal/Makefile">here</a>.
</p>

### Host program

The Makefile builds `hls_blowfish/sw_minimal/snap_blowfish_minimal.c` which we created by copying and modifing `snap/actions/hls_bfs/sw/snap_bfs.c`. In the following paragraphs we explain the most important parts. View the complete code [here](https://github.com/ldurdel/hls_blowfish/blob/master/sw_minimal/snap_blowfish_minimal.c).

The `prepare_blowfish` method takes a pointer to an empty SNAP job, two empty job structs (one for in and one for out) and the parameters for the job. It then initializes the `snap_addr` variables that the AFU for the input and output data and fills the input job struct with its parameters. The 'out' job struct is not really needed in our use case, but is part of the method signature for combining everything into a SNAP job.

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
    // Initialize SNAP addresses
    snap_addr_set(&bjob_in->input_data, addr_in, data_length_in,
		  type_in, SNAP_ADDRFLAG_ADDR | SNAP_ADDRFLAG_SRC);

    snap_addr_set(&bjob_in->output_data, addr_out, data_length_in,
		  type_out, SNAP_ADDRFLAG_ADDR | SNAP_ADDRFLAG_DST |
		  SNAP_ADDRFLAG_END );

    // Fill job struct
    bjob_in->mode = mode_in;
    bjob_in->data_length = data_length_in;

    // Fill the 108byte MMIO settings input with our input parameters
    snap_job_set(job, bjob_in, sizeof(*bjob_in),
		 bjob_out, sizeof(*bjob_out));
}
```
<p class="figure-caption">Method for preparing a job based on input and output structures that contain SNAP addresses.
</p>

For how to execute an action, let's have a look at the method `blowfish_cipher` that can be used for either en- or decryption &mdash; depending on the `mode` supplied. A job is first created and then is executed via `snap_action_sync_execute_job(action, &job, timeout);`.

```c
static int blowfish_cipher(struct snap_action *action,
               int mode, unsigned long timeout,
               const uint8_t *ibuf,
               unsigned int in_len,
               uint8_t *obuf)
{
    struct snap_job job;
    blowfish_job_t bjob_in;

    if (in_len % 8 != 0 || in_len <= 0) {
        printf("err: data to en- or decrypt has to be multiple of "
               "8 bytes and at least 8 bytes long!\n");
        return -EINVAL;
    }

    snap_prepare_blowfish(&job, mode, in_len, &bjob_in, NULL,
                  (void *)ibuf, SNAP_ADDRTYPE_HOST_DRAM,
                  (void *)obuf, SNAP_ADDRTYPE_HOST_DRAM);

    snap_action_sync_execute_job(action, &job, timeout);

    return 0;
}
```
<p class="figure-caption">Method for encryption and decryption with a previously set key.
</p>

Now we define some example data and create the method `blowfish_test`, that first sets the key, then encrypts data with said key and decrypts it again.

```c
blowfish_cipher(action, MODE_ENCRYPT, timeout,
                 example_plaintext, sizeof(example_plaintext),
                 example_encrypted);
```
<p class="figure-caption">Example on how our test method calls the ciphering method for encryption.
</p>

Now we can allocate the card in our main method, select the action we want to use (as there can be multiple on the card) and execute our tests. After that we need to de-select the action and free the card to make it usable for others again.

```c
int main()
{
    int card_no = 0;
    unsigned long timeout = 10000;
    struct snap_card *card = NULL;
    struct snap_action *action = NULL;
    snap_action_flag_t action_irq = (SNAP_ACTION_DONE_IRQ | SNAP_ATTACH_IRQ);
    
    // Hardcoded card number for simplicity
    card = snap_card_alloc_dev("/dev/cxl/afu0.0s", SNAP_VENDOR_ID_IBM, SNAP_DEVICE_ID_SNAP);
    
    if (card == NULL) {
        fprintf(stderr, "err: failed to open card %u: %s\n",
            card_no, strerror(errno));
            return 1;
    }

    action = snap_attach_action(card, BLOWFISH_ACTION_TYPE, action_irq, 60);
    blowfish_test(action, timeout);

    snap_detach_action(action);
    snap_card_free(card);
    exit(EXIT_SUCCESS);

    return 0;
}
```
<p class="figure-caption">The main method of our minimal blowfish test.
</p>

Again, for the complete implementation, please refer to the [code on Github](https://github.com/ldurdel/hls_blowfish/blob/master/sw_minimal/snap_blowfish_minimal.c).

Now we can call `make` in `hls_blowfish/sw_minimal` to build our software part and use it like described in _Simulating an action_ or _Running on actual hardware_. If you use the default directory name (`sw`), building is done automatically when calling `make model` on your development machine. Please remember that, when using real hardware, you want to build the software part on the POWER machine to avoid cross compiling.

<div class="brainbox"><span>
Due to CAPI reading only full cache lines (128 byte), you have to allocate addresses that you want to share 128 byte aligned. Also, when choosing the size of our test data, only full 64 bytes arrived at the FPGA. When using less data, only zeros arrived.
</span></div>

### Further comments

Our minimal implementation lacks error handling and is inflexible due to hardcoded information. To bring it to the level of a SNAP example, it would need some more features, including commmand line options, file reading and debug output. You can find the code for the real example in the same repository located in the `sw` folder: [snap_blowfish.c](https://github.com/ldurdel/hls_blowfish/blob/master/sw/snap_blowfish.c).
If you clone the whole [repository](https://github.com/ldurdel/hls_blowfish), it can be used like the original examples with both, `sw` and `sw_minimal`, containing working host programs. We also &mdash; like it is done in other examples &mdash; included a software implementation of the blowfish algorithm in `hls_blowfish/sw/action_blowfish.c`. Although not necessary for the hardware implementation itself, maintaining such a separate implementation of the AFU is often a good idea. Besides being a reference for testing the hardware implementation correctness, is also serves as a baseline for performance analyses.
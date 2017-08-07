## Creating your own action

This chapter will walk you through the complete process of creating an FPGA-based implementation of encrypting and decrypting with the [blowfish cipher](https://en.wikipedia.org/wiki/Blowfish_\(cipher\)).

We chose it, because it is easy to implement and quite fast when encrypting or decrypting. Only the key-preprocessing when changing the key is expensive. Note that this implementation is not intended for real life use with confidential information but rather a demonstration for SNAP. To effectivly use the parallelization capabilities of the FPGA, we deviced for [Electronic Codebook](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Electronic_Codebook_.28ECB.29) as blockmode, because each block can be en- and decrypted individually. In a real-world scenario, we would recommend [CTR](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Counter_.28CTR.29).

As the key pre-processing is quite expensive, we want to be able to handle large data streams with one pre-processing. Therefore we decided to include three operations in our action:

1. Set the key
1. Encrypt data with the current key
1. Decrypt data with the current key

Each call to our action will be one of these operations. 
TODO correct this: When calling an action, the parameters are shared as a struct in host memory: the AFU gets the pointer to this struct and can thereby access the parameters. How 

We create an empty repository ourside of the SNAP folder structure and call it `hls_blowfish`. To use the existing build structures, we divide the directory into `hw`, `sw` and `include`. While files execlusivly needed for the hardware or software definition are placed in their respective directory, the `include` directory contains the header files important for both.

In `hls_blowfish/include/action_blowfish.h` we specify the our action. By copying `snap/actions/hls_bfs/include/action_bfs.h` and deleting and renaming action-specific parts, we get the following frame:

```
#ifndef __ACTION_BLOWFISH_H__
#define __ACTION_BLOWFISH_H__

#include <snap_types.h>

#ifdef __cplusplus
extern "C" {
#endif

#define BLOWFISH_ACTION_TYPE 0xXXXXXX

#ifndef CACHELINE_BYTES
#define CACHELINE_BYTES 128
#endif
typedef struct blowfish_job {
    // TODO
} blowfish_job_t;

#ifdef __cplusplus
}
#endif
#endif	/* __ACTION_BLOWFISH_H__ */
```

According to https://github.com/open-power/snap/blob/master/ActionTypes.md, we choose an action type number from the "free for experimental use" range:

`#define BLOWFISH_ACTION_TYPE 0x00000108
`

For the job struct, we need to specify the operation, the address of the input data and its length and where to write output data. We do not need a field where the length of the output data can be written, as it is defined by operation and input length.

TODO data has to be multiple of 128 bytes

```
#define MODE_SET_KEY 0
#define MODE_ENCRYPT 1
#define MODE_DECRYPT 2

typedef struct blowfish_job {
    struct snap_addr input_data;
    struct snap_addr output_data; // not needed for MODE_SET_KEY
    uint32_t mode;
    uint32_t data_length;
} blowfish_job_t;
```
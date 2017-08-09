To focus on what may be new for software developers — i.e. the hardware part — we create a minimal program to test our blowfish AFU. It should encrypt and then decrypt some hardcoded example data with a hardcoded key.

First, we copy a Makefile from one of the examples \(e.g. `snap/actions/hls_bfs/sw/Makefile`\) into our `hls_blowfish/sw` directory.

Makefile

CODE



To bring it to the structure of a snap example, we need more commmand line options, file reading and printed information. This can be found in \[link\]. Also, software impl. to compare \[link\].


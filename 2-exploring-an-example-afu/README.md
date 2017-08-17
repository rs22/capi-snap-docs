## Exploring an Example AFU

This chapter will walk you through the process of compiling one of the examples provided with SNAP; its different components will be explained in the context of the SNAP action development workflow.

As an example action, we chose the breadth-first-search (bfs) implementation.

The general steps are:

1. Set up the SNAP environment by installing/cloning its dependencies. Make them known to SNAP's Makefiles by means of environment variables
1. Select an example and build a simulation model for it. Then run the simulation and explore the design on virtual hardware.
1. Build a hardware image ready to be written to a real FPGA. Setup and configure the FPGA card and experience the design in full speed.


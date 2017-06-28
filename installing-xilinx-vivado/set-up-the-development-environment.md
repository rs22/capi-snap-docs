# Set up the Development Environment

* Goal: Build the hdl\_example for FGT card and start simulation
* cd &lt;workspace&gt;
* git clone https://github.com/ibm-capi/pslse
* cd pslse && make && cd ..

* Download the PSL Checkpoint \(b_route_design.dcp\) from https://slack-redir.net/link?url=https%3A%2F%2Fwww-355.ibm.com%2Fsystems%2Fpower%2Fopenpower%2FtgcmDocumentRepository.xhtml%3FaliasId%3DCAPI

* \`export XILINXD\_LICENSE_FILE=&lt;path\_to&gt;_/Xilinx.lic\`
* \`export PSL_DCP=/&lt;path\_to&gt;_/b\_route\_design.dcp\`
* `export PSLSE`_`ROOT=<path`_`to>/pslse`
* git clone https://github.com/open-power/snap
* cd snap/software && make
* cd ../hardware && source snap\_settings.sh
* `make config model`




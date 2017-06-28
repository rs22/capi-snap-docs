# Set up the Development Environment

* Goal: Build the hdl\_example for FGT card and start simulation
* cd &lt;workspace&gt;
* git clone [https://github.com/ibm-capi/pslse](https://github.com/ibm-capi/pslse)
* cd pslse && make && cd ..

* Download the PSL Checkpoint \(b\_route\_design.dcp\) from https://www-355.ibm.com/systems/power/openpower/tgcmDocumentRepository.xhtml?aliasId=CAPI 

* \`export XILINXD\_LICENSE_FILE=&lt;path\_to&gt;_/Xilinx.lic\`

* \`export PSL_DCP=/&lt;path\_to&gt;_/b\_route\_design.dcp\`
* `export PSLSEROOT=<pathto>/pslse`
* git clone [https://github.com/open-power/snap](https://github.com/open-power/snap)
* cd snap/software && make
* cd ../hardware && source snap\_settings.sh
* `make config model`




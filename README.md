### Anouncement: this git repo is obsolete, since from Robert C. Nelson's kernel 5.14.x
### mikrobus patches are added to the kernel. These mikrobus patches are placed here:
https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/am33x-v5.13/drivers/mikrobus

### The original patches are placed in the mikrobus-staging directory:
https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/am33x-v5.13/drivers/mikrobus-staging

Extention to Robert C. Nelson's Embedded Debian Beagle Board kernel:

https://github.com/ZoranStojsavljevic/BeagleBoard-Workshop-Examples/tree/master/Kernel_Porting_Guide

	$ git clone https://github.com/RobertCNelson/bb-kernel.git
	$ cd bb-kernel
	$ git remote show origin
	$ git checkout am33x-v5.14
	$ git branch ## to verify the execution of the last command
	$ git describe
	$ ./build_kernel.sh

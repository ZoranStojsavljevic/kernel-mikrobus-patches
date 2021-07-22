### Kernel Mikrobus Patches for kernel 5.12.13-bone23

Extention to Robert C. Nelson's Embedded Debian Beagle Board kernel:

https://github.com/ZoranStojsavljevic/BeagleBoard-Workshop-Examples/tree/master/Kernel_Porting_Guide

Modified string to original, presented above:

	$ git clone https://github.com/RobertCNelson/bb-kernel.git
	$ cd bb-kernel
	$ git remote show origin
	$ git checkout 5.12.13-bone23
	$ git branch ## to verify the execution of the last command
	$ git describe
	$ ./build_kernel.sh

This is a support to 5.12.13-bone23 in order to create test framework for MikroBUS driver:

https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/5.12.13-bone23

The bash script part which does the addition trick (keeping intact rcn's bb_kernel
repo) is schetched here (just a sketch for mikrobus patches, to give an idea):

	patch_kernel () {
		### --- Since the mikrobus patches are not there, add them to the patch set ---
		cd "${DIR}/"

		${git_bin} clone https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches.git
		cd "${DIR}/"kernel-mikrobus-patches/
		${git_bin} checkout 5.12.13-bone23
		cp -Rfp "${DIR}/"kernel-mikrobus-patches/drivers/* "${DIR}/"patches/drivers/

		ls -al "${DIR}/"patches/drivers
		ls -al "${DIR}/"patches/drivers/ti

		### --- End mikrobus patch set addition ---

		### --- Start processing patch set ---
		cd "${DIR}/KERNEL" || exit

		export DIR
		/bin/bash -e "${DIR}/patch.sh" || { ${git_bin} add . ; exit 1 ; }

		if [ ! -f "${DIR}/.yakbuild" ] ; then
			if [ ! "${RUN_BISECT}" ] ; then
				${git_bin} add --all
				${git_bin} commit --allow-empty -a -m "${KERNEL_TAG}${BUILD} patchset"
			fi
		fi

		cd "${DIR}/" || exit
	}

### Modifications of patch.sh in order to add additional patches and enable kernel 5.12.13-bone23 to work

	drivers () {
		#https://github.com/raspberrypi/linux/branches
		#exit 2
		dir 'RPi'
		dir 'drivers/ar1021_i2c'
		dir 'drivers/spi'
		dir 'drivers/tps65217'

		dir 'drivers/ti/overlays'
		dir 'drivers/ti/cpsw'
		dir 'drivers/ti/serial'
		dir 'drivers/ti/tsc'
		dir 'drivers/ti/gpio'
		dir 'drivers/greybus'
		dir 'drivers/mikrobus'
		dir 'drivers/serdev'
		dir 'drivers/fb_ssd1306'
		dir 'drivers/bluetooth'
	}

### Kernel 5.12.13-bone23 Status

	Status: kernel 5.12.13-bone23 always works (Stable - Green)

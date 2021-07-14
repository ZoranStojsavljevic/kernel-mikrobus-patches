### Kernel Mikrobus Patches for kernel 5.12.x

Addition to latest Robert C. Nelson Beagle Board kernels in order to create test
framework for mikrobus driver.

For adding the additional patches, presented here:

https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/main/drivers/

The bash script part which does the addition trick (keeping intact rcn's bb_kernel
repo) is schetched here (just a sketch for mikrobus patches, to give an idea):

	patch_kernel () {
		### --- Since the mikrobus patches are not there, add them to the patch set ---
		cd "${DIR}/"

		${git_bin} clone https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches.git
		cd "${DIR}/"kernel-mikrobus-patches/
		${git_bin} checkout 5.12
		cp -Rfp "${DIR}/"kernel-mikrobus-patches/drivers/* "${DIR}/"patches/drivers/

		rm -rf "${DIR}/"patches/drivers/iio/
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

### Modifications of patch.sh in order to add additional patches and enable kernel 5.12.x to work

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
		## dir 'drivers/iio'
		dir 'drivers/fb_ssd1306'
		dir 'drivers/bluetooth'
}

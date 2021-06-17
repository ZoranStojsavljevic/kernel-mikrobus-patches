### Kernel Mikrobus Patches

Addition to latest Robert C. Nelson Beagle Board kernels in order to create test
framework for mikrobus driver.

For adding the mikrobus patches, presented here:

https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/main/mikrobus-patches/actual-mikrobus-patches

The bash script part which does the addition trick (keeping intact rcn's bb_kernel repo):

	patch_kernel () {
		### --- Since the mikrobus patches are not there, add them to the patch set ---
		cd "${DIR}/"
		mkdir patches/drivers/mikrobus

		## cd "${DIR}/"
		${git_bin} clone https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches.git
		cp kernel-mikrobus-patches/mikrobus-patches/actual-mikrobus-patches/* \
			/home/vuser/projects/kernel.bb/bb-kernel/patches/drivers/mikrobus

		## Mikrobus patch should be added to the patch set!
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

### NEXT TO DO: Overlay patches are NOT included (MUST include them in order mikrobus driver to work)

This should be the very next task:

	[1] To track and include in the repo overlay patches in the separate directory;
	[2] To include in the above script also the overlay patches handling!

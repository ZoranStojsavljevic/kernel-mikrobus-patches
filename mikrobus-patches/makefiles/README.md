### Applying the patch produces an error!
https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/blob/main/mikrobus-patches/overlays/0001-ARM-DT-Enable-symbols-when-CONFIG_OF_OVERLAY-is-used.patchhttps://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/blob/main/mikrobus-patches/overlays/0001-ARM-DT-Enable-symbols-when-CONFIG_OF_OVERLAY-is-used.patch

	Starting patch.sh
	dir: aufs
	Applying: merge: aufs-kbuild
	Applying: merge: aufs-base
	Applying: merge: aufs-mmap
	Applying: merge: aufs-standalone
	Applying: merge: aufs
	dir: wpanusb
	Applying: merge: wpanusb: https://github.com/statropy/wpanusb
	Applying: Add WPANUSB driver
	dir: wireless_regdb
	Applying: Add wireless-regdb regulatory database file
	dir: drivers/ti/firmware
	Applying: Add AM335x CM3 Power Managment Firmware
	dir: soc/ti/beagleboard_dtbs
	Applying: Add BeagleBoard.org DTBS: v5.11.x
	dir: backports/greybus
	Applying: backports: greybus: from: linux.git
	dir: backports/wlcore
	Applying: backports: wlcore: from: linux.git
	dir: backports/spidev
	Applying: backports: spidev: from: linux.git
	dir: backports/pinctrl
	Applying: backports: pinctrl: from: linux.git
	dir: backports/pru_rproc
	Applying: backports: pru_rproc: from: linux.git
	dir: RPi
	Applying: Overlays: Port RPi Overlay building
	dir: drivers/ar1021_i2c
	Applying: ar1021_i2c: invert/swap and offset options
	dir: drivers/spi
	Applying: NFM: spi: spidev: allow use of spidev in DT
	dir: drivers/tps65217
	Applying: HACK: tps65217_pwr_but
	dir: drivers/ti/cpsw
	Applying: cpsw: search for phy
	Applying: cpsw: fix undefined function with PM disabled

### The error by patch.sh script, not yet resolved!

	dir: drivers/ti/overlays
	Applying: ARM: DT: Enable symbols when CONFIG_OF_OVERLAY is used
	error: patch failed: arch/arm/boot/dts/Makefile:1
	error: arch/arm/boot/dts/Makefile: patch does not apply
	Patch failed at 0001 ARM: DT: Enable symbols when CONFIG_OF_OVERLAY is used
	hint: Use 'git am --show-current-patch=diff' to see the failed patch
	When you have resolved this problem, run "git am --continue".
	If you prefer to skip this patch, run "git am --skip" instead.
	To restore the original branch and stop patching, run "git am --abort".

### Workaround

arch/arm/boot/dts/Makefile

Replaced by prepached file: 

https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/blob/main/mikrobus-patches/makefiles/dts/Makefile

arch/arm/boot/Makefile

Replaced by prepached file:

https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/blob/main/mikrobus-patches/makefiles/Makefile

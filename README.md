### Kernel Mikrobus Patches for kernel 5.13.3-bone12

Extention to Robert C. Nelson's Embedded Debian Beagle Board kernel:

https://github.com/ZoranStojsavljevic/BeagleBoard-Workshop-Examples/tree/master/Kernel_Porting_Guide

Modified string to original, presented above:

	$ git clone https://github.com/RobertCNelson/bb-kernel.git
	$ cd bb-kernel
	$ git remote show origin
	$ git checkout 5.13.3-bone12
	$ git branch ## to verify the execution of the last command
	$ git describe
	$ ./build_kernel.sh

This is a support to 5.13.3-bone12 in order to create test framework for MikroBUS driver:

https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/5.13.3-bone12

The bash script part which does the addition trick (keeping intact rcn's bb_kernel
repo) is sketched here (just a sketch for mikrobus patches, to give an idea):

	patch_kernel () {
		### --- Since the mikrobus patches are not there, add them to the patch set ---
		cd "${DIR}/"

		${git_bin} clone https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches.git
		cd "${DIR}/"kernel-mikrobus-patches/
		${git_bin} checkout 5.13.3-bone12
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

### Modifications of patch.sh in order to add additional patches and enable kernel 5.13.3-bone12 to work

	drivers () {
		#https://github.com/raspberrypi/linux/branches
		#exit 2
		dir 'RPi'
		dir 'drivers/ar1021_i2c'
		dir 'drivers/spi'
		dir 'drivers/tps65217'

		dir 'drivers/ti/overlays'	## Added from kernel-mikrobus-patches github repo
		dir 'drivers/ti/cpsw'
		dir 'drivers/ti/serial'
		dir 'drivers/ti/tsc'
		dir 'drivers/ti/gpio'
		dir 'drivers/greybus'
		dir 'drivers/mikrobus'		## Added from kernel-mikrobus-patches github repo
		dir 'drivers/serdev'
		dir 'drivers/fb_ssd1306'
	}

### defconfig
https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/blob/5.13.3-bone12/defconfig

Used as .config to build Robert C. Nelson's Embedded Debian Beagle PB Board kernel.

### Issues

Seems that i2c structure in include/linux/i2c.h had some struct members' changes!

	struct i2c_board_info {
		char            type[I2C_NAME_SIZE];
		unsigned short  flags;
		unsigned short  addr;
		const char	*dev_name;
		void            *platform_data;
		struct device_node *of_node;
		struct fwnode_handle *fwnode;
	==>>	const struct software_node *swnode;	 <<== new memeber of the structure
	==>>	const struct property_entry *properties; <<== obsolete member of the structure
		const struct resource *resources;
		unsigned int    num_resources;
		int             irq;
	};

Thus, to formally solve this issue the lines 572 and 753 were commented out!

	572:	// if (dev->properties)
	573:	//	i2c->properties = dev->properties;

To solve error below:

	  CC [M]  fs/isofs/rock.o
	drivers/misc/mikrobus/mikrobus_core.c: In function 'mikrobus_device_register':
	drivers/misc/mikrobus/mikrobus_core.c:573:28: error: 'struct i2c_board_info' has no member named 'properties'
	  573 |                         i2c->properties = dev->properties;
	      |                            ^~
	In file included from ./include/linux/device.h:15,
	                 from ./include/linux/w1.h:9,
	                 from drivers/misc/mikrobus/mikrobus_core.c:20:
	drivers/misc/mikrobus/mikrobus_core.c: In function 'mikrobus_port_gb_register':
	drivers/misc/mikrobus/mikrobus_core.c:894:37: warning: format '%lu' expects argument of type 'long unsigned int', but argument 3 has type 'size_t' {aka 'unsigned int'} [-Wformat=]
	  894 |                 dev_err(&port->dev, "failed to parse manifest, size %lu\n",
	      |                                     ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	./include/linux/dev_printk.h:19:22: note: in definition of macro 'dev_fmt'
	   19 | #define dev_fmt(fmt) fmt
	      |                      ^~~
	drivers/misc/mikrobus/mikrobus_core.c:894:17: note: in expansion of macro 'dev_err'
	  894 |                 dev_err(&port->dev, "failed to parse manifest, size %lu\n",
	      |                 ^~~~~~~
	drivers/misc/mikrobus/mikrobus_core.c:894:71: note: format string is defined here
	  894 |                 dev_err(&port->dev, "failed to parse manifest, size %lu\n",
	      |                                                                     ~~^
	      |                                                                       |
	      |                                                                       long unsigned int
	      |                                                                     %u
	drivers/misc/mikrobus/mikrobus_core.c:840:27: warning: unused variable 'desc' [-Wunused-variable]
	  840 |         struct gpio_desc *desc;
	      |                           ^~~~
	make[3]: *** [scripts/Makefile.build:273: drivers/misc/mikrobus/mikrobus_core.o] Error 1
	make[3]: *** Waiting for unfinished jobs....
	  CC [M]  fs/isofs/export.o
	make[2]: *** [scripts/Makefile.build:516: drivers/misc/mikrobus] Error 2
	make[1]: *** [scripts/Makefile.build:516: drivers/misc] Error 2
	make: *** [Makefile:1847: drivers] Error 2
	make: *** Waiting for unfinished jobs....
	  CC [M]  fs/isofs/joliet.o

### Kernel 5.13.3-bone12 Status

	Status: kernel 5.13.3-bone12 works randomly (UnStable - Red)

Intermittent typical log crash follows:

	Starting kernel ...

	[    0.000000] Booting Linux on physical CPU 0x0
	[    0.000000] Linux version 5.13.3-bone12 (vuser@fedora33-ssd) (arm-linux-gnueabi-gcc (GCC) 11.1.0, GNU ld (GNU Binutils) 2.36.1) #1 PREEMPT Mon Jul 26 05:18:11 CEST 2021
	[    0.000000] CPU: ARMv7 Processor [413fc082] revision 2 (ARMv7), cr=50c5387d
	[    0.000000] CPU: PIPT / VIPT nonaliasing data cache, VIPT aliasing instruction cache
	[    0.000000] OF: fdt: Machine model: TI AM335x PocketBeagle
	[    0.000000] Memory policy: Data cache writeback
	[    0.000000] efi: UEFI not found.
	[    0.000000] cma: Reserved 48 MiB at 0x9c800000
	[    0.000000] Zone ranges:
	[    0.000000]   Normal   [mem 0x0000000080000000-0x000000009fefffff]
	[    0.000000]   HighMem  empty
	[    0.000000] Movable zone start for each node
	[    0.000000] Early memory node ranges
	[    0.000000]   node   0: [mem 0x0000000080000000-0x000000009fefffff]
	[    0.000000] Initmem setup node 0 [mem 0x0000000080000000-0x000000009fefffff]
	[    0.000000] CPU: All CPU(s) started in SVC mode.
	[    0.000000] AM335X ES2.1 (sgx neon)
	[    0.000000] Built 1 zonelists, mobility grouping on.  Total pages: 129666
	[    0.000000] Kernel command line: console=ttyO0,115200n8 bone_capemgr.uboot_capemgr_enabled=1 root=/dev/mmcblk0p1 ro rootfstype=ext4 rootwait coherent_pool=1M net.ifnames=0 rng_core.default_quality=100
	[    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
	[    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
	[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
	[    0.000000] Memory: 450540K/523264K available (10240K kernel code, 1620K rwdata, 3504K rodata, 1024K init, 324K bss, 23572K reserved, 49152K cma-reserved, 0K highmem)
	[    0.000000] random: get_random_u32 called from __kmem_cache_create+0x1b/0x290 with crng_init=0
	[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
	[    0.000000] ftrace: allocating 43318 entries in 85 pages
	[    0.000000] ftrace: allocated 85 pages with 4 groups
	[    0.000000] trace event string verifier disabled
	[    0.000000] rcu: Preemptible hierarchical RCU implementation.
	[    0.000000] 	Trampoline variant of Tasks RCU enabled.
	[    0.000000] 	Rude variant of Tasks RCU enabled.
	[    0.000000] 	Tracing variant of Tasks RCU enabled.
	[    0.000000] rcu: RCU calculated value of scheduler-enlistment delay is 25 jiffies.
	[    0.000000] NR_IRQS: 16, nr_irqs: 16, preallocated irqs: 16
	[    0.000000] IRQ: Found an INTC at 0x(ptrval) (revision 5.0) with 128 interrupts
	[    0.000000] TI gptimer clocksource: always-on /ocp/interconnect@44c00000/segment@200000/target-module@31000
	[    0.000002] sched_clock: 32 bits at 24MHz, resolution 41ns, wraps every 89478484971ns
	[    0.000024] clocksource: dmtimer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
	[    0.000372] TI gptimer clockevent: 24000000 Hz at /ocp/interconnect@48000000/segment@0/target-module@40000
	[    0.001971] Console: colour dummy device 80x30
	[    0.002018] WARNING: Your 'console=ttyO0' has been replaced by 'ttyS0'
	[    0.002029] This ensures that you still see kernel messages. Please
	[    0.002037] update your kernel commandline.
	[    0.002083] Calibrating delay loop... 995.32 BogoMIPS (lpj=1990656)
	[    0.020453] pid_max: default: 32768 minimum: 301
	[    0.020669] LSM: Security Framework initializing
	[    0.020815] Yama: becoming mindful.
	[    0.020988] AppArmor: AppArmor initialized
	[    0.021007] TOMOYO Linux initialized
	[    0.021093] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
	[    0.021114] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
	[    0.022191] CPU: Testing write buffer coherency: ok
	[    0.022266] CPU0: Spectre v2: using BPIALL workaround
	[    0.023410] Setting up static identity map for 0x80100000 - 0x80100054
	[    0.023602] rcu: Hierarchical SRCU implementation.
	[    0.024631] EFI services will not be available.
	[    0.025243] devtmpfs: initialized
	[    0.049682] VFP support v0.3: implementor 41 architecture 3 part 30 variant c rev 3
	[    0.050034] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
	[    0.050064] futex hash table entries: 256 (order: -1, 3072 bytes, linear)
	[    0.055177] pinctrl core: initialized pinctrl subsystem
	[    0.056569] NET: Registered protocol family 16
	[    0.060325] DMA: preallocated 1024 KiB pool for atomic coherent allocations
	[    0.060881] audit: initializing netlink subsys (disabled)
	[    0.061819] thermal_sys: Registered thermal governor 'fair_share'
	[    0.061835] thermal_sys: Registered thermal governor 'bang_bang'
	[    0.061847] thermal_sys: Registered thermal governor 'step_wise'
	[    0.062662] cpuidle: using governor menu
	[    0.068509] audit: type=2000 audit(0.060:1): state=initialized audit_enabled=0 res=1
	[    0.088096] hw-breakpoint: debug architecture 0x4 unsupported.
	[    0.106751] raid6: skip pq benchmark and using algorithm neonx8
	[    0.106787] raid6: using neon recovery algorithm
	[    0.108172] iommu: Default domain type: Translated 
	[    0.110867] SCSI subsystem initialized
	[    0.111170] usbcore: registered new interface driver usbfs
	[    0.111225] usbcore: registered new interface driver hub
	[    0.111288] usbcore: registered new device driver usb
	[    0.111681] pps_core: LinuxPPS API ver. 1 registered
	[    0.111698] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
	[    0.111723] PTP clock support registered
	[    0.112756] Advanced Linux Sound Architecture Driver Initialized.
	[    0.113395] NetLabel: Initializing
	[    0.113412] NetLabel:  domain hash size = 128
	[    0.113421] NetLabel:  protocols = UNLABELED CIPSOv4 CALIPSO
	[    0.113502] NetLabel:  unlabeled traffic allowed by default
	[    0.114417] clocksource: Switched to clocksource dmtimer
	[    0.197746] VFS: Disk quotas dquot_6.6.0
	[    0.197844] VFS: Dquot-cache hash table entries: 1024 (order 0, 4096 bytes)
	[    0.198703] AppArmor: AppArmor Filesystem Enabled
	[    0.220936] NET: Registered protocol family 2
	[    0.221144] IP idents hash table entries: 8192 (order: 4, 65536 bytes, linear)
	[    0.222127] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 4096 bytes, linear)
	[    0.222208] TCP established hash table entries: 4096 (order: 2, 16384 bytes, linear)
	[    0.222258] TCP bind hash table entries: 4096 (order: 2, 16384 bytes, linear)
	[    0.222302] TCP: Hash tables configured (established 4096 bind 4096)
	[    0.222470] UDP hash table entries: 256 (order: 0, 4096 bytes, linear)
	[    0.222499] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes, linear)
	[    0.222703] NET: Registered protocol family 1
	[    0.226720] RPC: Registered named UNIX socket transport module.
	[    0.226744] RPC: Registered udp transport module.
	[    0.226753] RPC: Registered tcp transport module.
	[    0.226762] RPC: Registered tcp NFSv4.1 backchannel transport module.
	[    0.226777] NET: Registered protocol family 44
	[    1.018417] random: fast init done
	[    1.198244] Initialise system trusted keyrings
	[    1.198663] workingset: timestamp_bits=14 max_order=17 bucket_order=3
	[    1.204043] zbud: loaded
	[    1.206673] NFS: Registering the id_resolver key type
	[    1.206734] Key type id_resolver registered
	[    1.206745] Key type id_legacy registered
	[    1.206862] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
	[    1.206877] nfs4flexfilelayout_init: NFSv4 Flexfile Layout Driver Registering...
	[    1.207167] fuse: init (API version 7.33)
	[    1.297347] xor: automatically using best checksumming function   neon      
	[    1.297381] Key type asymmetric registered
	[    1.297392] Asymmetric key parser 'x509' registered
	[    1.297458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 246)
	[    1.297668] io scheduler mq-deadline registered
	[    1.301009] Serial: 8250/16550 driver, 6 ports, IRQ sharing disabled
	[    1.304242] sdhci: Secure Digital Host Controller Interface driver
	[    1.304266] sdhci: Copyright(c) Pierre Ossman
	[    1.304387] sdhci-pltfm: SDHCI platform and OF driver helper
	[    1.307293] libphy: Fixed MDIO Bus: probed
	[    1.308025] CAN device driver interface
	[    1.308505] usbcore: registered new interface driver smsc95xx
	[    1.308957] usbcore: registered new interface driver uas
	[    1.309018] usbcore: registered new interface driver usb-storage
	[    1.309053] usbcore: registered new interface driver ums-alauda
	[    1.309085] usbcore: registered new interface driver ums-cypress
	[    1.309117] usbcore: registered new interface driver ums-datafab
	[    1.309152] usbcore: registered new interface driver ums_eneub6250
	[    1.309196] usbcore: registered new interface driver ums-freecom
	[    1.309230] usbcore: registered new interface driver ums-isd200
	[    1.309263] usbcore: registered new interface driver ums-jumpshot
	[    1.309296] usbcore: registered new interface driver ums-karma
	[    1.309330] usbcore: registered new interface driver ums-onetouch
	[    1.309374] usbcore: registered new interface driver ums-realtek
	[    1.309408] usbcore: registered new interface driver ums-sddr09
	[    1.309448] usbcore: registered new interface driver ums-sddr55
	[    1.309483] usbcore: registered new interface driver ums-usbat
	[    1.311016] i2c /dev entries driver
	[    1.311986] softdog: initialized. soft_noboot=0 soft_margin=60 sec soft_panic=0 (nowayout=0)
	[    1.312007] softdog:              soft_reboot_cmd=<not set> soft_active_on_boot=0
	[    1.312970] cpuidle: enable-method property 'ti,am3352' found operations
	[    1.313430] ledtrig-cpu: registered to indicate activity on CPUs
	[    1.313895] hid: raw HID events driver (C) Jiri Kosina
	[    1.314155] usbcore: registered new interface driver usbhid
	[    1.314170] usbhid: USB HID core driver
	[    1.315521] drop_monitor: Initializing network drop monitor service
	[    1.316359] NET: Registered protocol family 10
	[    1.320187] Segment Routing with IPv6
	[    1.320326] mip6: Mobile IPv6
	[    1.320344] NET: Registered protocol family 17
	[    1.320376] can: controller area network core
	[    1.320463] NET: Registered protocol family 29
	[    1.320771] Key type dns_resolver registered
	[    1.320790] mpls_gso: MPLS GSO support
	[    1.321065] ThumbEE CPU extension supported.
	[    1.321085] Registering SWP/SWPB emulation handler
	[    1.321096] omap_voltage_late_init: Voltage driver support not added
	[    1.321337] PM: Cannot get wkup_m3_ipc handle
	[    1.321964] registered taskstats version 1
	[    1.321996] Loading compiled-in X.509 certificates
	[    1.322186] zswap: loaded using pool lzo/zbud
	[    1.322961] Key type ._fscrypt registered
	[    1.322983] Key type .fscrypt registered
	[    1.322992] Key type fscrypt-provisioning registered
	[    1.326322] Btrfs loaded, crc32c=crc32c-generic, zoned=yes
	[    1.326589] AppArmor: AppArmor sha1 policy hashing enabled
	[    1.344918] remoteproc remoteproc0: wkup_m3 is available
	[    1.352300] 44e09000.serial: ttyS0 at MMIO 0x44e09000 (irq = 18, base_baud = 3000000) is a 8250
	[    2.333803] printk: console [ttyS0] enabled
	[    2.348845] pinctrl-single 44e10800.pinmux: 142 pins, size 568
	[    2.361630] omap_wdt: OMAP Watchdog Timer Rev 0x01: initial timeout 60 sec
	[    2.371613] omap_rtc 44e3e000.rtc: registered as rtc0
	[    2.376800] omap_rtc 44e3e000.rtc: setting system clock to 2000-01-01T00:00:00 UTC (946684800)
	[    2.390188] 48022000.serial: ttyS1 at MMIO 0x48022000 (irq = 25, base_baud = 3000000) is a 8250
	[    2.401574] 48024000.serial: ttyS2 at MMIO 0x48024000 (irq = 26, base_baud = 3000000) is a 8250
	[    2.412881] omap_i2c 4802a000.i2c: bus 1 rev0.11 at 400 kHz
	[    2.431276] OMAP GPIO hardware version 0.1
	[    2.440596] omap-mailbox 480c8000.mailbox: omap mailbox rev 0x400
	[    2.450628] omap_i2c 4819c000.i2c: bus 2 rev0.11 at 400 kHz
	[    2.460476] 481a8000.serial: ttyS4 at MMIO 0x481a8000 (irq = 39, base_baud = 3000000) is a 8250
	[    2.469490] serial serial0: tty port ttyS4 registered
	[    2.481554] c_can_platform 481cc000.can: c_can_platform device registered (regs=(ptrval), irq=42)
	[    2.492843] c_can_platform 481d0000.can: c_can_platform device registered (regs=(ptrval), irq=43)
	[    2.513676] omap_rng 48310000.rng: Random Number Generator ver. 20
	[    2.520140] random: crng init done
	[    2.527601] 8<--- cut here ---
	[    2.530690] Unhandled fault: external abort on non-linefetch (0x1008) at 0xe0266000
	[    2.538382] pgd = eee6ef21
	[    2.541104] [e0266000] *pgd=82c1d811, *pte=4a326653, *ppte=4a326453
	[    2.547423] Internal error: : 1008 [#1] PREEMPT THUMB2
	[    2.552588] Modules linked in:
	[    2.555659] CPU: 0 PID: 7 Comm: kworker/u2:0 Not tainted 5.13.3-bone12 #1
	[    2.562481] Hardware name: Generic AM33XX (Flattened Device Tree)
	[    2.568604] Workqueue: events_unbound deferred_probe_work_func
	[    2.574481] PC is at sysc_probe+0x99a/0xf3c
	[    2.578689] LR is at pwrdm_unlock+0x17/0x34
	[    2.582903] pc : [<c061d116>]    lr : [<c0119f03>]    psr: 40000033
	[    2.589197] sp : c196d848  ip : 00000000  fp : c2c03540
	[    2.594443] r10: c0b6bd08  r9 : 00000001  r8 : c0b6b848
	[    2.599689] r7 : 00000000  r6 : c1cf1c10  r5 : c11d6ff0  r4 : c2c03540
	[    2.606244] r3 : e0266000  r2 : 00026000  r1 : e0240000  r0 : 01000000
	[    2.612800] Flags: nZcv  IRQs on  FIQs on  Mode SVC_32  ISA Thumb  Segment none
	[    2.620143] Control: 50c5387d  Table: 80004019  DAC: 00000051
	[    2.625912] Register r0 information: non-paged memory
	[    2.630989] Register r1 information: 0-page vmalloc region starting at 0xe0240000 allocated at devm_ioremap+0x2b/0x4c
	[    2.641664] Register r2 information: non-paged memory
	[    2.646738] Register r3 information: 0-page vmalloc region starting at 0xe0240000 allocated at devm_ioremap+0x2b/0x4c
	[    2.657402] Register r4 information: slab kmalloc-256 start c2c03500 pointer offset 64 size 256
	[    2.666163] Register r5 information: non-slab/vmalloc memory
	[    2.671849] Register r6 information: slab kmalloc-512 start c1cf1c00 pointer offset 16 size 512
	[    2.680603] Register r7 information: NULL pointer
	[    2.685328] Register r8 information: non-slab/vmalloc memory
	[    2.691014] Register r9 information: non-paged memory
	[    2.696088] Register r10 information: non-slab/vmalloc memory
	[    2.701861] Register r11 information: slab kmalloc-256 start c2c03500 pointer offset 64 size 256
	[    2.710703] Register r12 information: NULL pointer
	[    2.715516] Process kworker/u2:0 (pid: 7, stack limit = 0xfa1cef72)
	[    2.721812] Stack: (0xc196d848 to 0xc196e000)
	[    2.726191] d840:                   00000001 00000000 c11dbca8 c100b6c4 c0d67fcc c11d6ff0
	[    2.734409] d860: 00000001 c1cf1c10 dfa468ec c0d67e78 00000001 00000001 00000030 f476940d
	[    2.742625] d880: 00000000 00000000 c1cf1c10 c1116b90 c11dbca8 00000000 c1116b90 00000056
	[    2.750842] d8a0: dfa468f8 c070767b c1cf1c10 c11dbca4 00000000 c070589b 00000000 f476940d
	[    2.759059] d8c0: 00000004 c1cf1c10 c1116b90 c0706025 c1cf1c54 c1122be8 dfa46908 dfa4690c
	[    2.767275] d8e0: dfa468f8 c0705c4f c196d93c 00000001 c1cf1c10 00000000 c196d93c c0706025
	[    2.775492] d900: c1cf1c54 c1122be8 dfa46908 dfa4690c dfa468f8 c0703fa9 40000013 c1930adc
	[    2.783709] d920: c1c6cd34 f476940d c1cf1c10 00000000 00000001 c0705e29 00000180 c1cf1c10
	[    2.791925] d940: 00000001 f476940d c2c17c10 c1cf1c10 00000000 c1cf1c10 c11231a8 c0704cc3
	[    2.800142] d960: c1959d00 c1cf1c10 00000000 c2c17c10 c11dbc78 c0702f8d c0d5a3a8 dfa46908
	[    2.808359] d980: dfa4690c dfa468f8 4a326007 ff88559c 00000200 00000000 00000000 00000000
	[    2.816576] d9a0: 00000000 f476940d 00000013 c1cf1c00 dfa468ec c100b6c4 00000000 c100b6a8
	[    2.824792] d9c0: c100b6c4 00000001 00000000 c084c30b c2c17c10 dfa468ec 00000000 c100b658
	[    2.833009] d9e0: 00000000 c084c44d dfa44b54 c0849f03 c0d7202c 00000000 c2c17c10 00000001
	[    2.841225] da00: 00000000 4a326000 4a326003 ff885598 00000200 00000000 00000000 00000000
	[    2.849442] da20: 00000000 f476940d 60000013 dfa468ec 00000000 dfa44b54 00000000 c100b658
	[    2.857658] da40: c2c17c10 00000001 dfa44b60 c084c709 00000001 00000055 c2c17c10 dfa44b54
	[    2.865875] da60: c100b658 c11dbca8 00000000 c1116ab4 00000055 c061b153 00000000 c2c17c10
	[    2.874092] da80: c1116ab4 c070767b c2c17c10 c11dbca4 00000000 c070589b 00000000 f476940d
	[    2.882309] daa0: 00000004 c2c17c10 c1116ab4 c0706025 c2c17c54 c1122be8 dfa44b70 dfa44b74
	[    2.890525] dac0: dfa44b60 c0705c4f c196db1c 00000001 c2c17c10 00000000 c196db1c c0706025
	[    2.898742] dae0: c2c17c54 c1122be8 dfa44b70 dfa44b74 dfa44b60 c0703fa9 40000013 c1930adc
	[    2.906959] db00: c1c6cb34 f476940d c2c17c10 00000000 00000001 c0705e29 00000180 c2c17c10
	[    2.915175] db20: 00000001 f476940d c2c17a10 c2c17c10 00000000 c2c17c10 c11231a8 c0704cc3
	[    2.923392] db40: c1959d00 c2c17c10 00000000 c2c17a10 c11dbc78 c0702f8d c0d5a3a8 dfa44b70
	[    2.931609] db60: dfa44b74 dfa44b60 00000000 c100b658 dfa44b54 00000000 c100b658 c0849e1f
	[    2.939826] db80: dfa44bcc f476940d 00000012 c2c17c00 dfa44b54 c100b658 00000000 c100b658
	[    2.948043] dba0: c100b658 00000001 00000000 c084c30b c2c17a10 dfa44b54 00000000 c100b658
	[    2.956259] dbc0: 00000000 c084c44d c2c1af00 c2c17a10 00000000 00000000 c2c17a10 00000000
	[    2.964476] dbe0: c2c17a10 c2c17a10 00000001 c0718d19 c196dbfc c1116ab4 00000055 c0849961
	[    2.972692] dc00: 00000000 f476940d 00000000 dfa44b54 dfa44884 dfa44884 00000000 c100b658
	[    2.980909] dc20: c2c17a10 00000001 dfa44890 c084c709 00000001 00000055 c2c17a10 dfa44884
	[    2.989125] dc40: c100b658 c11dbca8 00000000 c1116ab4 00000055 c061b153 00000000 c2c17a10
	[    2.997342] dc60: c1116ab4 c070767b c2c17a10 c11dbca4 00000000 c070589b 00000000 f476940d
	[    3.005559] dc80: 00000004 c2c17a10 c1116ab4 c0706025 c2c17a54 c1122be8 dfa448a0 dfa448a4
	[    3.013776] dca0: dfa44890 c0705c4f c196dcfc 00000001 c2c17a10 00000000 c196dcfc c0706025
	[    3.021993] dcc0: c2c17a54 c1122be8 dfa448a0 dfa448a4 dfa44890 c0703fa9 40000013 c1930adc
	[    3.030209] dce0: c1c6cb34 f476940d c2c17a10 00000000 00000001 c0705e29 00000180 c2c17a10
	[    3.038426] dd00: 00000001 f476940d 00000000 c2c17a10 00000000 c2c17a10 c11231a8 c0704cc3
	[    3.046643] dd20: c1959d00 c2c17a10 00000000 c1ae3c10 c11dbc78 c0702f8d c0d5a3a8 dfa448a0
	[    3.054859] dd40: dfa448a4 dfa44890 4a0013ff ff884b76 00000200 00000000 00000000 00000000
	[    3.063076] dd60: 00000000 f476940d 00000012 c2c17a00 dfa44884 c100b658 00000000 c100b6a8
	[    3.071293] dd80: c100b658 00000001 00000000 c084c30b c1ae3c10 dfa44884 00000000 c100b658
	[    3.079509] dda0: 00000000 c084c44d 00000000 c0718a11 00000000 00000000 c1ae3c10 00000001
	[    3.087726] ddc0: 00000001 4a000000 4a0007ff ff884b70 00000200 00000000 00000000 00000000
	[    3.095942] dde0: 00000000 f476940d 60000013 dfa44884 00000000 dfa06f08 00000000 c100b658
	[    3.104159] de00: c1ae3c10 00000001 c193193c c084c709 00000001 00000001 c1ae3c10 dfa06f08
	[    3.112376] de20: c100b658 c11dbca8 00000000 c1116ab4 00000001 c061b153 00000000 c1ae3c10
	[    3.120593] de40: c1116ab4 c070767b c1ae3c10 c11dbca4 00000000 c070589b 00000000 f476940d
	[    3.128810] de60: 00000004 c1ae3c10 c1116ab4 c0706025 c1ae3c54 c0d8b434 c1179e68 c1122f5c
	[    3.137026] de80: c193193c c0705c4f c196dedc 00000001 c1ae3c10 00000000 c196dedc c0706025
	[    3.145243] dea0: c1ae3c54 c0d8b434 c1179e68 c1122f5c c193193c c0703fa9 40000013 c1930adc
	[    3.153460] dec0: c1c6cb34 f476940d c1ae3c10 c1ae3c10 00000001 c0705e29 c1122f70 c1ae3c10
	[    3.161677] dee0: 00000001 f476940d c1959d00 c1ad8950 c1ae3c10 c1ae3c10 c11231a8 c0704cc3
	[    3.169894] df00: 00000025 c1ad8950 c1ae3c10 c1122f2c c1122f70 c07054d1 c0705461 c1122f58
	[    3.178111] df20: c1931900 00000040 c182b500 00000000 00000000 c0134d6f c196c000 c1805800
	[    3.186327] df40: c1805814 c1931900 c1805800 c1931914 c1080aa0 c1805814 c196c000 c1805800
	[    3.194544] df60: c1805840 c0135173 00000000 c192ef80 c192e040 c0134ff5 c1931900 c196c000
	[    3.202761] df80: c1953eb8 c192e060 00000000 c013a095 c192ef80 c0139f81 00000000 00000000
	[    3.210978] dfa0: 00000000 00000000 00000000 c0100159 00000000 00000000 00000000 00000000
	[    3.219193] dfc0: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
	[    3.227409] dfe0: 00000000 00000000 00000000 00000000 00000013 00000000 00000000 00000000
	[    3.235633] [<c061d116>] (sysc_probe) from [<c070767b>] (platform_probe+0x43/0x84)
	[    3.243255] [<c070767b>] (platform_probe) from [<c070589b>] (really_probe+0xb3/0x3b4)
	[    3.251130] [<c070589b>] (really_probe) from [<c0705c4f>] (driver_probe_device+0xb3/0x160)
	[    3.259441] [<c0705c4f>] (driver_probe_device) from [<c0703fa9>] (bus_for_each_drv+0x4d/0x80)
	[    3.268012] [<c0703fa9>] (bus_for_each_drv) from [<c0705e29>] (__device_attach+0x89/0x12c)
	[    3.276321] [<c0705e29>] (__device_attach) from [<c0704cc3>] (bus_probe_device+0x5b/0x60)
	[    3.284543] [<c0704cc3>] (bus_probe_device) from [<c0702f8d>] (device_add+0x295/0x688)
	[    3.292502] [<c0702f8d>] (device_add) from [<c084c30b>] (of_platform_device_create_pdata+0x6b/0x94)
	[    3.301612] [<c084c30b>] (of_platform_device_create_pdata) from [<c084c44d>] (of_platform_bus_create+0x10d/0x244)
	[    3.311930] [<c084c44d>] (of_platform_bus_create) from [<c084c709>] (of_platform_populate+0x59/0xd8)
	[    3.321113] [<c084c709>] (of_platform_populate) from [<c061b153>] (simple_pm_bus_probe+0x2b/0x50)
	[    3.330034] [<c061b153>] (simple_pm_bus_probe) from [<c070767b>] (platform_probe+0x43/0x84)
	[    3.338431] [<c070767b>] (platform_probe) from [<c070589b>] (really_probe+0xb3/0x3b4)
	[    3.346305] [<c070589b>] (really_probe) from [<c0705c4f>] (driver_probe_device+0xb3/0x160)
	[    3.354614] [<c0705c4f>] (driver_probe_device) from [<c0703fa9>] (bus_for_each_drv+0x4d/0x80)
	[    3.363183] [<c0703fa9>] (bus_for_each_drv) from [<c0705e29>] (__device_attach+0x89/0x12c)
	[    3.371492] [<c0705e29>] (__device_attach) from [<c0704cc3>] (bus_probe_device+0x5b/0x60)
	[    3.379713] [<c0704cc3>] (bus_probe_device) from [<c0702f8d>] (device_add+0x295/0x688)
	[    3.387672] [<c0702f8d>] (device_add) from [<c084c30b>] (of_platform_device_create_pdata+0x6b/0x94)
	[    3.396767] [<c084c30b>] (of_platform_device_create_pdata) from [<c084c44d>] (of_platform_bus_create+0x10d/0x244)
	[    3.407085] [<c084c44d>] (of_platform_bus_create) from [<c084c709>] (of_platform_populate+0x59/0xd8)
	[    3.416269] [<c084c709>] (of_platform_populate) from [<c061b153>] (simple_pm_bus_probe+0x2b/0x50)
	[    3.425190] [<c061b153>] (simple_pm_bus_probe) from [<c070767b>] (platform_probe+0x43/0x84)
	[    3.433586] [<c070767b>] (platform_probe) from [<c070589b>] (really_probe+0xb3/0x3b4)
	[    3.441459] [<c070589b>] (really_probe) from [<c0705c4f>] (driver_probe_device+0xb3/0x160)
	[    3.449768] [<c0705c4f>] (driver_probe_device) from [<c0703fa9>] (bus_for_each_drv+0x4d/0x80)
	[    3.458337] [<c0703fa9>] (bus_for_each_drv) from [<c0705e29>] (__device_attach+0x89/0x12c)
	[    3.466646] [<c0705e29>] (__device_attach) from [<c0704cc3>] (bus_probe_device+0x5b/0x60)
	[    3.474867] [<c0704cc3>] (bus_probe_device) from [<c0702f8d>] (device_add+0x295/0x688)
	[    3.482826] [<c0702f8d>] (device_add) from [<c084c30b>] (of_platform_device_create_pdata+0x6b/0x94)
	[    3.491920] [<c084c30b>] (of_platform_device_create_pdata) from [<c084c44d>] (of_platform_bus_create+0x10d/0x244)
	[    3.502238] [<c084c44d>] (of_platform_bus_create) from [<c084c709>] (of_platform_populate+0x59/0xd8)
	[    3.511423] [<c084c709>] (of_platform_populate) from [<c061b153>] (simple_pm_bus_probe+0x2b/0x50)
	[    3.520343] [<c061b153>] (simple_pm_bus_probe) from [<c070767b>] (platform_probe+0x43/0x84)
	[    3.528739] [<c070767b>] (platform_probe) from [<c070589b>] (really_probe+0xb3/0x3b4)
	[    3.536612] [<c070589b>] (really_probe) from [<c0705c4f>] (driver_probe_device+0xb3/0x160)
	[    3.544921] [<c0705c4f>] (driver_probe_device) from [<c0703fa9>] (bus_for_each_drv+0x4d/0x80)
	[    3.553491] [<c0703fa9>] (bus_for_each_drv) from [<c0705e29>] (__device_attach+0x89/0x12c)
	[    3.561799] [<c0705e29>] (__device_attach) from [<c0704cc3>] (bus_probe_device+0x5b/0x60)
	[    3.570020] [<c0704cc3>] (bus_probe_device) from [<c07054d1>] (deferred_probe_work_func+0x71/0xa4)
	[    3.579027] [<c07054d1>] (deferred_probe_work_func) from [<c0134d6f>] (process_one_work+0x137/0x3bc)
	[    3.588220] [<c0134d6f>] (process_one_work) from [<c0135173>] (worker_thread+0x17f/0x410)
	[    3.596444] [<c0135173>] (worker_thread) from [<c013a095>] (kthread+0x115/0x13c)
	[    3.603884] [<c013a095>] (kthread) from [<c0100159>] (ret_from_fork+0x11/0x38)
	[    3.611146] Exception stack(0xc196dfb0 to 0xc196dff8)
	[    3.616221] dfa0:                                     00000000 00000000 00000000 00000000
	[    3.624437] dfc0: 00000000 00000000 00000000 00000000 00000000 00000000 00000000 00000000
	[    3.632652] dfe0: 00000000 00000000 00000000 00000000 00000013 00000000
	[    3.639302] Code: 0747 eb01 0302 d454 (681b) 65e3
	[    3.644116] ---[ end trace da37fde2ed057c90 ]---

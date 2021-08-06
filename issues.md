## Issues (kernel v5.13.x)

### [1] i2c structure (kernel v5.13.x) in include/linux/i2c.h changed

i2c structure in include/linux/i2c.h had some struct members' changes!

	struct i2c_board_info {
		char            type[I2C_NAME_SIZE];
		unsigned short  flags;
		unsigned short  addr;
		const char	*dev_name;
		void            *platform_data;
		struct device_node *of_node;
		struct fwnode_handle *fwnode;
	+	const struct software_node *swnode;	 // <<== new member of the structure
	-	const struct property_entry *properties; // <<== obsolete member of the structure
		const struct resource *resources;
		unsigned int    num_resources;
		int             irq;
	};

Thus, to solve this issue the line 753 adds function helper:

https://elixir.bootlin.com/linux/v5.13.6/source/drivers/base/swnode.c#L723

	diff --git a/mikrobus_core.c b/mikrobus_core.c
	index 53d2e16..8729d68 100644
	--- a/mikrobus_core.c
	+++ b/mikrobus_core.c
	@@ -590,7 +590,7 @@ static int mikrobus_device_register(struct mikrobus_port *port,
	                if (dev->irq)
	                        i2c->irq = mikrobus_irq_get(port, dev->irq, dev->irq_type);
	                if (dev->properties)
	-                       i2c->properties = dev->properties;
	+                       i2c->swnode = software_node_alloc(dev->properties);
	                i2c->addr = dev->reg;
	                dev->dev_client = (void *) i2c_new_client_device(port->i2c_adap, i2c);
	                break;

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

### [2] GPIO Issues (kernel 5.13.5-bone13, listing part of a dmesg log)

```
Starting kernel ...

[    0.000000] Booting Linux on physical CPU 0x0
[    0.000000] Linux version 5.13.5-bone13 (vuser@fedora33-ssd) (arm-linux-gnueabi-gcc (GCC) 11.1.0, GNU ld (GNU Binutils) 2.36.1) #1
 PREEMPT Sat Jul 31 07:10:12 CEST 2021
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
[    0.000000] Kernel command line: console=ttyO0,115200n8 bone_capemgr.uboot_capemgr_enabled=1 root=/dev/mmcblk0p1 ro rootfstype=ext
4 rootwait coherent_pool=1M net.ifnames=0 rng_core.default_quality=100
[    0.000000] Dentry cache hash table entries: 65536 (order: 6, 262144 bytes, linear)
[    0.000000] Inode-cache hash table entries: 32768 (order: 5, 131072 bytes, linear)
[    0.000000] mem auto-init: stack:off, heap alloc:off, heap free:off
[    0.000000] Memory: 450540K/523264K available (10240K kernel code, 1620K rwdata, 3504K rodata, 1024K init, 324K bss, 23572K reserv
ed, 49152K cma-reserved, 0K highmem)
[    0.000000] random: get_random_u32 called from __kmem_cache_create+0x1b/0x290 with crng_init=0
[    0.000000] SLUB: HWalign=64, Order=0-3, MinObjects=0, CPUs=1, Nodes=1
[    0.000000] ftrace: allocating 43321 entries in 85 pages
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
[    0.000023] clocksource: dmtimer: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 79635851949 ns
[    0.000371] TI gptimer clockevent: 24000000 Hz at /ocp/interconnect@48000000/segment@0/target-module@40000
[    0.001970] Console: colour dummy device 80x30
[    0.002019] WARNING: Your 'console=ttyO0' has been replaced by 'ttyS0'
[    0.002031] This ensures that you still see kernel messages. Please
[    0.002039] update your kernel commandline.
[    0.002084] Calibrating delay loop... 995.32 BogoMIPS (lpj=1990656)
[    0.020450] pid_max: default: 32768 minimum: 301
[    0.020654] LSM: Security Framework initializing
[    0.020783] Yama: becoming mindful.
[    0.020952] AppArmor: AppArmor initialized
[    0.020972] TOMOYO Linux initialized
[    0.021059] Mount-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.021079] Mountpoint-cache hash table entries: 1024 (order: 0, 4096 bytes, linear)
[    0.022127] CPU: Testing write buffer coherency: ok
[    0.022201] CPU0: Spectre v2: using BPIALL workaround
[    0.023343] Setting up static identity map for 0x80100000 - 0x80100054
[    0.023527] rcu: Hierarchical SRCU implementation.
[    0.024584] EFI services will not be available.
[    0.025195] devtmpfs: initialized
[    0.049609] VFP support v0.3: implementor 41 architecture 3 part 30 variant c rev 3
[    0.049964] clocksource: jiffies: mask: 0xffffffff max_cycles: 0xffffffff, max_idle_ns: 7645041785100000 ns
[    0.049994] futex hash table entries: 256 (order: -1, 3072 bytes, linear)
[    0.055125] pinctrl core: initialized pinctrl subsystem
[    0.056520] NET: Registered protocol family 16
[    0.060212] DMA: preallocated 1024 KiB pool for atomic coherent allocations
[    0.060773] audit: initializing netlink subsys (disabled)
[    0.061701] thermal_sys: Registered thermal governor 'fair_share'
[    0.061716] thermal_sys: Registered thermal governor 'bang_bang'
[    0.061729] thermal_sys: Registered thermal governor 'step_wise'
[    0.062550] cpuidle: using governor menu
[    0.068511] audit: type=2000 audit(0.060:1): state=initialized audit_enabled=0 res=1
[    0.087756] hw-breakpoint: debug architecture 0x4 unsupported.
[    0.106508] raid6: skip pq benchmark and using algorithm neonx8
[    0.106545] raid6: using neon recovery algorithm
[    0.107913] iommu: Default domain type: Translated 
[    0.110248] SCSI subsystem initialized
[    0.110545] usbcore: registered new interface driver usbfs
[    0.110597] usbcore: registered new interface driver hub
[    0.110662] usbcore: registered new device driver usb
[    0.111050] pps_core: LinuxPPS API ver. 1 registered
[    0.111066] pps_core: Software ver. 5.3.6 - Copyright 2005-2007 Rodolfo Giometti <giometti@linux.it>
[    0.111098] PTP clock support registered
[    0.111808] Advanced Linux Sound Architecture Driver Initialized.
[    0.112954] NetLabel: Initializing
[    0.112976] NetLabel:  domain hash size = 128
[    0.112985] NetLabel:  protocols = UNLABELED CIPSOv4 CALIPSO
[    0.113081] NetLabel:  unlabeled traffic allowed by default
[    0.114025] clocksource: Switched to clocksource dmtimer
[    0.194660] VFS: Disk quotas dquot_6.6.0
[    0.194758] VFS: Dquot-cache hash table entries: 1024 (order 0, 4096 bytes)
[    0.195526] AppArmor: AppArmor Filesystem Enabled
[    0.214197] NET: Registered protocol family 2
[    0.214417] IP idents hash table entries: 8192 (order: 4, 65536 bytes, linear)
[    0.218136] tcp_listen_portaddr_hash hash table entries: 512 (order: 0, 4096 bytes, linear)
[    0.218216] TCP established hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.218261] TCP bind hash table entries: 4096 (order: 2, 16384 bytes, linear)
[    0.218304] TCP: Hash tables configured (established 4096 bind 4096)
[    0.218432] UDP hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.218459] UDP-Lite hash table entries: 256 (order: 0, 4096 bytes, linear)
[    0.218657] NET: Registered protocol family 1
[    0.222428] RPC: Registered named UNIX socket transport module.
[    0.222452] RPC: Registered udp transport module.
[    0.222461] RPC: Registered tcp transport module.
[    0.222469] RPC: Registered tcp NFSv4.1 backchannel transport module.
[    0.222485] NET: Registered protocol family 44
[    1.018026] random: fast init done
[    1.196741] Initialise system trusted keyrings
[    1.197099] workingset: timestamp_bits=14 max_order=17 bucket_order=3
[    1.202786] zbud: loaded
[    1.205335] NFS: Registering the id_resolver key type
[    1.205392] Key type id_resolver registered
[    1.205404] Key type id_legacy registered
[    1.205519] nfs4filelayout_init: NFSv4 File Layout Driver Registering...
[    1.205534] nfs4flexfilelayout_init: NFSv4 Flexfile Layout Driver Registering...
[    1.205828] fuse: init (API version 7.34)
[    1.297537] xor: automatically using best checksumming function   neon      
[    1.297572] Key type asymmetric registered
[    1.297583] Asymmetric key parser 'x509' registered
[    1.297650] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 246)
[    1.297861] io scheduler mq-deadline registered
[    1.301177] Serial: 8250/16550 driver, 6 ports, IRQ sharing disabled
[    1.304410] sdhci: Secure Digital Host Controller Interface driver
[    1.304436] sdhci: Copyright(c) Pierre Ossman
[    1.304558] sdhci-pltfm: SDHCI platform and OF driver helper
[    1.307489] libphy: Fixed MDIO Bus: probed
[    1.308213] CAN device driver interface
[    1.308685] usbcore: registered new interface driver smsc95xx
[    1.309138] usbcore: registered new interface driver uas
[    1.309201] usbcore: registered new interface driver usb-storage
[    1.309235] usbcore: registered new interface driver ums-alauda
[    1.309269] usbcore: registered new interface driver ums-cypress
[    1.309301] usbcore: registered new interface driver ums-datafab
[    1.309334] usbcore: registered new interface driver ums_eneub6250
[    1.309380] usbcore: registered new interface driver ums-freecom
[    1.309415] usbcore: registered new interface driver ums-isd200
[    1.309448] usbcore: registered new interface driver ums-jumpshot
[    1.309481] usbcore: registered new interface driver ums-karma
[    1.309515] usbcore: registered new interface driver ums-onetouch
[    1.309557] usbcore: registered new interface driver ums-realtek
[    1.309591] usbcore: registered new interface driver ums-sddr09
[    1.309630] usbcore: registered new interface driver ums-sddr55
[    1.309663] usbcore: registered new interface driver ums-usbat
[    1.311181] i2c /dev entries driver
[    1.312139] softdog: initialized. soft_noboot=0 soft_margin=60 sec soft_panic=0 (nowayout=0)
[    1.312159] softdog:              soft_reboot_cmd=<not set> soft_active_on_boot=0
[    1.313152] cpuidle: enable-method property 'ti,am3352' found operations
[    1.313630] ledtrig-cpu: registered to indicate activity on CPUs
[    1.314256] hid: raw HID events driver (C) Jiri Kosina
[    1.314531] usbcore: registered new interface driver usbhid
[    1.314547] usbhid: USB HID core driver
[    1.315722] drop_monitor: Initializing network drop monitor service
[    1.316566] NET: Registered protocol family 10
[    1.320364] Segment Routing with IPv6
[    1.320502] mip6: Mobile IPv6
[    1.320518] NET: Registered protocol family 17
[    1.320553] can: controller area network core
[    1.320646] NET: Registered protocol family 29
[    1.320955] Key type dns_resolver registered
[    1.320974] mpls_gso: MPLS GSO support
[    1.321246] ThumbEE CPU extension supported.
[    1.321266] Registering SWP/SWPB emulation handler
[    1.321277] omap_voltage_late_init: Voltage driver support not added
[    1.321523] PM: Cannot get wkup_m3_ipc handle
[    1.322276] registered taskstats version 1
[    1.322308] Loading compiled-in X.509 certificates
[    1.322499] zswap: loaded using pool lzo/zbud
[    1.323111] Key type ._fscrypt registered
[    1.323130] Key type .fscrypt registered
[    1.323139] Key type fscrypt-provisioning registered
[    1.326723] Btrfs loaded, crc32c=crc32c-generic, zoned=yes
[    1.326849] AppArmor: AppArmor sha1 policy hashing enabled
[    1.344833] remoteproc remoteproc0: wkup_m3 is available
[    1.352292] 44e09000.serial: ttyS0 at MMIO 0x44e09000 (irq = 18, base_baud = 3000000) is a 8250
[    2.333788] printk: console [ttyS0] enabled
[    2.348999] pinctrl-single 44e10800.pinmux: 142 pins, size 568
[    2.361837] omap_wdt: OMAP Watchdog Timer Rev 0x01: initial timeout 60 sec
[    2.370943] omap_rtc 44e3e000.rtc: already running
[    2.376692] omap_rtc 44e3e000.rtc: registered as rtc0
[    2.381838] omap_rtc 44e3e000.rtc: setting system clock to 2000-01-01T00:03:24 UTC (946685004)
[    2.395274] 48022000.serial: ttyS1 at MMIO 0x48022000 (irq = 25, base_baud = 3000000) is a 8250
[    2.406715] 48024000.serial: ttyS2 at MMIO 0x48024000 (irq = 26, base_baud = 3000000) is a 8250
[    2.417935] omap_i2c 4802a000.i2c: bus 1 rev0.11 at 400 kHz
[    2.436293] OMAP GPIO hardware version 0.1
[    2.445614] omap-mailbox 480c8000.mailbox: omap mailbox rev 0x400
[    2.455689] omap_i2c 4819c000.i2c: bus 2 rev0.11 at 400 kHz
[    2.465520] 481a8000.serial: ttyS4 at MMIO 0x481a8000 (irq = 39, base_baud = 3000000) is a 8250
[    2.474535] serial serial0: tty port ttyS4 registered
[    2.486612] c_can_platform 481cc000.can: c_can_platform device registered (regs=(ptrval), irq=42)
[    2.497866] c_can_platform 481d0000.can: c_can_platform device registered (regs=(ptrval), irq=43)
[    2.518813] omap_rng 48310000.rng: Random Number Generator ver. 20
[    2.525256] random: crng init done
[    2.540087] debugfs: Directory '49000000.dma' with parent 'dmaengine' already present!
[    2.548130] edma 49000000.dma: TI EDMA DMA engine driver
[    2.559267] am335x-phy-driver 47401300.usb-phy: supply vcc not found, using dummy regulator
[    2.579499] am335x-phy-driver 47401b00.usb-phy: supply vcc not found, using dummy regulator
[    2.608345] omap-sham 53100000.sham: hw accel on OMAP rev 4.3
[    2.614445] omap-sham 53100000.sham: will run requests pump with realtime priority
[    2.624944] omap-aes 53500000.aes: OMAP AES hw accel rev: 3.2
[    2.631245] omap-aes 53500000.aes: will run requests pump with realtime priority
[    2.642962] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.649715] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_20_default_pin
[    2.657969] bone-pinmux-helper: probe of ocp:P1_20_pinmux failed with error -22
[    2.666835] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.673417] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_26_default_pin
[    2.681632] bone-pinmux-helper: probe of ocp:P1_26_pinmux failed with error -22
[    2.690311] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.696831] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_28_default_pin
[    2.704991] bone-pinmux-helper: probe of ocp:P1_28_pinmux failed with error -22
[    2.713503] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.720047] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_29_default_pin
[    2.728319] bone-pinmux-helper: probe of ocp:P1_29_pinmux failed with error -22
[    2.736859] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.743406] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_30_default_pin
[    2.751569] bone-pinmux-helper: probe of ocp:P1_30_pinmux failed with error -22
[    2.760042] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.766580] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_31_default_pin
[    2.774742] bone-pinmux-helper: probe of ocp:P1_31_pinmux failed with error -22
[    2.783186] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.789716] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_32_default_pin
[    2.797876] bone-pinmux-helper: probe of ocp:P1_32_pinmux failed with error -22
[    2.806382] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.812898] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_33_default_pin
[    2.821058] bone-pinmux-helper: probe of ocp:P1_33_pinmux failed with error -22
[    2.829478] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.836016] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_34_default_pin
[    2.844174] bone-pinmux-helper: probe of ocp:P1_34_pinmux failed with error -22
[    2.852672] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.859218] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_35_default_pin
[    2.867376] bone-pinmux-helper: probe of ocp:P1_35_pinmux failed with error -22
[    2.875845] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.882389] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P1_36_default_pin
[    2.890549] bone-pinmux-helper: probe of ocp:P1_36_pinmux failed with error -22
[    2.899024] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.905559] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_02_default_pin
[    2.913724] bone-pinmux-helper: probe of ocp:P2_02_pinmux failed with error -22
[    2.922423] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.928945] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_04_default_pin
[    2.937106] bone-pinmux-helper: probe of ocp:P2_04_pinmux failed with error -22
[    2.945594] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.952136] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_06_default_pin
[    2.960299] bone-pinmux-helper: probe of ocp:P2_06_pinmux failed with error -22
[    2.968769] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.975307] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_08_default_pin
[    2.983467] bone-pinmux-helper: probe of ocp:P2_08_pinmux failed with error -22
[    2.991946] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    2.998484] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_10_default_pin
[    3.006644] bone-pinmux-helper: probe of ocp:P2_10_pinmux failed with error -22
[    3.015123] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.021656] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_17_default_pin
[    3.029816] bone-pinmux-helper: probe of ocp:P2_17_pinmux failed with error -22
[    3.038337] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.044853] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_18_default_pin
[    3.053011] bone-pinmux-helper: probe of ocp:P2_18_pinmux failed with error -22
[    3.061480] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.068018] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_19_default_pin
[    3.076184] bone-pinmux-helper: probe of ocp:P2_19_pinmux failed with error -22
[    3.084657] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.091192] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_20_default_pin
[    3.099362] bone-pinmux-helper: probe of ocp:P2_20_pinmux failed with error -22
[    3.107762] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.114291] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_22_default_pin
[    3.122451] bone-pinmux-helper: probe of ocp:P2_22_pinmux failed with error -22
[    3.130873] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.137400] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_24_default_pin
[    3.145558] bone-pinmux-helper: probe of ocp:P2_24_pinmux failed with error -22
[    3.153915] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.160445] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_25_default_pin
[    3.168601] bone-pinmux-helper: probe of ocp:P2_25_pinmux failed with error -22
[    3.177113] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.183647] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_27_default_pin
[    3.191808] bone-pinmux-helper: probe of ocp:P2_27_pinmux failed with error -22
[    3.200193] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.206720] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_28_default_pin
[    3.214879] bone-pinmux-helper: probe of ocp:P2_28_pinmux failed with error -22
[    3.223345] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.229881] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_29_default_pin
[    3.238043] bone-pinmux-helper: probe of ocp:P2_29_pinmux failed with error -22
[    3.246508] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.253025] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_30_default_pin
[    3.261185] bone-pinmux-helper: probe of ocp:P2_30_pinmux failed with error -22
[    3.269678] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.276217] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_31_default_pin
[    3.284380] bone-pinmux-helper: probe of ocp:P2_31_pinmux failed with error -22
[    3.292864] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.299400] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_32_default_pin
[    3.307560] bone-pinmux-helper: probe of ocp:P2_32_pinmux failed with error -22
[    3.315950] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.322481] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_33_default_pin
[    3.330642] bone-pinmux-helper: probe of ocp:P2_33_pinmux failed with error -22
[    3.339025] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.345560] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_34_default_pin
[    3.353718] bone-pinmux-helper: probe of ocp:P2_34_pinmux failed with error -22
[    3.362235] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[    3.368745] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_35_default_pin
[    3.376906] bone-pinmux-helper: probe of ocp:P2_35_pinmux failed with error -22
[    3.385412] gpio-of-helper ocp:cape-universal: Allocated GPIO id=0 name='P1_02'
[    3.392980] gpio-of-helper ocp:cape-universal: Allocated GPIO id=1 name='P1_04'
[    3.400368] gpio-of-helper ocp:cape-universal: Failed to get gpio property of 'P1_06'
[    3.408247] gpio-of-helper ocp:cape-universal: Failed to create gpio entry
[    3.417878] hw perfevents: enabled with armv7_cortex_a8 PMU driver, 5 counters available
[    3.431274] l3-aon-clkctrl:0000:0: failed to disable
[    3.437925] PM: Cannot get wkup_m3_ipc handle
[    4.450193] tps65217 0-0024: Read from reg 0x2 failed
[    4.455607] tps65217-pmic: Failed to locate of_node [id: -1]
[    5.466188] tps65217 0-0024: Read from reg 0x16 failed
[    5.471376] vdds_dpr: failed to enable: -EBUSY
[    5.475862] tps65217 0-0024: failed to register tps65217-pmic regulator
[    5.482535] tps65217-pmic: probe of tps65217-pmic failed with error -16
[    5.489341] tps65217-bl: Failed to locate of_node [id: -1]
[    6.502186] tps65217 0-0024: Read from reg 0x2 failed
[    6.507286] tps65217 0-0024: Failed to sync IRQ masks
[    7.518187] tps65217 0-0024: Failed to read revision register: -16
[    7.525123] tps65217: probe of 0-0024 failed with error -16
[    7.531392] at24 0-0050: supply vcc not found, using dummy regulator
[    9.556203] omap_i2c 44e0b000.i2c: bus 0 rev0.11 at 400 kHz
[    9.565916] channel@1 enforce active low on chipselect handle
[    9.571860] remoteproc remoteproc0: powering up wkup_m3
[    9.577590] remoteproc remoteproc0: Booting fw image am335x-pm-firmware.elf, size 217148
[    9.587144] omap_gpio 44e07000.gpio: Could not set line 6 debounce to 200000 microseconds (-22)
[    9.601291] musb-hdrc musb-hdrc.0: MUSB HDRC host driver
[    9.606754] sdhci-omap 48060000.mmc: Got CD GPIO
[    9.611642] remoteproc remoteproc0: remote processor wkup_m3 is now up
[    9.618211] wkup_m3_ipc 44e11324.wkup_m3_ipc: CM3 Firmware Version = 0x192
[    9.625196] musb-hdrc musb-hdrc.0: new USB bus registered, assigned bus number 1
[    9.632843] sdhci-omap 48060000.mmc: supply vqmmc not found, using dummy regulator
[    9.640723] usb usb1: New USB device found, idVendor=1d6b, idProduct=0002, bcdDevice= 5.13
[    9.651275] usb usb1: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    9.658661] usb usb1: Product: MUSB HDRC host driver
[    9.663825] usb usb1: Manufacturer: Linux 5.13.5-bone13 musb-hcd
[    9.669939] usb usb1: SerialNumber: musb-hdrc.0
[    9.675223] hub 1-0:1.0: USB hub found
[    9.679115] hub 1-0:1.0: 1 port detected
[    9.690140] mmc0: SDHCI controller on 48060000.mmc [48060000.mmc] using External DMA
[    9.702281] musb-hdrc musb-hdrc.1: MUSB HDRC host driver
[    9.707947] musb-hdrc musb-hdrc.1: new USB bus registered, assigned bus number 2
[    9.715699] usb usb2: New USB device found, idVendor=1d6b, idProduct=0002, bcdDevice= 5.13
[    9.724056] usb usb2: New USB device strings: Mfr=3, Product=2, SerialNumber=1
[    9.731354] usb usb2: Product: MUSB HDRC host driver
[    9.736384] usb usb2: Manufacturer: Linux 5.13.5-bone13 musb-hcd
[    9.742520] usb usb2: SerialNumber: musb-hdrc.1
[    9.748070] hub 2-0:1.0: USB hub found
[    9.751980] hub 2-0:1.0: 1 port detected
[    9.762673] gpio-of-helper ocp:cape-universal: Allocated GPIO id=0 name='P1_02'
[    9.770388] gpio-of-helper ocp:cape-universal: Allocated GPIO id=1 name='P1_04'
[    9.778009] gpio-of-helper ocp:cape-universal: Allocated GPIO id=2 name='P1_06'
[    9.785624] gpio-of-helper ocp:cape-universal: Allocated GPIO id=3 name='P1_08'
[    9.793173] gpio-of-helper ocp:cape-universal: Allocated GPIO id=4 name='P1_10'
[    9.800724] gpio-of-helper ocp:cape-universal: Allocated GPIO id=5 name='P1_12'
[    9.808243] gpio-of-helper ocp:cape-universal: Allocated GPIO id=6 name='P1_20'
[    9.815753] gpio-of-helper ocp:cape-universal: Allocated GPIO id=7 name='P1_26'
[    9.823279] gpio-of-helper ocp:cape-universal: Allocated GPIO id=8 name='P1_28'
[    9.831296] gpio-of-helper ocp:cape-universal: Allocated GPIO id=9 name='P1_29'
[    9.839099] gpio-of-helper ocp:cape-universal: Allocated GPIO id=10 name='P1_30'
[    9.846759] gpio-of-helper ocp:cape-universal: Allocated GPIO id=11 name='P1_31'
[    9.854403] gpio-of-helper ocp:cape-universal: Allocated GPIO id=12 name='P1_32'
[    9.862086] gpio-of-helper ocp:cape-universal: Allocated GPIO id=13 name='P1_33'
[    9.869800] gpio-of-helper ocp:cape-universal: Allocated GPIO id=14 name='P1_34'
[    9.877460] gpio-of-helper ocp:cape-universal: Allocated GPIO id=15 name='P1_35'
[    9.885138] gpio-of-helper ocp:cape-universal: Allocated GPIO id=16 name='P1_36'
[    9.892805] gpio-of-helper ocp:cape-universal: Allocated GPIO id=17 name='P2_01'
[    9.900392] mmc0: new high speed SDHC card at address 1234
[    9.906103] gpio-of-helper ocp:cape-universal: Allocated GPIO id=18 name='P2_02'
[    9.914462] mmcblk0: mmc0:1234 SA16G 14.5 GiB 
[    9.919201] gpio-of-helper ocp:cape-universal: Allocated GPIO id=19 name='P2_03'
[    9.928039] gpio-of-helper ocp:cape-universal: Allocated GPIO id=20 name='P2_04'
[    9.935704]  mmcblk0: p1
[    9.939401] gpio-of-helper ocp:cape-universal: Allocated GPIO id=21 name='P2_05'
[    9.947138] gpio-of-helper ocp:cape-universal: Allocated GPIO id=22 name='P2_06'
[    9.954772] gpio-of-helper ocp:cape-universal: Allocated GPIO id=23 name='P2_07'
[    9.962658] gpio-of-helper ocp:cape-universal: Allocated GPIO id=24 name='P2_08'
[    9.970268] gpio-of-helper ocp:cape-universal: Allocated GPIO id=25 name='P2_09'
[    9.977851] gpio-of-helper ocp:cape-universal: Allocated GPIO id=26 name='P2_10'
[    9.985425] gpio-of-helper ocp:cape-universal: Allocated GPIO id=27 name='P2_11'
[    9.993020] gpio-of-helper ocp:cape-universal: Allocated GPIO id=28 name='P2_17'
[   10.000602] gpio-of-helper ocp:cape-universal: Allocated GPIO id=29 name='P2_18'
[   10.008180] gpio-of-helper ocp:cape-universal: Allocated GPIO id=30 name='P2_19'
[   10.015762] gpio-of-helper ocp:cape-universal: Allocated GPIO id=31 name='P2_20'
[   10.023344] gpio-of-helper ocp:cape-universal: Allocated GPIO id=32 name='P2_22'
[   10.030942] gpio-of-helper ocp:cape-universal: Allocated GPIO id=33 name='P2_24'
[   10.038522] gpio-of-helper ocp:cape-universal: Allocated GPIO id=34 name='P2_25'
[   10.046109] gpio-of-helper ocp:cape-universal: Allocated GPIO id=35 name='P2_27'
[   10.053774] gpio-of-helper ocp:cape-universal: Allocated GPIO id=36 name='P2_28'
[   10.061367] gpio-of-helper ocp:cape-universal: Allocated GPIO id=37 name='P2_29'
[   10.068952] gpio-of-helper ocp:cape-universal: Allocated GPIO id=38 name='P2_30'
[   10.076535] gpio-of-helper ocp:cape-universal: Allocated GPIO id=39 name='P2_31'
[   10.084107] gpio-of-helper ocp:cape-universal: Allocated GPIO id=40 name='P2_32'
[   10.091696] gpio-of-helper ocp:cape-universal: Allocated GPIO id=41 name='P2_33'
[   10.099282] gpio-of-helper ocp:cape-universal: Allocated GPIO id=42 name='P2_34'
[   10.106869] gpio-of-helper ocp:cape-universal: Allocated GPIO id=43 name='P2_35'
[   10.114320] gpio-of-helper ocp:cape-universal: ready
[   10.121672] PM: bootloader does not support rtc-only!
[   10.130239] ALSA device list:
[   10.133251]   No soundcards found.
[   10.151743] EXT4-fs (mmcblk0p1): mounted filesystem with ordered data mode. Opts: (null). Quota mode: none.
[   10.161693] VFS: Mounted root (ext4 filesystem) readonly on device 179:1.
[   10.178056] devtmpfs: mounted
[   10.185657] Freeing unused kernel memory: 1024K
[   10.190768] Run /sbin/init as init process
[   10.238397] Not activating Mandatory Access Control as /sbin/tomoyo-init does not exist.
[   10.811637] systemd[1]: System time before build time, advancing clock.
[   11.006582] systemd[1]: systemd 241 running in system mode. (+PAM +AUDIT +SELINUX +IMA +APPARMOR +SMACK +SYSVINIT +UTMP +LIBCRYPTS
ETUP +GCRYPT +GNUTLS +ACL +XZ +LZ4 +SECCOMP +BLKID +ELFUTILS +KMOD -IDN2 +IDN -PCRE2 default-hierarchy=hybrid)
[   11.028780] systemd[1]: Detected architecture arm.

Welcome to Debian GNU/Linux 10 (buster)!

[   11.060216] systemd[1]: Set hostname to <arm>.
[   12.608239] systemd[1]: Listening on Device-mapper event daemon FIFOs.
[  OK  ] Listening on Device-mapper event daemon FIFOs.
[   12.631824] systemd[1]: Listening on Journal Audit Socket.
[  OK  ] Listening on Journal Audit Socket.
[   12.655186] systemd[1]: Started Dispatch Password Requests to Console Directory Watch.
[  OK  ] Started Dispatch Password …ts to Console Directory Watch.
[   12.679527] systemd[1]: Listening on Journal Socket (/dev/log).
[  OK  ] Listening on Journal Socket (/dev/log).
[   12.706223] systemd[1]: Created slice User and Session Slice.
[  OK  ] Created slice User and Session Slice.
[   12.731142] systemd[1]: Started Forward Password Requests to Wall Directory Watch.
[  OK  ] Started Forward Password R…uests to Wall Directory Watch.
[   12.754660] systemd[1]: Reached target Local Encrypted Volumes.
[  OK  ] Reached target Local Encrypted Volumes.
[  OK  ] Listening on udev Control Socket.
[  OK  ] Listening on udev Kernel Socket.
[  OK  ] Listening on fsck to fsckd communication Socket.
[  OK  ] Listening on Journal Socket.
         Starting Set the console keyboard layout...
         Starting Load Kernel Modules...
         Starting File System Check on Root Device...
         Starting udev Coldplug all Devices...
         Starting Create list of re…odes for the current kernel...
[  OK  ] Reached target Swap.
[  OK  ] Reached target System Time Synchronized.
[  OK  ] Set up automount Arbitrary…s File System Automount Point.
[  OK  ] Reached target Remote File Systems.
         Starting Restore / save the current clock...
[  OK  ] Created slice system-serial\x2dgetty.slice.
[  OK  ] Reached target Paths.
[  OK  ] Listening on Syslog Socket.
         Starting Journal Service...
[   13.300550] Driver for 1-wire Dallas network protocol.
         Mounting Kernel Debug File System...
[  OK  ] Reached target Slices.
[   13.485500] pinctrl-single 44e10800.pinmux: Invalid number of rows: 0
[   13.530184] pinctrl-single 44e10800.pinmux: no pins entries for pinmux_P2_03_gpio_input_pin
         Mounting POSIX Message Queue File System..[   13.586659] mikrobus: probe of mikrobus-0 failed with error -22
.
[  OK  ] Listening on initctl Compatibility Named Pipe.
[  OK  ] Started Load Kernel Modules.
[  OK  ] Started File System Check on Root Device.
[  OK  ] Started Create list of req… nodes for the current kernel.
[  OK  ] Started Restore / save the current clock.
[  OK  ] Mounted Kernel Debug File System.
[  OK  ] Mounted POSIX Message Queue File System.
[  OK  ] Started File System Check Daemon to report status.
         Starting Remount Root and Kernel File Systems...
         Starting Apply Kernel Variables...
         Mounting FUSE Control File System...
         Mounting Kernel Configuration File System...
[   14.541438] EXT4-fs (mmcblk0p1): re-mounted. Opts: errors=remount-ro. Quota mode: none.
[  OK  ] Started Apply Kernel Variables.
[  OK  ] Started Remount Root and Kernel File Systems.
[  OK  ] Mounted FUSE Control File System.
[  OK  ] Mounted Kernel Configuration File System.
         Starting Load/Save Random Seed...
         Starting Create System Users...
[  OK  ] Started Load/Save Random Seed.
[  OK  ] Started Set the console keyboard layout.
[  OK  ] Started Create System Users.
         Starting Create Static Device Nodes in /dev...
[  OK  ] Started Journal Service.
         Starting Flush Journal to Persistent Storage...
[  OK  ] Started Create Static Device Nodes in /dev.
[  OK  ] Reached target Local File Systems (Pre).
[  OK  ] Reached target Local File Systems.
         Starting Restore /run/initramfs on shutdown...
         Starting Load AppArmor profiles...
         Starting Set console font and keymap...
         Starting udev Kernel Device Manager...
[  OK  ] Started Restore /run/initramfs on shutdown.
[  OK  ] Started Set console font and keymap.
[   15.943758] systemd-journald[164]: Received request to flush runtime journal from PID 1
[  OK  ] Started Flush Journal to Persistent Storage.
         Starting Create Volatile Files and Directories...
[  OK  ] Started udev Kernel Device Manager.
[   16.317735] audit: type=1400 audit(1627674770.848:2): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/s
bin/haveged" pid=195 comm="apparmor_parser"
[   16.416752] audit: type=1400 audit(1627674770.948:3): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia
_modprobe" pid=199 comm="apparmor_parser"
[   16.486923] audit: type=1400 audit(1627674770.976:4): apparmor="STATUS" operation="profile_load" profile="unconfined" name="nvidia
_modprobe//kmod" pid=199 comm="apparmor_parser"
[   16.572645] audit: type=1400 audit(1627674771.088:5): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/b
in/man" pid=202 comm="apparmor_parser"
[   16.654665] audit: type=1400 audit(1627674771.088:6): apparmor="STATUS" operation="profile_load" profile="unconfined" name="man_fi
lter" pid=202 comm="apparmor_parser"
[   16.733893] audit: type=1400 audit(1627674771.088:7): apparmor="STATUS" operation="profile_load" profile="unconfined" name="man_gr
off" pid=202 comm="apparmor_parser"
[  OK  ] Started Create Volatile Files and Directories.
[  OK  ] Started Load AppArmor profiles.
[   16.800762] audit: type=1400 audit(1627674771.256:8): apparmor="STATUS" operation="profile_load" profile="unconfined" name="/usr/s
bin/ntpd" pid=204 comm="apparmor_parser"
[  OK  ] Started Entropy daemon using the HAVEGE algorithm.
         Starting Update UTMP about System Boot/Shutdown...
[  OK  ] Started Update UTMP about System Boot/Shutdown.
[  OK  ] Started udev Coldplug all Devices.
[  OK  ] Reached target System Initialization.
[  OK  ] Started Daily Cleanup of Temporary Directories.
[  OK  ] Started Daily rotation of log files.
[  OK  ] Started Daily man-db regeneration.
[  OK  ] Reached target Timers.
[  OK  ] Listening on Avahi mDNS/DNS-SD Stack Activation Socket.
[  OK  ] Listening on D-Bus System Message Bus Socket.
[  OK  ] Reached target Sockets.
[  OK  ] Reached target Basic System.
[  OK  ] Started D-Bus System Message Bus.
         Starting Connection service...
[  OK  ] Started Regular background program processing daemon.
         Starting controls configuration of serial ports...
         Starting System Logging Service...
         Starting WPA supplicant...
         Starting Login Service...
         Starting LSB: Start busybox udhcpd at boot time...
         Starting Avahi mDNS/DNS-SD Stack...
         Starting controls configuration of serial ports...
         Starting LSB: Load kernel …d to enable cpufreq scaling...
         Starting Helper to synchronize boot up for ifupdown...
[  OK  ] Started System Logging Service.
[  OK  ] Started Helper to synchronize boot up for ifupdown.
[  OK  ] Started LSB: Start busybox udhcpd at boot time.
[  OK  ] Started controls configuration of serial ports.
[  OK  ] Started Login Service.
         Starting Raise network interfaces...
[  OK  ] Started WPA supplicant.
[  OK  ] Started Connection service.
[  OK  ] Started controls configuration of serial ports.
[  OK  ] Created slice system-getty.slice.
[  OK  ] Started Avahi mDNS/DNS-SD Stack.
[  OK  ] Started Raise network interfaces.
[  OK  ] Started LSB: Load kernel m…ded to enable cpufreq scaling.
         Starting LSB: set CPUFreq kernel parameters...
[  OK  ] Reached target Network.
         Starting OpenBSD Secure Shell server...
[  OK  ] Started Unattended Upgrades Shutdown.
         Starting Generic Board Startup...
         Starting Network Time Service...
         Starting Permit User Sessions...
         Starting A high performanc… and a reverse proxy server...
[  OK  ] Reached target Network is Online.
         Starting LSB: exim Mail Transport Agent...
[  OK  ] Started Permit User Sessions.
[  OK  ] Started OpenBSD Secure Shell server.
[  OK  ] Started Getty on tty1.
[  OK  ] Started LSB: set CPUFreq kernel parameters.
[  OK  ] Started Network Time Service.
         Starting Hostname Service...
[  OK  ] Started A high performance…er and a reverse proxy server.
[   29.326324] using random self ethernet address
[   29.335042] using random host ethernet address
[  OK  ] Started Hostname Service.
[   30.006315] using random self ethernet address
[   30.014163] using random host ethernet address
[   30.310597] usb0: HOST MAC 22:91:f3:f2:fb:97
[   30.335286] usb0: MAC a2:ba:78:15:2b:cd
[   30.362592] usb1: HOST MAC d2:59:5f:6a:33:c3
[   30.385296] usb1: MAC 42:07:1c:e0:78:58
[   31.936355] IPv6: ADDRCONF(NETDEV_CHANGE): usb1: link becomes ready
```

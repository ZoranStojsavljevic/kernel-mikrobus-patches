### Mikrobus patches, used to patch both pre-released and stable kernel trees

#### The latest mikrobus patches are presented here
https://github.com/vaishnav98/bb-kernel/tree/clickid/patches/drivers/mikrobus

#### Actual Mikrobus Patches
https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/main/drivers/mikrobus

	0001-mikrobus-driver-update-patch.patch
	0002-mikrobus_add-check-for-missing-w1-gpio.patch
	0003-mikrobus-add-re-enter-id-mode-and-fix-multiple-port.patch
	0004-w1-change-device-name-in-w1_attach_slave.patch
	0005-mikrobus-fix-mikrobus-id-rescan.patch
	0006-greybus-add-gb_netlink-patch.patch
	0007-mikrobus-mikrobus-over-greybus.patch

They are working with the latest Linux trees kernels, with Robert C. Nelson's patches:

https://github.com/RobertCNelson/bb-kernel/tree/am33x-v5.13/patches

The mikrobus_core.c compilation introduces the following compiling warnings!

	  CC [M]  drivers/misc/mikrobus/mikrobus_core.o
	In file included from ./include/linux/device.h:15,
	                 from ./include/linux/w1.h:9,
	                 from drivers/misc/mikrobus/mikrobus_core.c:20:
	drivers/misc/mikrobus/mikrobus_core.c: In function 'mikrobus_port_gb_register':
	drivers/misc/mikrobus/mikrobus_core.c:894:23: warning: format '%lu' expects argument of type 'long unsigned int', but argument 3 has type 'size_t' {aka 'unsigned int'} [-Wformat=]
	  894 |   dev_err(&port->dev, "failed to parse manifest, size %lu\n",
	      |                       ^~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
	./include/linux/dev_printk.h:19:22: note: in definition of macro 'dev_fmt'
	   19 | #define dev_fmt(fmt) fmt
	      |                      ^~~
	drivers/misc/mikrobus/mikrobus_core.c:894:3: note: in expansion of macro 'dev_err'
	  894 |   dev_err(&port->dev, "failed to parse manifest, size %lu\n",
	      |   ^~~~~~~
	drivers/misc/mikrobus/mikrobus_core.c:894:57: note: format string is defined here
	  894 |   dev_err(&port->dev, "failed to parse manifest, size %lu\n",
	      |                                                       ~~^
	      |                                                         |
	      |                                                         long unsigned int
	      |                                                       %u
	drivers/misc/mikrobus/mikrobus_core.c:840:20: warning: unused variable 'desc' [-Wunused-variable]
	  840 |  struct gpio_desc *desc;
	      |                    ^~~~

In order to make work Actual Mikrobus Patches, there were few changes to be done to
files 0001-mikrobus-driver-update-patch.patch & 0006-greybus-add-gb_netlink-patch.patch
to make latest kernels compile/link.

There is well known fact that Robert C Nelson made mikrobus.h as addition and
available in the kernel tree (positioned in the following location):

	.../include/linux/mikrobus.h

This file is known as mikrobus_core.h (and included in the mikrobus driver itself)
from the following Out Of Tree mikrobus driver:

https://github.com/ZoranStojsavljevic/experimental_mikrobus/tree/mikrobusv3

#### Latest Mikrobus Patches
https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/main/doc

	0001-mikrobus-driver-update-patch.patch
	0002-mikrobus_add-check-for-missing-w1-gpio.patch
	0003-mikrobus-add-re-enter-id-mode-and-fix-multiple-port.patch
	0004-w1-change-device-name-in-w1_attach_slave.patch
	0005-mikrobus-fix-mikrobus-id-rescan.patch
	0006-greybus-add-gb_netlink-patch.patch
	0007-mikrobus-mikrobus-over-greybus.patch
	0008-drivers-mikrobus-changes-for-new-Click-ID-Mechanism-.patch
	0009-drivers-misc-mikrobus-update-for-click-ID-adapter-wr.patch

These patches are not tested, not used still by author, thus they are in unknown
state (especially last two patches) for time being!

#### Overlays
https://github.com/ZoranStojsavljevic/kernel-mikrobus-patches/tree/main/drivers/ti/overlays

	0001-ARM-DT-Enable-symbols-when-CONFIG_OF_OVERLAY-is-used.patch

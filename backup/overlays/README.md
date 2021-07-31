### [PATCH] Adding modified arch/arm/boot/dts/am335x-pocketbeagle.dts

File 0001-Adding-modified-arch-arm-boot-dts-am335x-pocketbeagl.patch

Modification of arch/arm/boot/dts/am335x-pocketbeagle.dts so overlays can work

### [PATCH] ARM: DT: Enable symbols when CONFIG_OF_OVERLAY is used

File 0002-ARM-DT-Enable-symbols-when-CONFIG_OF_OVERLAY-is-used.patch

The first part of this patch is obsolete
(file 0002-ARM-DT-Enable-symbols-when-CONFIG_OF_OVERLAY-is-used.patch removed):

```
diff --git a/arch/arm/boot/Makefile b/arch/arm/boot/Makefile
index 0b3cd7a33..87a8b8db3 100644
--- a/arch/arm/boot/Makefile
+++ b/arch/arm/boot/Makefile
@@ -29,6 +29,10 @@ export ZRELADDR INITRD_PHYS PARAMS_PHYS
 
 targets := Image zImage xipImage bootpImage uImage
 
+ifeq ($(CONFIG_OF_OVERLAY),y)
+DTC_FLAGS += -@
+endif
+
 ifeq ($(CONFIG_XIP_KERNEL),y)
 
 cmd_deflate_xip_data = $(CONFIG_SHELL) -c \
```

The second part of this patch is part of the RPi patch
(file 0001-Overlays-Port-RPi-Overlay-building.patch):

```
diff --git a/arch/arm/boot/dts/Makefile b/arch/arm/boot/dts/Makefile
index 8e5d4ab4e75e..364971a93d70 100644
--- a/arch/arm/boot/dts/Makefile
+++ b/arch/arm/boot/dts/Makefile
@@ -1,4 +1,9 @@
 # SPDX-License-Identifier: GPL-2.0
+
+ifeq ($(CONFIG_OF_OVERLAY),y)
+DTC_FLAGS += -@
+endif
+
 dtb-$(CONFIG_ARCH_ALPINE) += \
 	alpine-db.dtb
 dtb-$(CONFIG_MACH_ARTPEC6) += \
@@ -1438,3 +1443,8 @@ dtb-$(CONFIG_ARCH_ASPEED) += \
 	aspeed-bmc-portwell-neptune.dtb \
 	aspeed-bmc-quanta-q71l.dtb \
 	aspeed-bmc-supermicro-x11spi.dtb
+
+targets += dtbs dtbs_install
+targets += $(dtb-y)
+
+subdir-y	:= overlays
```

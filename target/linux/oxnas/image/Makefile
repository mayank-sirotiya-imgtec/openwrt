#
# Copyright (C) 2013 OpenWrt.org
#
# This is free software, licensed under the GNU General Public License v2.
# See /LICENSE for more information.
#
include $(TOPDIR)/rules.mk
include $(INCLUDE_DIR)/image.mk

STG212_UBI_OPTS = -m 2048 -p 128KiB -s 512
STG212_UBIFS_OPTS = -m 2048 -e 126KiB -c 4096

KD20_UBI_OPTS = -m 2048 -p 128KiB -s 512
KD20_UBIFS_OPTS = -m 2048 -e 126KiB -c 4096

POGOPLUG_PRO_UBI_OPTS = -m 2048 -p 128KiB -s 512
POGOPLUG_PRO_UBIFS_OPTS = -m 2048 -e 126KiB -c 4096

POGOPLUG_V3_UBI_OPTS = -m 2048 -p 128KiB -s 512
POGOPLUG_V3_UBIFS_OPTS = -m 2048 -e 126KiB -c 4096

VMLINUX:=$(BIN_DIR)/$(IMG_PREFIX)-vmlinux
UIMAGE:=$(BIN_DIR)/$(IMG_PREFIX)-uImage

define Image/BuildKernel/Template
	$(CP) $(DTS_DIR)/ox820-$(1).dtb $(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb

	$(call Image/BuildKernel/MkFIT,$(1),$(KDIR)/zImage,$(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb,none,0x60008000,0x60008000)
	$(CP) $(KDIR)/fit-$(1).itb $(BIN_DIR)/$(IMG_PREFIX)-$(1)-fit-uImage.itb

 ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
	$(call Image/BuildKernel/MkFIT,$(1),$(KDIR)/zImage-initramfs,$(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb,none,0x60008000,0x60008000,-initramfs)
	$(CP) $(KDIR)/fit-$(1)-initramfs.itb $(BIN_DIR)/$(IMG_PREFIX)-$(1)-fit-uImage-initramfs.itb
	if [ -e "$(KDIR)/u-boot.bin" ]; then \
	( dd if=$(KDIR)/u-boot.bin bs=128k conv=sync ; dd if=$(KDIR)/fit-$(1)-initramfs.itb bs=128k conv=sync ) \
		> $(BIN_DIR)/$(IMG_PREFIX)-$(1)-u-boot-initramfs.bin; \
	fi
 endif

	$(CP) $(KDIR)/zImage $(BIN_DIR)/$(IMG_PREFIX)-zImage
	$(call Image/BuildKernel/MkuImage, \
		none, 0x60008000, 0x60008000, \
		$(BIN_DIR)/$(IMG_PREFIX)-zImage, \
		$(BIN_DIR)/$(IMG_PREFIX)-uImage \
	)

 ifneq ($(CONFIG_TARGET_ROOTFS_INITRAMFS),)
	$(CP) $(KDIR)/zImage-initramfs $(BIN_DIR)/$(IMG_PREFIX)-zImage-initramfs
	$(call Image/BuildKernel/MkuImage, \
		none, 0x60008000, 0x60008000, \
		$(BIN_DIR)/$(IMG_PREFIX)-zImage-initramfs, \
		$(BIN_DIR)/$(IMG_PREFIX)-uImage-initramfs \
	)
 endif
endef

define Image/InstallKernel/Template

 ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_KERNEL),)
	$(INSTALL_DIR) $(TARGET_DIR)/boot
   ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_UIMAGE),)
	$(CP) $(BIN_DIR)/$(IMG_PREFIX)-uImage $(TARGET_DIR)/boot/
	ln -sf $(IMG_PREFIX)-uImage $(TARGET_DIR)/boot/uImage
   endif
   ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_ZIMAGE),)
	$(CP) $(BIN_DIR)/$(IMG_PREFIX)-zImage $(TARGET_DIR)/boot/
	ln -sf $(IMG_PREFIX)-zImage $(TARGET_DIR)/boot/zImage
   endif
   ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_FIT),)
	$(CP) $(BIN_DIR)/$(IMG_PREFIX)-$(1)-fit-uImage.itb $(TARGET_DIR)/boot/
	ln -sf $(IMG_PREFIX)-$(1)-fit-uImage.itb $(TARGET_DIR)/boot/uImage.itb
   endif
 endif

 ifneq ($(CONFIG_TARGET_ROOTFS_INCLUDE_DTB),)
	$(INSTALL_DIR) $(TARGET_DIR)/boot
  ifneq ($(1),)
	$(CP) $(BIN_DIR)/$(IMG_PREFIX)-$(1).dtb $(TARGET_DIR)/boot/
	ln -sf $(IMG_PREFIX)-$(1).dtb $(TARGET_DIR)/boot/$(1).dtb
  endif
 endif
endef

define Image/Build/squashfs
	$(call Image/Build/SysupgradeNAND,$(PROFILE_SANITIZED),$(1),$(KDIR)/fit-$(PROFILE_SANITIZED).itb)
	$(call Image/Build/UbinizeImage,$(PROFILE_SANITIZED),--uboot-env --kernel $(KDIR)/fit-$(PROFILE_SANITIZED).itb,$(1),$($(PROFILE)_UBI_OPTS))
	cp $(KDIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-squashfs-ubinized.bin $(BIN_DIR)
endef

define Image/Build/ubifs

 ifneq ($($(PROFILE)_UBIFS_OPTS),)
	$(call Image/Build/SysupgradeNAND,$(PROFILE_SANITIZED),ubifs,$(KDIR)/fit-$(PROFILE_SANITIZED).itb)
	$(call Image/Build/UbinizeImage,$(PROFILE_SANITIZED),--uboot-env --kernel $(KDIR)/fit-$(PROFILE_SANITIZED).itb,ubifs,$($(PROFILE)_UBI_OPTS))
	cp $(KDIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-ubifs-ubinized.bin $(BIN_DIR)
 endif

endef

Image/BuildKernel/Template/POGOPLUG_PRO=$(call Image/BuildKernel/Template,pogoplug-pro)
Image/InstallKernel/Template/POGOPLUG_PRO=$(call Image/InstallKernel/Template,pogoplug-pro)

Image/BuildKernel/Template/POGOPLUG_V3=$(call Image/BuildKernel/Template,pogoplug-v3)
Image/InstallKernel/Template/POGOPLUG_V3=$(call Image/InstallKernel/Template,pogoplug-v3)

Image/BuildKernel/Template/STG212=$(call Image/BuildKernel/Template,stg212)
Image/InstallKernel/Template/STG212=$(call Image/InstallKernel/Template,stg212)

Image/BuildKernel/Template/KD20=$(call Image/BuildKernel/Template,kd20)
Image/InstallKernel/Template/KD20=$(call Image/InstallKernel/Template,kd20)

define Image/BuildKernel
	$(call Image/BuildKernel/Template/$(PROFILE))
endef

define Image/InstallKernel
	$(call Image/InstallKernel/Template/$(PROFILE))
endef

define Image/Build
	$(if $(Image/Build/$(1)), \
		$(call Image/Build/$(1),$(1)), \
		$(CP) $(KDIR)/root.$(1) $(BIN_DIR)/$(IMG_PREFIX)-$(PROFILE_SANITIZED)-$(1).img \
	)
endef

$(eval $(call BuildImage))

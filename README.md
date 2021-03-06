
![Creator logo](creatorlogo.png)

# Using OpenWrt on Creator (Ci40) Marduk platform

#### For more details about the platform please visit [Creator (Ci40) Marduk](https://community.imgtec.com/platforms/creator-ci40/)
----

## Overview

OpenWrt is a highly extensible GNU/Linux distribution for embedded devices. Instead of trying to create a single, static firmware,
OpenWrt provides a fully writable filesystem with optional package management. This frees you from the restrictions of the application
 selection and configuration provided by the vendor and allows you to use packages to customize an embedded device to suit any application.
For developers, OpenWrt provides a framework to build an application without having to create a complete firmware image and distribution around it.
For users, this means the freedom of full customisation, allowing the use of an embedded device in ways the vendor never envisioned.

Creator is MIPS based platform series from Imagination and Marduk is the latest board under that series. Pistachio is the name of Imagination's SoC based on MIPS.
This guide helps as a quick start but for full details please see the [OpenWrt project wiki](http://wiki.openwrt.org/doc/start).

## Package Content

The release distribution is structured as documented in the upstream project however
there are some key paths where IMG have added solutions:

| Folder              				| Content                                              							|
| :----               				| :----                                                							|
| target/linux/pistachio			| IMG pistachio SoC based board configs and makefiles   						|
| target/linux/pistachio/config-4.1		| IMG pistachio SoC specific kernel config							        |
| target/linux/pistachio/marduk/profiles/marduk_cc2520.mk	| IMG Marduk platform profile with TI cc2520	|
| target/linux/pistachio/marduk/profiles/marduk_ca8210.mk	| IMG Marduk platform profile with Cascoda ca8210  |
## Getting started

Firstly, to obtain a copy of the (Ci40) Marduk platform supported OpenWrt source code:

 * Sign up for a GitHub account

 * Install Git:  ```` sudo apt-get install git ````

 * Clone the repository: ```` git clone https://github.com/CreatorDev/openwrt.git ````

To simply make a build based on the IMG default config run the following commands:

    $ cd openwrt
    $ ./scripts/feeds update -a
    $ ./scripts/feeds install -a

_Ignore any "WARNING: No feed for package..." from the install feeds step._

Load Marduk platform specific OpenWrt configuration for Pistachio.

    $ make menuconfig

1. Select the "Target System" as IMG MIPS Pistachio

        Target System (Atheros AR7xxx/AR9xxx) --->(X) IMG MIPS Pistachio

2. Check the "Target Profile" is set to Basic platform profile for Marduk

        Target Profile (Basic platform profile for Marduk)  --->
            (X) Basic platform profile for Marduk with TI cc2520
            ( ) Basic platform profile for Marduk with Cascoda ca8210

Alternatively, you can use default configuration for Marduk platform with TI cc2520 specific OpenWrt configuration for IMG Pistachio by copying following into .config file:

    $ cat target/linux/pistachio/creator-platform-default.config > .config

Similarly, you can do the same for Marduk platform with Cascoda ca8210 specific OpenWrt configuration:

    $ cat target/linux/pistachio/creator-platform-cascoda-default.config > .config

You can add git revision number as DISTRIB_REVISION in the openwrt image by doing following:

    $ getver.sh . > version

Now build OpenWrt in standard way:

    $ make V=s -j1

Once the build is completed, you will find the resulting output i.e. images, dtbs and rootfs at "bin/pistachio", depending upon the selected profile.
Where PROFILE can be marduk_cc2520, marduk_ca8210, marduk_cc2520_wifi and VERSION is whatever mentioned in CONFIG_VERSION_NUMBER.
By default VERSION is blank if you do not use the creator-platform-default.config for loading the configuration.

- openwrt-$(VERSION)-pistachio-pistachio_$(PROFILE)-uImage
- openwrt-$(VERSION)-pistachio-pistachio_$(PROFILE)-uImage-initramfs
- openwrt-$(VERSION)-pistachio-marduk-$(PROFILE)-rootfs.tar.gz
- pistachio_marduk_cc2520.dtb (for marduk_cc2520 board) or
- pistachio_marduk_ca8210.dtb (for marduk_ca8210 board)

For simplicity, let's assume that marduk_cc2520 PROFILE has been selected and VERSION as 1.0.0 hence the filenames will be as follows: 
- openwrt-1.0.0-pistachio-pistachio_marduk_cc2520-uImage
- openwrt-1.0.0-pistachio-pistachio_marduk_cc2520-uImage-initramfs
- openwrt-1.0.0-pistachio-marduk-marduk_cc2520-rootfs.tar.gz
- pistachio_marduk_cc2520.dtb

## Customising your OpenWrt
You can configure OpenWrt from scratch but it's best to start from a base profile
e.g. the one for the IMG pistachio board as it has some useful defaults.

Simply select the package you need and add into the marduk profile:

    $ vi target/linux/pistachio/marduk/profiles/marduk_cc2520.mk

If you want to add new package, then simply add package name in the list of packages:

    define Profile/marduk
    NAME:=Basic platform profile for Marduk with TI cc2520
    PACKAGES:=kmod-i2c kmod-marduk-cc2520 kmod-sound-pistachio-soc \
                wpan-tools tcpdump uhttpd uboot-envtools \
                alsa-lib alsa-utils alsa-utils-tests \
                iw hostapd wpa-supplicant kmod-uccp420wlan kmod-cfg80211
    endef

Please ensure to remove .config, tmp folders before running make menuconfig.

    $ make menuconfig

Verify that package name is selected. Just save and exit.

## Customising your Linux Kernel

Kernel configuration is saved at target/linux/pistachio/config-4.1
e.g. This the one for the IMG pistachio board as it has some useful defaults.

To customise to suit your requirements use the following command:

    $ make kernel_menuconfig

If you change any option you need and then save & quit GUI, the changed configuration will get written into target/linux/pistachio/config-4.1

For more details please refer to [OpenWrt Build System]("http://wiki.openwrt.org/doc/howto/build")

## Adding Linux Kernel patches
Linux kernel patches are added at :-

    target/linux/pistachio/patches-<kernel_version>/

* All kernel patches are created from the linux kernel hosted [here](https://github.com/CreatorDev/linux).
* Kernel patches for specific kernel version used in OpenWrt are prepared from corresponding branches in [linux](https://github.com/CreatorDev/linux) repo. e.g.

        target/linux/pistachio/patches-4.1/

contains patches created from [openwrt-4.1.13](https://github.com/CreatorDev/linux/tree/openwrt-4.1.13) branch.
* For adding the kernel patch in OpenWrt, create a PR in [linux](https://github.com/CreatorDev/linux) with the change, and also create a PR with the patch of the change in OpenWrt repository.
* Following command can be used for creating the kernel patch :-

        git format-patch <commit_id> --keep-subject --start-number <number>

*NOTE :* Number should the next one from the last patch already added in OpenWrt.

## Serial Console
Connect the Marduk board to development computer over serial port. Open a serial console on host PC.

- In Windows PuTTY, HyperTerminal, or any other free application can be used.

- In Linux, the simplest option is to use the miniterm.py application that comes bundled with Python.

First work out which device you want, you can see them by running:

    $ ls /dev/ttyUSB*

Then connect to it (remember to choose the correct device listed in above command):

    $ sudo miniterm.py /dev/ttyUSB0 -b 115200

## Boot from USB
As a first step, you need to format USB drive into ext4 parition. Partition your media with one big ext4 partion (use dos partition table)
You can check the partition number by following way:

    $ mount | grep /media/dev/sdx1
    on /media/user/43e3311e-7b95-4693-bfcd-b07fa4590a0d type ext4 (rw,nosuid,nodev,uhelper=udisks2)

Alternatively, you can also check the output of the dmesg command.

Next you need to ensure that it has a DOS partition table containing a Linux ext4 partition.
<b>IMPORTANT! Make sure that you replace /dev/sdx for the right device name, e.g. /dev/sdb</b>

Unmount the filesystem before creating new partition table:

    $ sudo umount /dev/sdx?

Create new partition table:

    $ sudo fdisk /dev/sdx
    > enter 'o' to create a new empty DOS partition table
    > enter 'n' to create a new partition; just press return at every prompt to accept the default values
    > enter 'w' to write the new partition table and exit
    $ sudo mkfs.ext4 /dev/sdx1

Mount the USB drive:

    $ sudo mount /dev/sdx1 /mnt/

To put filesystem on USB you will need 'openwrt-1.0.0-pistachio-marduk-marduk-cc2520-rootfs.tar.gz' which is available at "openwrt/bin/pistachio"

Extract the openwrt-1.0.0-pistachio-marduk-marduk-cc2520-rootfs.tar.gz onto the partition you just created:

    $ sudo rm -rf /mnt/*
    $ sudo tar -xf bin/pistachio/openwrt-1.0.0-pistachio-marduk-marduk-cc2520-rootfs.tar.gz -C /mnt/
    $ sudo umount /mnt/

Run "sync" command to synchronize the data properly on the USB drive:

    $ sync

Connect the USB drive and power on the board.

Press Enter on the [Serial Console](#serial-console) to cancel the autoboot and to get the u-boot prompt. Type following command to boot from USB:

    pistachio # run usbboot

To make boot from USB as default boot method, run the following commands on the u-boot prompt (_ignore any warnings shown_):

    pistachio # setenv bootcmd 'run usbboot'
    pistachio # saveenv

## Boot from SD Card
Similar to the instructions for Boot from USB, you need to format the SD card into ext4 parition. Follow the steps mentioned above by making subtle changes

    $ mount | grep /media/dev/sdc1
    on /media/user/5c0dac14-1d47-4a39-8b06-e27861473670 type ext4 (rw,nosuid,nodev,uhelper=udisks2)

SD media essentially mounts at /dev/sdc1 on a computer.  However, make sure to double-check if it gets somehow mounted differently on your computer, in which case, <b>IMPORTANT! Make sure that you replace /dev/sdc with /dev/sdx for the right device name</b>

Unmount the filesystem before creating new partition table:

    $ sudo umount /dev/sdc1

Create new partition table:

    $ sudo fdisk /dev/sdc
    > enter 'o' to create a new empty DOS partition table
    > enter 'n' to create a new partition; just press return at every prompt to accept the default values
    > enter 'w' to write the new partition table and exit
    $ sudo mkfs.ext4 /dev/sdc1

Mount the SD media:

    $ sudo mount /dev/sdc1 /mnt/

To put filesystem on SD you will need 'openwrt-1.0.0-pistachio-marduk-marduk-cc2520-rootfs.tar.gz' which is available at "openwrt/bin/pistachio"

Extract the openwrt-1.0.0-pistachio-marduk-marduk-cc2520-rootfs.tar.gz onto the partition you just created:

    $ sudo rm -rf /mnt/*
    $ sudo tar -xf bin/pistachio/openwrt-1.0.0-pistachio-marduk-marduk-cc2520-rootfs.tar.gz -C /mnt/
    $ sudo umount /mnt/

Run "sync" command to synchronize the data properly on the SD card:

    $ sync

Connect the SD card in the slot and power on the board.

Press Enter on the [Serial Console](#serial-console) to cancel the autoboot and to get the u-boot prompt. Type following command to boot from SD card:

    pistachio # run mmcboot

To make boot from SD card as default boot method, run the following commands on u-boot prompt (_ignore any warnings shown_):

    pistachio # setenv bootcmd 'run mmcboot'
    pistachio # saveenv

## TFTP Boot
For TFTP boot, you will need TFTP server serving kernel image (uImage), dtb (*.dtb)
and initramfs filesystem.

    $ sudo cp bin/pistachio/openwrt-1.0.0-pistachio-pistachio_marduk_cc2520-uImage-initramfs /tftpboot/uImage
    $ sudo cp bin/pistachio/pistachio_marduk_cc2520.dtb /tftpboot/pistachio_marduk.dtb

### Setting up TFTP Server

####Install tftpd package:

    $ sudo apt-get install tftpd

Create /etc/xinetd.d/tftp with following contents:

    service tftp
    {
     protocol        = udp
     port            = 69
     socket_type     = dgram
     wait            = yes
     user            = root
     server          = /usr/sbin/in.tftpd
     server_args     = /tftpboot
     disable         = no
    }

Now restart service:

    $ sudo service xinetd restart

####Configuring U-boot

Now use Serial Console to connect device to host PC. Switch on device and press any key in first 2 seconds. After that you will drop to u-boot console.

To use tftp boot, set the following environment variables

 **Note:**
 - No need to set these environment variables for next boot since these are already saved
 - To get the MAC address, look for a barcode at the bottom side of your board that contains an ID as '0019F5xxxxxx'

1. Set mac address for Ethernet:

        pistachio # setenv ethaddr <00:19:F5:xx:xx:xx>

2. Set Server IP address where TFTP server is running:

        pistachio # setenv serverip <server_ip>

3. Save environment variables:

        pistachio # saveenv

4. Now start tftp boot:

        pistachio # run ethboot

##Boot from Flash

###Flash Partitions

    root@OpenWrt:/# cat /proc/mtd
    dev:    size   erasesize  name
    mtd0: 00180000 00001000 "uboot"
    mtd1: 00002000 00001000 "data-ro"
    mtd2: 00002000 00001000 "uEnv"
    mtd3: 0007c000 00001000 "data-rw"
    mtd4: 10000000 00040000 "firmware0"
    mtd5: 10000000 00040000 "firmware1"

There are two nand paritions `/dev/mtd4, /dev/mtd5` available for NAND boot.
"Dual nandboot" is the default boot method set on Marduk platform. 

There are mutliple ways of flash the ubifs image on one of the NAND parition.

##### Flashing on uboot prompt
Either you can copy the openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img on the USB drive or you can place the same on TFTP server.
To set up TFTP server on your development PC, refer to [Setting up TFTP Server](#setting-up-tftp-server) section.

1. Init flash device on given SPI bus and chip select:

        pistachio # sf probe 1:0

2. Set MAC address and Obtain an IP address (only needed if you are using TFTP server to load the image):

        pistachio # setenv ethaddr <00:19:F5:xx:xx:xx>
        pistachio # dhcp

3. Define flash/nand partitions:

        pistachio # mtdpart default

4. Erase partition:

        pistachio # nand erase.part firmwareX
_firmwareX needs to be replaced with firmware0 or firmware1._

5. Loading the ubifs image

    Copy the openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img to TFTP server (/tftproot) and load from TFTP server:

        pistachio # setenv serverip <development_PC_IP> && tftpboot 0xe000000 openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img
OR
    Copy the openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img to (ext4 formatted) USB drive and load from USB drive:

        pistachio # usb start && ext4load usb 0 0x0E000000 openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img

_Please note that maximum permissible size of ubifs flash image is 16MB. If you need to flash bigger size images, then need to change the load address._

6. Initialize write to nand device:

        pistachio # nand write 0xe000000 firmwareX ${filesize};
_firmwareX needs to be replaced with firmware0 or firmware1._

7. Select the NAND parition to boot from.

        pistachio # setenv boot_partition X
_X needs to be replaced with 0 or 1 depending upon firmware0 or firmware1 respectively._

8. Save dual nand boot environment variables and reboot.

        pistachio # saveenv

##### Flashing on OpenWrt prompt

You can use ubiformat utility to flash the ubifs image when system is booted up and running. *However extra care needs to be taken to select the appropriate mtd partition, as selecting a wrong partition may erase your bootloader completely.*

1. Check the boot partition from which system is booted from.

        root@OpenWrt:/# fw_printenv boot_partition
_If boot_partition is 0, then booted from firmware0 and if 1, then booted from firmware1._

2. Flash the ubifs image on other partition.

        root@OpenWrt:/# ubiformat /dev/mtdX -y -f openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img
_Replace X with **4** or **5** depending upon firmware0 or firmware1 respectively._

3. Select the NAND partition to boot from and reboot.

        root@OpenWrt:/# fw_setenv boot_partition X
_X needs to be replaced with 0 or 1 depending upon firmware0 or firmware1 respectively._

## Building u-boot for pistachio-marduk:

Feed for u-boot is available at [here](https://github.com/Creatordev/openwrt-feeds/tree/master/u-boot)
You can enable u-boot from menuconfig as follows:

    $ make menuconfig
        Boot Loaders  --->
          <*> uboot-pistachio_marduk....................... U-Boot for Pistachio Marduk
          [ ]   Install U-Boot SPL binary image
          [ ]   Environment image  ----  

Build as usual

    $ make V=s -j1

Once built successfully you should be able to see `openwrt-pistachio-pistachio_marduk-u-boot-x.y.z.img` at usual bin/pistachio.

_x.y.z is the tag number as per [u-boot releases](https://github.com/Creatordev/u-boot/releases)_

### Flashing u-boot binary:

You can flash the recently built u-boot image or download a pre-built u-boot binary from [Downloads server](https://downloads.creatordev.io/pistachio/marduk).

1. Enable flashcp in OpenWrt menuconfig and build as usual


        Base system -> busybox -> Cutomize busybox options -> Miscellaneous Utilities -> flashcp
 
2. Erase and write u-boot image on bootloader partition of Ci40


        $ flashcp -v openwrt-pistachio-pistachio_marduk-u-boot-x.y.z.img /dev/mtd0

3. Reboot


        $ reboot

	Once restarted the board, you should see the date when you have built the u-boot on the console as follows:

        		U-Boot SPL 2015.07-rc2 (Jun 14 2016 - 19:20:36)
        		
        		
        		U-Boot 2015.07-rc2 (Jun 14 2016 - 19:20:36 +0530)
        		
        		MIPS(interAptiv): IMG Pistachio 546MHz.
        		Model: IMG Marduk
        		DRAM:  256 MiB
        		NAND:  512 MiB
        		MMC:   Synopsys Mobile storage: 0
        		SF: Detected W25Q16CL with page size 256 Bytes, erase size 4 KiB, total 2 MiB

This means your built u-boot image has been flashed properly on the board!!

_Please be aware that you may brick the board if you flashed a wrong bootloader. Only way to re-cover back the board is to use Dedi-prog SF100 programmer to flash the pre-built bootloader again._

##System upgrade

You can upgrade your existing OpenWrt image using sysupgrade utility. However please note that sysupgrade cannot be used if openwrt image is not there in flash. In such cases, please refer [Flashing on uboot prompt](#flashing-on-uboot-prompt), to update the ubifs image into nand flash. 

You can download the ubifs image from webserver using wget or copy from USB drive.But the image must be put into /tmp as OpenWRT switches to a ramfs to do upgrade.

###Downloading ubifs image from webserver

    root@OpenWrt:/# cd /tmp
    root@OpenWrt:/# wget http://webserver/openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img

###Copying ubifs image from USB drive

    root@OpenWrt:/# mount /dev/sda1 /mnt/
    root@OpenWrt:/# cp /mnt/openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img /tmp

The image will be flashed onto the mtd partition that is not in use (firmware0 or firmware1) then uboot is updated to boot from that partition. 

Pre-requisite :
U-boot environment variables should be set to default as follows:

    pistachio # env default -a
    pistachio # saveenv
This is not required always, but it is required only for the first time when you are doing the system upgrade.

###Fallback mechanism
If image fails to boot in 5 successive attempts, then bootloader will try to boot image from alternate partition.
Uboot variable bootcount is reset after successful boot.

###Upgrading the flash image
Sysupgrade can now be used to flash ubifs images from within OpenWrt:

    root@OpenWrt:/# sysupgrade -v /tmp/openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img

You should see the logs on the console as below:

    root@OpenWrt:/# sysupgrade /tmp/openwrt-1.0.0-pistachio-marduk-marduk_cc2520-ubifs.img
    Saving config files...
    Sending TERM to remaining processes ... logd rpcd netifd odhcpd uhttpd dnsmasq awa_bootstrapd awa_clientd awa_serverd ntpd button_gateway_ button_gateway_ device_manager_ sleep ubusd
    Sending KILL to remaining processes ... device_manager_
    Switching to ramdisk...
    [  612.202026] UBIFS (ubi0:0): background thread "ubifs_bgt0_0" stops
    Performing system upgrade...
    Current boot partiton  1
    Writing image to  firmware0
    .
    .
    sysupgrade successful
    [  631.064624] reboot: Restarting system

On successful, it will restart the system and you should following logs on the console:

    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    BusyBox v1.24.1 (2016-05-02 10:15:17 IST) built-in shell (ash)
    _______                     ________        __
    |       |.-----.-----.-----.|  |  |  |.----.|  |_
    |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
    |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
    -----------------------------------------------------
    DESIGNATED DRIVER (Bleeding Edge, r48138)
    -----------------------------------------------------
    * 2 oz. Orange Juice         Combine all juices in a
    * 2 oz. Pineapple Juice      tall glass filled with
    * 2 oz. Grapefruit Juice     ice, stir well.
    * 2 oz. Cranberry Juice
    $root@OpenWrt:/#
    $root@OpenWrt:/#
    ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

##Configure Network using command line
You can check "ifconfig -a" to check list of interfaces. Ethernet, WiFi and 6loWPAN should be up.

**Note:**

1. 6loWPAN IP has been hardcoded to 2001:1418:0100::1/48. You can change that by editing /etc/config/network script and restarting the network daemon.

        $root@OpenWrt:/# /etc/init.d/network restart

2. You can enable WiFi by default by following below steps:

- set ssid and password for WiFi either at compile time from file target/linux/pistachio/base-files/etc/uci-defaults/config/wireless

        config wifi-iface
            option device       radio0
            option network      sta
            option mode         sta
            option ssid         <XYZ>
            option encryption   psk2
            option key          <Password>

   OR after booting the board, update /etc/config/wireless as above and restart the network by running following command from serial console.

        $root@OpenWrt:/# /etc/init.d/network restart
 
- set default route for WiFi in target/linux/pistachio/base-files/etc/uci-defaults/config/network as

        option 'defaultroute' '1'

### Configure Network using luci web interface
Alternatively, you can also configure network interfaces using luci web interface. Please refer [LUCI Essenetials](https://wiki.openwrt.org/doc/howto/luci.essentials) for more details.

Note that luci is not enabled by default, but you should be able to use opkg utility to install luci.


##Using opkg utility
All the packages built for pistachio-marduk are hosted on [CreatorDev downloads server](https://downloads.creatordev.io/pistachio/marduk), so it should be possible to 
use opkg utility to install/upgrade/remove the OpenWrt packages.

1. Check if you have /etc/opkg/distfeeds.conf pointing to [CreatorDev downloads server](https://downloads.creatordev.io/pistachio/marduk), else for your local development you can edit this to point to your local webserver which has the required packages.

        root@OpenWrt:/# sed -i "s|https://downloads.creatordev.io/pistachio/marduk/|http://localserver:port/|" /etc/opkg/distfeeds.conf

2. If you are using locally built openwrt image and want to update packages from [CreatorDev downloads server](https://downloads.creatordev.io/pistachio/marduk/) then you may need to remove the signature check.

        root@OpenWrt:/# vi /etc/opkg.conf

   Comment out the line `option check_signature 1`  to `#option check_signature 1`

2. Update the list of available packages from this download server.

        root@OpenWrt:/# opkg update

3. Install the intended package as follows:

        root@OpenWrt:/# opkg install fping
        Installing fping (2.4b2_to-ipv6-1) to root...
        Downloading http://downloads.creatordev.io/pistachio/marduk/packages/img/fping_2.4b2_to-ipv6-1_pistachio.ipk.
        Configuring fping.
 
4. In case you have the package already on your board, then it will say so.

        root@OpenWrt:/# opkg install wpa-supplicant
        Package wpa-supplicant (2015-03-25-2) installed in root is up to date.

5. Remove the intended package as follows:

        root@OpenWrt:/# opkg remove fping
        Removing package fping from root...

6. For more information regarding other opkg options, please refer [OPKG manager](https://wiki.openwrt.org/doc/techref/opkg)

##Check your openwrt version
You can check your opewrt version and release information by following:

        root@OpenWrt:/# cat /etc/openwrt_release
        DISTRIB_ID='OpenWrt'
        DISTRIB_RELEASE='0.9.1'
        DISTRIB_REVISION='9403ba4'
        DISTRIB_CODENAME='ci40'
        DISTRIB_TARGET='pistachio/marduk'
        DISTRIB_DESCRIPTION='OpenWrt Ci40 0.9.1'
        DISTRIB_TAINTS='no-all'

Also the device info as follows:

        root@OpenWrt:/# cat /etc/device_info
        DEVICE_MANUFACTURER='Imagination Technologies'
        DEVICE_MANUFACTURER_URL='www.imgtec.com'
        DEVICE_PRODUCT='Creator Ci40(Marduk)'
        DEVICE_REVISION='Rev4 with CC2520'

Depending upon the config chosen to build, `DEVICE_REVISION` will be `Rev4 with CC2520` or `Rev5 with CA8210`.

### Known Issues:

- Cleaned up kernel patches will be upstreamed soon.

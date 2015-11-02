# Plan (Would it work?)
Initially, I intended to boot Ubuntu from SD and install Gentoo to NAND. However, I sadly failed because booting from SD will disable NAND. 

Alternative plan is to make a Gentoo rootfs image. See this one for details: https://www.lisenet.com/2015/install-gentoo-linux-from-debian-wheezy/


## Steps
- Flash Rabian to NAND and boot it
- make an image file (<2GB) and mount it to /mnt/gentoo  
    ```mkfs.ext4 -F -L linuxroot -m 0 -i 8192 linuxroot.img```  
    深坑预警：Inodes may be all comsumed by Gentoo...
- Install Gentoo to /mnt/gentoo with chroot
    - stage3
    - portage
    - setting (CFLAG/USE/locale/etc...)
    - drivers (* not necessary? I will try.)
    - ultilities
- Flash the image to NAND. For rabian image, 
    - uboot.img 0x2000
    - boot.img 0x8000
    - rootfs.img 0xe3a000

## Portage Setting ```/etc/portage/make.conf```
Credit to Mrueg: https://wiki.gentoo.org/wiki/User:Mrueg/Radxa_Rock
```
CHOST="armv7a-hardfloat-linux-gnueabi"
CFLAGS="-march=armv7-a -mtune=cortex-a9 -mfpu=neon -mfloat-abi=hard -fomit-frame-pointer -O2 -pipe"
CXXFLAGS="${CFLAGS}"
```
You may try ```-mfpu=neon-vfpv4``` or ```-mfpu=neon-fp16``` instead of ```-mfpu=neon```, which is not recommended.  
You may add ```MAKEOPTS=--jobs 5 --load-average 4``` for parallel tasks, and if it leads to an error, use ```emerge -j1``` to fix it temporarily.  

## Some tools
- For lsusb, 
```emerge -j4 --ask sys-apps/usbutils 	sys-apps/lshw	sys-apps/pciutils```
- some wireless-tools:
```
emerge -j4 --ask sys-kernel/linux-firmware
emerge -j4 --ask net-wireless/wireless-tools net-wireless/iw
emerge -j4 --ask sys-apps/iproute2 sys-apps/net-tools
emerge -j4 --ask dhcpd
```
- Install nmcli
```
emerge -j4 --ask net-misc/networkmanager
```
## Driver for WiFi adapter
It seems Gentoo rootfs does not contain wifi driver (ethernet seems to be OK).  
In this case, you may ethier build corresponding kernel modules after flashing img with Ethernet, 
    or build those modules somewhere else and copy them to the image.
- IMPORTANT: Manually correct the date and time...(I didn't install a battery on it)...
- Build kernel modules
    - ```emerge --ask dev-vcs/git```
    - Clone subbranch radxa-stable-3.0 for Rock Pro to /usr/src.
        ```cd /usr/src/ && git clone -b radxa-stable-3.0 https://github.com/radxa/linux-rockchip.git```
    - ```ln -s /usr/src/linux-rockchip /usr/src/linux```
    - Modify Makefile ```EXTRAVERSION=+``` (Extremely Important!)
    - I'm not sure whether it is necessary to build kernel first, you can try building modules directly.
    - Build those kernel modules ```mkdir modules && make INSTALL_MOD_PATH=./modules modules modules_install```
    - ```ln -s /usr/src/linux/modules/lib/modules/3.0.36+ /lib/modules/3.0.36+```
    - ```/sbin/depmod -a 3.0.36+ && modprobe 8723au```
    - Reboot.
    - ``nmcli dev wifi connect MY-ACCESS-POINT password MY-PASSWORD``

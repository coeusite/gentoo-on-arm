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

## Network tools
- For lsusb, 
```emerge -j4 --ask sys-apps/usbutils 	sys-apps/lshw	sys-apps/pciutils```
- Rock Pro uses Realtek RTL8723AU wifi chipset with Support Added To Linux 3.15 through r8723au kernel driver
- However, rock pro only contains a kernel of linux 3.0.36.
- In this case, I do doubt whether wifi is OOB or not.
- https://wiki.gentoo.org/wiki/Lenovo_IdeaPad_yoga_13_(i5)
- commands:
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
It seems Gentoo rootfs does not contain wifi driver (ethernet seems to be recognized).  
In this case, you may ethier build corresponding kernel modules after flashing img with Ethernet, 
    or build those modules somewhere else and copy them to the image.
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
    

- Build driver & etc
```
cd /usr/src/driver
git clone https://github.com/lwfinger/rtl8723au.git
cd rtl8723au/
make -j4 
make install && modprobe rtl8723au

git clone https://github.com/lwfinger/rtl8723au_bt.git
cd rtl8723au_bt/
make -j4
make install
```


## Last Question: How to boot Gentoo instead of Rabian?

# Filesystem on a Radxa

## How does it boot?
bootloader => boot to ramdisk => switch to system

## Partitions
- bootloader  [Rabian]
    RK3188Loader(L)_V2.10.bin / RK3188Loader(L)_V2.19.bin
- parameter  [Rabian]
    http://dl.radxa.com/rock/images/parameter/parameter_linux_nand
- boot  [Rabian] => [Gentoo]
    kernel + ramdisk
- system [Rabian / Gentoo]
- recovery 
- misc

# Install Gentoo in Rabian's chroot
- Reference: https://www.lisenet.com/2015/install-gentoo-linux-from-debian-wheezy/
## Prepare Chroot Environment
- install Rabian to NAND
- udpate and install tools
```
    apt-get update
    apt-get upgrade
    apt-get install nano wget bzip2
```
## Installation Media
- Download the Stage3   
    http://distfiles.gentoo.org/releases/arm/autobuilds/current-stage3-armv7a_hardfp/stage3-armv7a_hardfp-20151025.tar.bz2  
- Verity and Extract to /mnt/gentoo  
```
    mkdir /mnt/gentoo  
    sha512sum ./stage3*bz2
    tar xvjpf ./stage3*.bz2 -C /mnt/gentoo/
```
- Download and Install Portage  
```
    wget http://distfiles.gentoo.org/releases/snapshots/current/portage-latest.tar.bz2
    wget http://distfiles.gentoo.org/releases/snapshots/current/portage-latest.tar.bz2.md5sum
    md5sum -c portage-latest.tar.bz2.md5sum 
    tar xvjf ./portage-latest.tar.bz2 -C /mnt/gentoo/usr/
```
- Set according CFLAGS to /mnt/gentoo/etc/portage/make.conf
```
CHOST="armv7a-hardfloat-linux-gnueabi"
CFLAGS="-march=armv7-a -mtune=cortex-a9 -mfpu=neon -mfloat-abi=hard -fomit-frame-pointer -O2 -pipe"
CXXFLAGS="${CFLAGS}"
```

## Installing the Gentoo Base System
- Copy DNS info from Debian and mount the necessary filesystems:
```
    cp -L /etc/resolv.conf /mnt/gentoo/etc/resolv.conf
    mount -t proc none /mnt/gentoo/proc
    mount -o bind /dev /mnt/gentoo/dev
```
- Chroot into the new environment:
```chroot /mnt/gentoo /bin/bash ```

- Reload the environment in memory:
``` /usr/sbin/env-update && source /etc/profile ```

- Remind yourself you are in chroot:
``` export PS1="(chroot) $PS1``` 

- Configure the USE variable according to your needs.   
    Currently I give it up.

- Configure timezone:
``` cp /usr/share/zoneinfo/Hongkong /etc/localtime ``` 

- Configure locales:
```
nano -w /etc/locale.gen
locale-gen
``` 
- Reload the environment in memory:
``` /usr/sbin/env-update && source /etc/profile ``` 

## Configure the Linux Kernel && Compile the Linux Kernel
Skipped.

Trying to use Rabian's kernel to boot Gentoo

## Configure the Gentoo System
- Create the fstab File  
The fstab in Rabian only contains
``` none    /tmp    tmpfs   defaults,noatime,mode=1777 0 0```.  
However, according to https://wiki.gentoo.org/wiki/Systemd/en, systemd will automatically mount tmpfs to /tmp.  
I do wonder whether I can create a blank fstab or not, because it seems something else handles mounting staffs for NAND.

- Host Configuration
```
(chroot)# echo 'hostname="gentoo-rock-pro"' > /etc/conf.d/hostname
```

## Driver for WiFi adapter
- Rock Pro uses Realtek RTL8723AU wifi chipset with Support Added To Linux 3.15 through r8723au kernel driver
- However, rock pro only contains a kernel of linux 3.0.36.
- In this case, I do doubt whether wifi is OOB or not.
- https://wiki.gentoo.org/wiki/Lenovo_IdeaPad_yoga_13_(i5)
- ```emerge --ask sys-kernel/linux-firmware```
- ```emerge --ask net-wireless/wireless-tools net-wireless/iw```
- ```emerge --ask sys-apps/iproute2 sys-apps/net-tools```

### Backup method
```
emerge --ask dev-vcs/git

git clone git@github.com:lwfinger/rtl8723au.git
cd rtl8723au/
make -j8 
make install && modprobe rtl8723au
```
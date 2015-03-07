calaos-os
=========

Calaos OS contains the scripts needed to build Calaos images. The buildsystem is using OpenEmbedded and the scripts comes from the great Angstrom build system.

This fork (tiramiseb/calaos-os) is made especially to support the BeagleBone Black board. It will be merged to "calaos/calaos-os" when the new version of Yocto will be used by Calaos OS, for a better support of this platform.


Supported boards
----------------

The following hardware are supported by the Calaos team:
- Mele A1000 / Mele A2000
- Raspberry Pi
- Intel Atom based board
- NUC intel platform
- Cubieboard

This fork makes Calaos OS work on the following hardware:
- BeagleBone
- BeagleBone Black

However, it has only been validated on BeagleBone Black, without the GUI.

How to build
------------

This is a quick howto for building a fresh Calaos image.

You need first install Yocto dependencies, on ubuntu machine you need to install these packages : 
```
sudo apt-get install gawk wget git-core diffstat unzip texinfo gcc-multilib build-essential chrpath libsdl1.2-dev xterm man
```

Launch the build script without arguments to get the list of options and supported machines:
```bash
./build.sh
```

Before you need to get all required modules and configure for the wanted machine:
```bash
./build.sh init beaglebone
```

Finally you can start a build using bitbake:
```bash
source ./env.sh
bitbake calaos-image
```

You will find the images in tmp-eglibc/deploy/images/

Beaglebone specific : how to install
------------------------------------

The final image is not automatically created, it will probably be created with the new version of Yocto, which will be used later by Calaos OS.

The following steps should make you able to boot on Calaos OS...

Create 2 partitions (FAT16 and ext4) on a microSD card:
```
Disk /dev/mmcblk0: 2008 MB, 2008023040 bytes
4 heads, 16 sectors/track, 61280 cylinders
Units = cylinders of 64 * 512 = 32768 bytes

        Device Boot      Start         End      Blocks  Id System
/dev/mmcblk0p1   *          33        3104       98304   e Win95 FAT16 (LBA)
Partition 1 does not end on cylinder boundary
/dev/mmcblk0p2            3105       54400     1641472  83 Linux
Partition 2 does not end on cylinder boundary
```

Copy MLO, u-boot.img and uImage to the FAT16 partition (example if this partition is mounted on /media/boot) :
```
cp MLO-beaglebone-2013.04 /media/boot/MLO
cp u-boot-beaglebone-2013.04-r0.img /media/boot/u-boot.img
cp uImage--3.8.13-r23a-beaglebone-20141110150906.bin /media/boot/uImage
```

Copy the system files to the ext4 partition (example if this partition is mounted on /media/root) :
```
tar zxvf calaos-image-beaglebone-oe-core.0-20141110150906.rootfs.tar.gz -C /media/root
```

Copy the kernel modules to the ext4 partition :
```
tar zxvf modules--3.8.13-r23a-beaglebone-20141110150906.tgz -C /media/root
```

A good'ol sync...
```
sync
```

Eject the card, put it in your BeagleBone, reboot while pressing on the S2 (boot) button, enjoy.

### Update

My procedure to update Calaos OS is the following.

1. on the Calaos server, make an archive of Calaos configuration:
```
tar zcvf etc_calaos.tar.gz /etc/calaos
```

2. on the computer, copy the Calaos configuration archive:
```
scp <calaos_server>:etc_calaos.tar.gz .
```

3. generate the new image, put it on a microSD card (see instructions above).

4. edit etc/hostname on the microSD card, put the Calaos server's name in there.

5. extract the configuration archive on the microSD card:
```
sudo tar zxvf etc_calaos.tar.gz -C /media/root
```

6. unmount the microSD card.

7. reboot the BeagleBone on the microSD card (see instructions above).

8. find the Calaos server's IP address (maybe amongst the DHCP server leases).

9. connect to the Calaos server with SSH.

10. use connmanctl to get the Ethernet device name:
```
# connmanctl services
*AO Wired                ethernet_d05fb8fecfd3_cable
```

11. use connmanctl to configure a static IP address:
```
connmanctl config ethernet_d05fb8fecfd3_cable --ipv4 manual 192.168.42.251 255.255.255.0 192.168.42.254 --nameservers 192.168.42.254
```

### Enable UART1

I use UART1 to read data from my electricity counter (Téléinformation from ErDF). Create the uEnv.txt file on the boot partition, with the following content:
```
optargs=capemgr.enable_partno=BB-UART1
```

### Boot on microSD card automatically

An option to boot on the microSD card automatically is to remove (or rather rename) the MLO file on the internal boot device. The following commands may be executed when running Calaos OS after booting while long-pressing on S2 :
```
mount /dev/mmcblk1p1 /mnt
mv /mnt/MLO /mnt/MLO.bak
umount /mnt
halt
```

Please note this is not a way to change the multiboot order (i.e. pressing on the S2 button will not boot the BeagleBone on the internal memory) but a way to totally invalidate boot on the internal memory.

### Flash the internal memory (BeagleBone Black)

The BeagleBone Black is equipped with an internal memory, which is an eMMC chip : it acts as a standard MMC memory module. So, flashing it is very similar to the steps needed to flash a SD/MMC card...

The following steps are executed on a Calaos OS instance running on a SD card.

Create 2 partitions (FAT16 and ext4) on the internal memory (this is the default partitioning scheme):
```
Disk /dev/mmcblk1: 3867 MB, 3867148288 bytes
4 heads, 16 sectors/track, 118016 cylinders
Units = cylinders of 64 * 512 = 32768 bytes

        Device Boot      Start         End      Blocks  Id System
/dev/mmcblk1p1   *          33        3104       98304   e Win95 FAT16 (LBA)
/dev/mmcblk1p2            3105      118016     3677184  83 Linux
```

Mount and flush the boot partition:
```
mount /dev/mmcblk1p1 /mnt
rm -fr /mnt/*
```

Copy the MLO, u-boot.img and uImage from the current boot partition:
```
mkdir /mnt2
mount /dev/mmcblk0p1 /mnt2
cp /mnt2/* /mnt
```

Unmount both boot partitions:
```
umount /mnt
umount /mnt2
rmdir /mnt2
```

Mount and flush the root partition:
```
mount /dev/mmcblk1p2 /mnt
rm -fr /mnt/*
```

Copy the system:
```
mkdir /mnt/proc /mnt/sys /mnt/mnt /mnt/media /mnt/run /mnt/tmp
cp -a /bin /boot /dev /etc /home /lib /sbin /usr /var /www /mnt/
```

Unmount the filesystem and shutdown:
```
umount /mnt
halt
```

Now, remove the microSD card, power on the board and enjoy!

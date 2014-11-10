calaos-os
=========

Calaos OS contains the scripts needed to build Calaos images. The buildsystem is using OpenEmbedded and the scripts comes from the great Angstrom build system.


Supported boards
----------------

The following hardware are supported by the Calaos team:
- Mele A1000 / Mele A2000
- Raspberry Pi
- Intel Atom based board
- NUC intel platform
- Cubieboard

Other hardware may be supported but only those one are heavily tested by us. Feel free to try to port to a new hardware, and come to check with us for all the details.

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
./build.sh init <machine>
```

Then you can configure another machine you want to build to:
```bash
./build.sh config raspberrypi
```

Finally you can start a build using bitbake:
```bash
source ./env.sh
bitbake calaos-image
```

You will find the images in tmp-eglibc/deploy/images/

Beaglebone specific : how to install
------------------------------------

The final image is not automatically created, the following steps should make us able to boot on the created stuff :

Create 2 partitions (FAT16 and ext4) on the microSD card :
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

TODO:
- automatically create a single ".img" file for the BeagleBone
- explain how to boot on the microSD card by default
- explain how to flash the internal memory with this system

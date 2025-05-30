---
title: "Bootbaren ESXi-USB Installer mit OS X erzeugen"
date:
  created: 2016-05-13

linktitle: "Bootbaren ESXi-USB Installer mit OS X erzeugen"
slug: "bootbaren-esxi-usb-installer-mit-os-x-erzeugen"

description: "Wie erzeugt man ein USB-Stick mit dem VMware ESXi installiert werden kann?"

tags:
- macOS
- VMware

authors:
- harry
---
Für alles gibt es ja meistens schon eine Lösung, so auch für dieses kleine Problem.
VMware stellt nur ISO Images zum Download bereit, von denen ESXi installiert werden kann.
Nun möchte man aber vielleicht einen USB-Stick verwenden, statt das ISO-Image auf eine CD zu brennen. Dafür gibt es verschiedene Anleitungen. Für OS X muss man vielleicht etwas länger suchen.

<!-- more -->

Aber auf [GitHub](https://github.com/cbednarski/vmware-usb-osx), findet man was passendes :-)

> ### vmware-usb-osx
> This project includes a simple makefile that helps you create a bootable USB installer for VMware ESXi on OSX. This is based on a similar script I made for ubuntu.

> ### Background on VMware ESXi
> VMware ESXi is a lightweight operating system designed to run virtual machines. Whenever you hear about someone running virtual machines or cloud servers they're using ESXi, Xen, QEMU/KVM, or a similar technology under the hood.

> ESXi is proprietary software, but you can download it and use it for free. You will need to register for a VMware account and download ESXi (also called "vSphere Hypervisor") before you use this script.

> ### Instructions
> 1. Download the ESXi ISO and copy it into the same folder as this script. It will have a long name with a version number like `VMware-VMvisor-Installer-6.0.0-2494585.x86_64.iso`. Rename it to `esxi.iso` so the script can find it.
> 2. Run `make devices`
> 3. Insert your USB stick. You'll see a new device appear in step 4.
> 4. Run `make devices`
> 5. Note the disk number for your USB stick. It will be something like /dev/disk2. The only part you care about is 2 (or 3, or 4, or whatever) for step 6.
> 6. `make vmware DISK=2` **Note:** This step will ask for your password for sudo.
> 7. Pop the USB stick into the computer you want to vmware-ify and restart it.

> Depending on how fast your USB stick is, step 6 make take awhile. On my computer it took around a minute and a half.

```
OSX:vmware-usb-osx-master admin$ make devices
diskutil list
...
/dev/disk2 (external, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *8.0 GB     disk2
   1:                 DOS_FAT_32 UNTITLED1               8.0 GB     disk2s1

OSX:vmware-usb-osx-master admin$ make vmware DISK=2
hdiutil convert -format UDRW -o esxi.img esxi.iso
Reading ESXI-6.0.0-20160302001-STANDARD  (Apple_ISO : 0)…
............................................................................................................................................................................................................................................
Elapsed Time:  1.992s
Speed: 179.6Mbytes/sec
Savings: 0.0%
created: /Users/admin/Downloads/vmware-usb-osx-master/esxi.img.dmg
# Format USB device as a bootable MS-DOS volume
diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
diskutil eraseDisk MS-DOS ESXI MBR /dev/disk2
Started erase on disk2
Unmounting disk
Creating the partition map
Waiting for the disks to reappear
Formatting disk2s1 as MS-DOS (FAT) with name ESXI
512 bytes per physical sector
/dev/rdisk2s1: 15636624 sectors in 1954578 FAT32 clusters (4096 bytes/cluster)
bps=512 spc=8 res=32 nft=2 mid=0xf8 spt=32 hds=255 hid=2 drv=0x80 bsec=15667198 bspf=15271 rdcl=2 infs=1 bkbs=6
Mounting disk
Finished erase on disk2
# Mount USB device to add syslinux.cfg
mkdir -p source
mkdir -p target
hdiutil mount esxi.iso -mountpoint ./source
/dev/disk3          	                               	/Users/admin/Downloads/vmware-usb-osx-master/source
cp -r source/ /Volumes/ESXI/
cp syslinux.cfg /Volumes/ESXI/
hdiutil eject ./source
"disk3" unmounted.
"disk3" ejected.
diskutil unmountDisk /dev/disk2
Unmount of all volumes on disk2 was successful
diskutil eject /dev/disk2
Disk /dev/disk2 ejected
```

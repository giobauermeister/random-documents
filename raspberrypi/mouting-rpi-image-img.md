# Mounting RPi sysroot from image file .img for cross development.

## Install dependecies on local workstation

```
sudo apt install qemu-user-static
```

## Download image from official website.

[https://www.raspberrypi.com/software/operating-systems/](https://www.raspberrypi.com/software/operating-systems/)

## Extract xz image file

```
xz -d 2024-07-04-raspios-bookworm-arm64-lite.img.xz
```

## Make image 2Gb bigger to make space for new packages

```
truncate -s +2G 2024-07-04-raspios-bookworm-arm64-lite.img
```

## Expand the image with 100% of the space

```
sudo parted 2024-07-04-raspios-bookworm-arm64-lite.img

(parted) print

Model:  (file)
Disk ~/raspberry/2024-07-04-raspios-bookworm-arm64-lite.img: 9278MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      4194kB  541MB   537MB   primary  fat32        lba
 2      541MB   2835MB  2294MB  primary  ext4

(parted) resizepart 2 100%

(parted) print
Model:  (file)
Disk ~/raspberry/2024-07-04-raspios-bookworm-arm64-lite.img: 9278MB
Sector size (logical/physical): 512B/512B
Partition Table: msdos
Disk Flags: 

Number  Start   End     Size    Type     File system  Flags
 1      4194kB  541MB   537MB   primary  fat32        lba
 2      541MB   9278MB  8737MB  primary  ext4

(parted) quit
```

Or use parted in non interactive mode:

```
sudo parted -s 2024-07-04-raspios-bookworm-arm64-lite-custom.img resizepart 2 100%
```

## Verify image Start value from second partition rootfs.

```
fdisk -l 2024-07-04-raspios-bookworm-arm64-lite.img
```

```
Disk 2024-07-04-raspios-bookworm-arm64-lite.img: 2,64 GiB, 2835349504 bytes, 5537792 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa3f161f3

Device                                      Boot   Start     End Sectors  Size Id Type
2024-07-04-raspios-bookworm-arm64-lite.img1         8192 1056767 1048576  512M  c W95 FAT32 (LBA)
2024-07-04-raspios-bookworm-arm64-lite.img2      1056768 5537791 4481024  2,1G 83 Linux
```

Start = 1056768

## Create folder to mount the rootfs

```
sudo mkdir /mnt/rpi-sysroot
```

## Mount the partition with Start value from above

```
sudo mount -o loop,offset=$((1056768*512)) 2024-07-04-raspios-bookworm-arm64-lite.img /mnt/rpi-sysroot/
```

```
ls -l /mnt/rpi-sysroot/

total 76
lrwxrwxrwx  1 root root     7 jul  3 21:04 bin -> usr/bin
drwxr-xr-x  3 root root  4096 jul  3 21:14 boot
drwxr-xr-x  4 root root  4096 jul  3 21:04 dev
drwxr-xr-x 89 root root  4096 jul  3 21:14 etc
drwxr-xr-x  3 root root  4096 jul  3 21:05 home
lrwxrwxrwx  1 root root     7 jul  3 21:04 lib -> usr/lib
drwx------  2 root root 16384 jul  3 21:13 lost+found
drwxr-xr-x  2 root root  4096 jul  3 21:04 media
drwxr-xr-x  2 root root  4096 jul  3 21:04 mnt
drwxr-xr-x  3 root root  4096 jul  3 21:06 opt
drwxr-xr-x  2 root root  4096 jul  3 21:04 proc
drwx------  3 root root  4096 jul  3 21:04 root
drwxr-xr-x  8 root root  4096 jul  3 21:04 run
lrwxrwxrwx  1 root root     8 jul  3 21:04 sbin -> usr/sbin
drwxr-xr-x  2 root root  4096 jul  3 21:04 srv
drwxr-xr-x  2 root root  4096 mar 29 14:20 sys
drwxrwxrwt  2 root root  4096 jul  3 21:04 tmp
drwxr-xr-x 11 root root  4096 jul  3 21:04 usr
drwxr-xr-x 11 root root  4096 jul  3 21:04 var
```

## Check which loop the partition was mounted

```
lsblk

loop14        7:14   0   8,1G  0 loop /mnt/rpi-sysroot
```

## Resize the filesystem in loop

```
sudo resize2fs /dev/loop14

resize2fs 1.46.5 (30-Dec-2021)
Filesystem at /dev/loop14 is mounted on /mnt/rpi-sysroot; on-line resizing required
old_desc_blocks = 1, new_desc_blocks = 1
The filesystem on /dev/loop14 is now 2132992 (4k) blocks long.
```

## Verify new size

```
df -h /mnt/rpi-sysroot/

Filesystem      Size  Used Avail Use% Mounted on
/dev/loop14     8,0G  2,0G  5,7G  26% /mnt/rpi-sysroot
```

## Mount necessary local system directories (`/dev`, `/proc` e `/sys`) for chroot to work

```
sudo mount -o bind /dev /mnt/rpi-sysroot/dev
sudo mount -o bind /proc /mnt/rpi-sysroot/proc
sudo mount -o bind /sys /mnt/rpi-sysroot/sys
sudo mount -o bind /dev/pts /mnt/rpi-sysroot/dev/pts
```

## Copy qemu-arm-static

```
sudo cp /usr/bin/qemu-arm-static /mnt/rpi-sysroot/bin
```

or for 64 bit OS

```
sudo cp /usr/bin/qemu-aarch64-static /mnt/rpi-sysroot/bin
```

## Enter chroot

```
sudo chroot /mnt/rpi-sysroot
```

## Install all necessary packages with apt

```
apt update
apt install ...
```

For example install Qt6 related packages:

```
apt install libboost-all-dev libudev-dev libinput-dev libts-dev libmtdev-dev libjpeg-dev libfontconfig1-dev libssl-dev libdbus-1-dev libglib2.0-dev libxkbcommon-dev libegl1-mesa-dev libgbm-dev libgles2-mesa-dev mesa-common-dev libasound2-dev libpulse-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev  gstreamer1.0-alsa libvpx-dev libsrtp2-dev libsnappy-dev libnss3-dev "^libxcb.*" flex bison libxslt-dev ruby gperf libbz2-dev libcups2-dev libatkmm-1.6-dev libxi6 libxcomposite1 libfreetype6-dev libicu-dev libsqlite3-dev libxslt1-dev libavcodec-dev libavformat-dev libswscale-dev libx11-dev freetds-dev libpq-dev libiodbc2-dev firebird-dev libxext-dev libxcb1 libxcb1-dev libx11-xcb1 libx11-xcb-dev libxcb-keysyms1 libxcb-keysyms1-dev libxcb-image0 libxcb-image0-dev libxcb-shm0 libxcb-shm0-dev libxcb-icccm4 libxcb-icccm4-dev libxcb-sync1 libxcb-sync-dev libxcb-render-util0 libxcb-render-util0-dev libxcb-xfixes0-dev libxrender-dev libxcb-shape0-dev libxcb-randr0-dev libxcb-glx0-dev libxi-dev libdrm-dev libxcb-xinerama0 libxcb-xinerama0-dev libatspi2.0-dev libxcursor-dev libxcomposite-dev libxdamage-dev libxss-dev libxtst-dev libpci-dev libcap-dev libxrandr-dev libdirectfb-dev libaudio-dev libxkbcommon-x11-dev libvulkan-dev vulkan-tools mesa-vulkan-drivers libbluetooth-dev bluez bluez-tools
```

And create folders as necessary, for example:

```
mkdir /usr/local/qt6
```

**NOTE**: if error `no space left on device` occurs, you need to make image even bigger with truncate command.

**NOTE**: if error `Could not resolve host: github.com` occurs, or if you cannot ping google.com, copy resolv.conf

```
sudo cp /etc/resolv.conf /mnt/rpi-sysroot/etc/resolv.conf
```

## Exit chroot and umount all sysroot

```
exit
sudo umount /mnt/rpi-sysroot/dev/pts
sudo umount /mnt/rpi-sysroot/dev
sudo umount /mnt/rpi-sysroot/proc
sudo umount /mnt/rpi-sysroot/sys
sudo umount /mnt/rpi-sysroot
```

## Install image and start cross developing!

Install `2024-07-04-raspios-bookworm-arm64-lite.img` as usual and use `/mnt/rpi-sysroot` as sysroot for cross development

## Compress image for small image size if desired.

```
xz 2024-07-04-raspios-bookworm-arm64-lite.img
```

Use `-k` to keep uncompressed .img file

```
xz -k 2024-07-04-raspios-bookworm-arm64-lite.img
```
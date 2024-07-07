# minimal-Linux-with-busybox
minimal (bootable) Linux with busybox, for both qemu and real machine.

To build a bootable linux system from, the linux kernel and a root file system is needed in minimal. So in this guide, we first build a linux kernel, and then build a static linked busybox and finally create a bootable linux system.

## Before build
You may need to install the required packages first:
```shell
sudo apt install build-essential libncurses-dev bison flex libssl-dev libelf-dev
```

## Build Linux kernel from source
First, download linux kernel source code from [kernel.org](https://kernel.org/) or use github mirror. In this guide, we select version 5.15 as an example.
```shell
wget https://mirrors.edge.kernel.org/pub/linux/kernel/v5.x/linux-5.15.tar.xz
```
Then unzip xz archive:
```shell
tar -xvf linux-5.15.tar.xz
cd linux-5.15
```
Now you can create default build config:
```shell
make defconfig
```
We can simply use this default config generated by defconfig, if you want change config file to enable or disable some features, DO NOT directlt edit .config file, you can use menuconfig to edit it through a TUI.
```shell
make menuconfig
```
Finally we can start to compile the kernel. You can set max parallel process number by -j flag.
```shell
make -j8 # or use make -j$(nproc) to enable all cores.
```
Old kernel version needs old gcc, so if you found some compile time issues, use this command to set gcc version.
```shell
make HOSTCC=gcc-11 CC=gcc-11 -j8
```
After build process, you can find kernel image in the following path:
```shell
arch/x86_64/boot/bzImage # replace x86_64 to your system architecture.
```
You can copy this image file to project root directory.
```shell
cp arch/x86_64/boot/bzImage ../
```

## Build busybox from source
You can download busybox source code from [busybox.net](https://www.busybox.net/), or download from **release**.
```shell
wget https://www.busybox.net/downloads/busybox-1.36.1.tar.bz2
```
Then unzip bz archive:
```shell
tar -xvf busybox-1.36.1.tar.bz2
cd busybox-1.36.1
```
Since we need a static-linked binary, we should enable static build by menuconfig TUI and set __Build static binary (no shared libs)__. 
```shell
make defconfig # create default config file first.
make menuconfig
```
Now you can start to compile the busybox.
```shell
make -j8 # or use make -j$(nproc) to enable all cores.
```
After build process, you can create a linux-like folder by simple run:
```shell
make install
```
You can find a new folder **_install** is created.

## Create rootfs
Now we can create rootfs based on busybox.
```shell
mkdir -p rootfs # outside busybox dir.
cd rootfs
mkdir -p bin sbin etc usr/bin usr/sbin dev sys proc
```
Copy all contains in **_install** to **rootfs**.
```shell
cp -a ./busybox-1.36.1/_install/* ./rootfs
```
Now create a executable file called **init** inside **rootfs** folder. In this guide, **init** is a shell script, but you can also use any executable as the **init** file.
```shell
#!/bin/sh

mount -t devtmpfs devtmpfs /dev
mount -t proc none /proc
mount -t sysfs none /sys

echo "Minimal linux with busybox"
exec /bin/sh

poweroff -f
```
And then:
```shell
chmod +x init
```
Create rootfs image:
```shell
find . -print0 | cpio --null -ov --format=newc | gzip -9 > ../rootfs.cpio.gz
```


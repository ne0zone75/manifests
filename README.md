# Rockchip Debian SDK
    
Below is the instructions of how to build image for ROCK960.

## Requirement

You need a type C to type A cable(USB 2.0 or USB 3.0) to flash ROCK960.

## Get the source code

You need repo to get multiple git repositories to build the image.

Install repo if you don't have it.

    mkdir ~/bin
    PATH=~/bin:$PATH
    curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
    chmod a+x ~/bin/repo

sync the source code

    repo init -u https://github.com/96rocks/manifests -m rock960.xml
    repo sync
    repo start rock960-dev --all

You will get 

    build  kernel  README.md  rkbin  rootfs  u-boot

## Install toolchain and other build tools

    sudo apt-get install gcc-aarch64-linux-gnu device-tree-compiler libncurses5 libncurses5-dev build-essential

## Build u-boot

    ./build/mk-uboot.sh rock960

The generated images will be copied to out/u-boot folder

    ls out/u-boot/
    idbloader.img  rk3399_loader_v1.08.106.bin  trust.img  uboot.img

## Build kernel

    ./build/mk-kernel.sh rock960

You will get the kernel image and dtb file

    ls out/kernel/
    Image  rock960-linux.dtb

## Make rootfs image

Building a base debian system by ubuntu-build-service from linaro.

    sudo apt-get install binfmt-support qemu-user-static
    sudo dpkg -i ubuntu-build-service/packages/*        # ignore the broken dependencies, we will fix it next step
    sudo apt-get install -f
    RELEASE=stretch TARGET=desktop ARCH=armhf ./mk-base-debian.sh

This will bootstrap a Debian stretch image, you will get a rootfs tarball named `linaro-stretch-alip-xxxx.tar.gz`. 

Building the rk-debain rootfs with debug:

    VERSION=debug ARCH=armhf ./mk-rootfs-stretch.sh  && ./mk-image.sh

This will install Rockchip specified packages and hooks on the standard Debian rootfs and generate an ext4 format rootfs image at `rootfs/linaro-rootfs.img` .

## Combine everything into one image

    build/mk-image.sh -c rk3399 -t system -r rootfs/linaro-rootfs.img

This will combine u-boot, kernel and rootfs into one image and generate GPT partition table. Output is out/system.img

## Flash the image

Press and hold the **Maskrom** key then press reset, the board will boot into [maskrom](http://opensource.rock-chips.com/wiki_Rockusb#Maskrom_mode) mode. Now connect the Sapphire and your PC with type C to type A cable.

    build/flash_tool.sh   -c rk3399 -p system  -i  out/system.img

This will flash the image to on board eMMC from the type C USB port. After flashing finished, the board will reboot, now you boot into Debian Linux.

## Troubleshooting

### Can not go to maskrom mode

1. Press and hold maskrom key longer, and short press and release reset.
2. Check your usb cable, plug and unplug the usb cable, reverse plug the type C port and try
3. On the host PC, lsusb should show the following VID/PID if the board is in maskrom mode:

    Bus 003 Device 061: **ID 2207:0011**

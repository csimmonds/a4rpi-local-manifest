# Building Android for Raspberry Pi

# General notes
This is project experimental; it is not guaranteed to build on all possible
systems and it is not guaranteed to run on all Raspberry Pi configurations.
You need a fair amount of knowledge of Linux and Android to get this to work.
Be warned.

My build system:
 - Ubuntu 16.04

My Raspberry Pi configuration:
 - Raspberry Pi 3B or 3B+
 - 7" HDMI touch screen 1024x600. There are many available. The one I use
   is from Elecrow


## Preparation

Make sure that you have a system capable of building AOSP in a reasonable
amount of time, as described here:
[http://source.android.com/source/building.html](http://source.android.com/source/building.html).

Then set it up following the steps given here:
[http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html)

Then install these extra packages
```
$ sudo apt install bmap-tools
```


## Download AOSP

Choose a directory for the AOSP source, e.g. $HOME/aosp:
```
$ export ANDROID_BUILD_TOP=$HOME/aosp
$ mkdir $ANDROID_BUILD_TOP && cd $ANDROID_BUILD_TOP
```

Select the version of AOSP you want:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-10.0.0_r25
```

If you want to reduce the size of the download and you don't care too much
about the git history you can do a shallow clone by setting the depth to 1:
```
$ repo init --depth=1 -u https://android.googlesource.com/platform/manifest -b android-10.0.0_r25
```

## Get the Raspberry Pi code and sync

```
$ git clone https://github.com/csimmonds/a4rpi-local-manifest .repo/local_manifests -b android10
$ repo sync -c
```


## Install a Linaro toolchain
You need this to build U-Boot and the kenel

Download
[https://releases.linaro.org/components/toolchain/binaries/7.4-2019.02/arm-linux-gnueabihf/
gcc-linaro-7.4.1-2019.02-x86_64_arm-linux-gnueabihf.tar.xz](https://releases.linaro.org/components/toolchain/binaries/7.4-2019.02/arm-linux-gnueabihf/gcc-linaro-7.4.1-2019.02-x86_64_arm-linux-gnueabihf.tar.xz)

Un-tar it somewhere convenient, for example your $HOME directory

## Build U-Boot

```
$ cd $ANDROID_BUILD_TOP/u-boot
$ PATH=$HOME/gcc-linaro-7.4.1-2019.02-x86_64_arm-linux-gnueabihf/bin:$PATH
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabihf-
$ cp $ANDROID_BUILD_TOP/device/rpiorg/rpi3/u-boot/rpi_3_32b_android_defconfig configs
$ make rpi_3_32b_android_defconfig
$ make
```

## Build the kernel
```
$ PATH=$HOME/gcc-linaro-7.4.1-2019.02-x86_64_arm-linux-gnueabihf/bin:$PATH
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabihf-
$ cd $ANDROID_BUILD_TOP/kernel/rpi
$ scripts/kconfig/merge_config.sh arch/arm/configs/bcm2709_defconfig \
kernel/configs/android-base.config kernel/configs/android-recommended.config
$ make -j $(nproc) zImage
$ cp arch/arm/boot/zImage $ANDROID_BUILD_TOP/device/rpiorg/rpi3
$ make dtbs
$ croot
```

## Build AOSP

Start a *new shell*

```
$ source build/envsetup.sh
$ lunch aosp_rpi3-eng
$ m
```
The command 'm' is a wrapper for 'make', with the additional benefit that it
will use all available CPU cores (see \filePath{build/soong/ui/build/config.go).

Even so, the build will take an hour or two...


## Write to SD card

You will need a micro SD card of at least 4 GB. Ideally it should
be class 10 or better.

Plug the card into your card reader. Run command `lsblk` to find which
device it is. Then run the script below, giving the device name as the
parameter. For example, if the card reader is `/dev/mmcblk0`, the
command would be:
```
$ scripts/write-sdcard-rpi3.sh mmcblk0
```
Note: I use bmap-tool to write the card because it is faster and
safer than dd. If you don't want to use bmap-tool, go ahead and edit the script
to switch back to dd.

When done, plug the card into your Raspberry Pi and boot up


## ADB

Because the Raspberry Pis up to and including 3 do not have a USB OTG port, the
only way to connect ADB is via Ethernet. Plug your Raspberry Pi into your Ethernet
LAN and wait for it to be given an IP address. Then from $ANDROID_BUILD_TOP type
this command:

```
$ adb connect Android.local:5555
```


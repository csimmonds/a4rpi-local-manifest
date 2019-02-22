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
 - Elecrow 7" HDMI touch screen


## Preparation

Make sure that you have a system capable of building AOSP in a reasonable
amount of time, as described here:
[http://source.android.com/source/building.html](http://source.android.com/source/building.html).

Then set it up following the steps given here:
[http://source.android.com/source/initializing.html](http://source.android.com/source/initializing.html)


## Download AOSP

```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-9.0.0_r30
```

If you want to reduce the size of the download and you don't care too much
about the git history you can do a shallow clone by setting the depth to 1:
```
$ repo init --depth=1 -u https://android.googlesource.com/platform/manifest -b android-9.0.0_r30
```
## Get the Raspberry Pi code and sync

```
$ git clone https://github.com/csimmonds/a4rpi-local-manifest .repo/local_manifests -b pie
$ repo sync -c
```
Note: if you are using Ubuntu 16.04 you may see errors like this
```
RPC failed; curl 56 GnuTLS recv error (-9)
```
You can try Googling various fixes, but in my experience the best solution is
to upgrade to Ubuntu 18.04.

## Build AOSP

```
$ source build/envsetup.sh
$ lunch aosp_rpi3-eng
$ m
```
The command 'm' is a wrapper for 'make', with the additional benefit that it
will use all available CPU cores (see \filePath{build/soong/ui/build/config.go).

Even so, the build will take an hour or two...


## Install Linaro toolchain
Go to
[https://releases.linaro.org/components/toolchain/binaries/4.9-2016.02/arm-linux-gnueabihf](https://releases.linaro.org/components/toolchain/binaries/4.9-2016.02/arm-linux-gnueabihf)
and get gcc-linaro-4.9-2016.02.tar.xz

Un-tar it somewhere convenient

## Build the kernel
```
$ PATH=/path to toolchain/bin:$PATH
$ export ARCH=arm
$ export CROSS_COMPILE=arm-linux-gnueabihf-
$ cd kernel/brcm/rpi3/
$ make lineageos_rpi3_defconfig
$ make -j $(nproc) zImage
$ make dtbs
$ croot
```

## Write to SD card

You will need a micro SD card of at least 4 GB. Ideally it should
be class 10 or better.

Plug the card into your card reader. Run command `lsblk` to find which
device it is. Then run the script below, giving the device name as the
parameter. For example, if the card reader is `/dev/mmcblk0`, the
command would be:
```
$ scripts/write-sdcard-beagleboneblack.sh mmcblk0
```
Note: I use bmap-tool to write the card because it is faster and
safer than dd. If you don't want to use bmap-tool, go ahead and edit the script
to switch back to dd.

When done, plug the card into your Raspberry Pi and boot up



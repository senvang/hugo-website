---
title: Banana Pi BPI-F3 
date: 2025-01-10T00:20:00+07:00
author: sroemer
categories:
- it
tags:
- embedded
- riscv
- bananapi
- bpif3
- bpi-f3
- spacemit
- linux
- gentoo
- fastboot
- uboot
- u-boot
keywords:
- embedded
- riscv
- bananapi
- bpif3
- bpi-f3
- spacemit
- linux
- gentoo
- fastboot
- uboot
- u-boot
showFullContent: true
readingTime: false
hideComments: false
toc: true
---

I recently bought an [Banana Pi BPI-F3](https://docs.banana-pi.org/en/BPI-F3/BananaPi_BPI-F3).
The BPI-F3 is a development board based on an
[SpacemiT K1](https://docs.banana-pi.org/en/BPI-F3/SpacemiT_K1_datasheet) 8-core RISC-V chip.

Here I will write up how I get it up and running. My plan is to build a custom Gentoo linux
image for it - but that's a long way to go. At first I will connect everything and use an
existing image.

Requirements:
- BPI-F3
- 12V Power Supply
- USB to serial adapter (3.3V TTL)
- USB-C to USB-A cable for connecting the BPI-F3 to the host

Contents:
{{% toc %}}

#  1 - Serial console {#serial-console}

## 1.1 Connecting a USB to serial adapter

Connect GND, RX and TX of the `UART0 Debug` connector to an USB serial adapter (3.3V TTL).
Once connected a terminal can be used to connect with the following settings:

```
baud rate: 115200
data bits: 8
parity   : none
stop bits: 1
```

## 1.2 Output with empty eMMC and no SD card inserted

In the default configuration the BPI-F3 tries to boot from SD card first and falls back to booting
from eMMC. However, on a new BPI-F3 the eMMC doesn't contain any data and therefore without an SD card the
output looks as shown below:
```
␀sys: 0x0
try sd...
bm:3
ERROR:   CMD8
ERROR:   sd f! l:76
bm:0
ERROR:   emmc: invalid bootinfo magic code:0x0
ERROR:   invalid bootinfo image.
ERROR:   entering download mode.
ROM: usb download handler
usb2d_initialize : enter
Controller Run
```

So in the next step we need to prepare an SD card and/or write an image to the eMMC.

# 2 - Installing an image {#install-image}

I initially planned to start with [Bianbu Linux](http://archive.spacemit.com/image/k1/version/bianbu/)
installed on an SD card, but for some reason this was not booting. The image was written correctly to the SD
card and I could not figure out the problem yet. A [Gentoo Linux image](https://dev.gentoo.org/~lu_zero/riscv/)
however booted scuccessfully after sorting out one old SD card which corrupted the image on my first attempt.
I therefore went on and installed Bianbu linux on eMMC.

## 2.1 - Boot selection settings {#boot-select}

For booting from eMMC the DIP switches must be set correctly. In the factory default settings all 4 switches
are set to `off`. Switch 1 and 2 are used for selecting the boot device and the factory default already
selects eMMC as we want it.

Here some details about the DIP switch functions (see BPI-F3 schematic from the BananaPi Docs):

```
# BPI-F3 SW1 functions

-------------
|   off   on |
| 1 -o------ | - boot select 1 (QSPI_DATA0)
| 2 -o------ | - boot select 2 (QSPI_DATA1)
| 3 -o------ | - not connected
| 4 -o------ | - not connected
-------------
     ^---------- factory defaults (all 4 switches set to off)

Boot select:
     1  |  2  | function
    ====|=====|====================
    off | off | TF card -> eMMC (factory default)
    off | on  | TF card -> SPI NAND
    on  | off | TF card -> SPI NOR
    on  | on  | TF card only
```

## 2.2 - Enter download mode {#download-mode}

To enter `download` mode on the BPI-F3 hold down the `FDL` button and then reset the device.
Under some circumstances the device also enters `download` mode automatically. This for example
is the case on a failed boot attempt with an empty eMMC.

Once the device is in `download` mode it shows following messages on the serial debug console:
```
ROM: usb download handler
usb2d_initialize : enter
Controller Run
```

After connecting the `PD12V` USB port of the BPI-F3 to the host a `fastboot devices` command
should show the device as shown below:
```
$ fastboot devices
dfu-device	 DFU download
```

Now we can continue to install the image with `fastboot`.

## 2.3 - Enter U-Boot fastboot mode {#fastboot-mode}

At the current state we only have a simple download mode provided by the boot ROM running.
Now we need to upload the `FSBL` (First Stage Boot Loader) which in case of the BPI-F3 is the
`U-Boot SPL` stage (U-Boot Secondary Program Loader - second after the code from the boot ROM).
After this we also can upload the main `U-Boot` binary (a.k.a. `U-Boot proper`) which then provides
the `fastboot` mode we use for flashing the firmware.

The [Boot Development Guide](https://bianbu-linux.spacemit.com/en/device/boot/) in the
[Bianbu Linux Docs](https://bianbu-linux.spacemit.com/en/) describes this and provides
additional information about the boot process of the `SpacemiT K1`.

For entering the `U-Boot` `fastboot` mode run the following commands:
```
fastboot stage factory/FSBL.bin
fastboot continue

# wait some time to ensure u-boot is ready
sleep 1

fastboot stage u-boot.itb
fastboot continue
```

The required files can be found in the `zip` files available for [download](http://archive.spacemit.com/image/k1/version/bianbu/)
on the `SpacemiT` server. Downloads ending in `img.zip` contain an image to be written onto an SD card. On the other hand
downloads ending in only `zip` contain separate files we can use to upload the firmware via fastboot - that's what we need.  

-> Here the file for [Bianbu Linux v2.0.4](http://archive.spacemit.com/image/k1/version/bianbu/v2.0.4/bianbu-24.04-minimal-k1-v2.0.4-release-20241205234138.zip) (minimal variant for K1)

## 2.4 - Flash eMMC {#flash-emmc}

Once the transition to the `U-Boot` `fastboot` mode is performed we can create the required partitions and
write the firmware to the eMMC. All required files are included in the `zip` file we already downloaded.

For creating the partitions and writing the firmware run following commands: 
```
fastboot flash gpt partition_universal.json
fastboot flash bootinfo factory/bootinfo_emmc.bin
fastboot flash fsbl factory/FSBL.bin
fastboot flash env env.bin
fastboot flash opensbi fw_dynamic.itb
fastboot flash uboot u-boot.itb
fastboot flash bootfs bootfs.ext4
fastboot flash rootfs rootfs.ext4
```

Now a reset should result in the `BPI-F3` booting up the installed `Linux`.
The boot process can be watched on the serial debug console.

# 3 - What to do next?

- Play around with the device
- Understand the boot process etc.
- Investigate why the SD card images of Bianbu Linux did not boot
- Custom builds of OpenSBI, U-Boot and Linux
- and maybe more ...

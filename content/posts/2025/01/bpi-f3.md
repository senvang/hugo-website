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
showFullContent: true
readingTime: false
hideComments: false
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
- USB-C to USB-A/C cable for connecting the BPI-F3 to host

#  1 - Serial console

Connect GND, RX and TX of the `UART0 Debug` connector to an USB serial adapter (3.3V TTL).
Once connected a terminal can be used to connect with the settings `115200-8-N-1`.

In the default configuration the BPI-F3 tries to boot from SD card first and falls back to booting
from eMMC. However, on a new BPI-F3 the eMMC doesn't contain any data and so without an SD card the
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


# 2 - Installing an image

I initially planned to start with [Bianbu Linux](http://archive.spacemit.com/image/k1/version/bianbu/v2.0.4/)
installed on an SD card, but for some reason this was not booting. The image was written correctly to the SD
card and I could not figure out the problem yet. A [Gentoo Linux image](https://dev.gentoo.org/~lu_zero/riscv/)
however booted scuccessfully after sorting out one old SD card which corrupted the image on my first attempt.
I therefore went on and installed Bianbu linux on eMMC.

## 2.1 Boot selection settings

For booting from eMMC the DIP switches must be set correctly. In the factory default settings all 4 switches
are set to `off`. Switch 1 and 2 are used for selecting the boot device and the factory default already
selects eMMC as we want it.

Here some details about the DIP switch functions (see BPI-F3 schematic from the BananaPi Docs):

```
# BPI-F3 SW1 functions

-------------
|   off   on |
| 1 -o------ | - boot select 1        (QSPI_DATA0)
| 2 -o------ | - boot select 2        (QSPI_DATA1)
| 3 -o------ | - download select      (QSPI_DATA2)
| 4 -o------ | - boot download select (QSPI_DATA3)
-------------
     ^---------- factory defaults (all 4 switches set to off)

Boot select:
     1  |  2  | function
    ====|=====|====================
    off | off | TF card -> eMMC (factory default)
    off | on  | TF card -> SPI NAND
    on  | off | TF card -> SPI NOR
    on  | on  | TF card only

Boot Download select:

     3  | function
    ====|=========
    off | normal boot (factory default)
    on  | download

Download select:

     4  | function
    ====|=========
    off | USB (factory default)
    on  | UART
```

## 2.2 Enter download mode

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

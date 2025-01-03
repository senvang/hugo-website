---
title: Banana Pi BPI-F3 
date: 2025-01-04T00:10:00+07:00
author: sroemer
categories:
- it
tags:
- embedded
- riscv
- bananapi
- bpif3
- spacemit
- linux
- gentoo
keywords:
- embedded
- riscv
- bananapi
- bpif3
- spacemit
- linux
- gentoo
showFullContent: true
readingTime: false
hideComments: false
---

I recently bought an [Banana Pi BPI-F3](https://docs.banana-pi.org/en/BPI-F3/BananaPi_BPI-F3).
The BPI-F3 is a development board based on an
[SpacemiT K1](https://docs.banana-pi.org/en/BPI-F3/SpacemiT_K1_datasheet) 8-core RISC-V chip.

Here I will write up how I get it up and running. My plan is to build a custom Gentoo linux
image for it - but that's a long way to go. For now I will connect everything and use an existing
image.

Requirements:
- BPI-F3
- 12V Power Supply
- USB to serial adapter (3.3V TTL)

## Step 1: Serial console

Connect GND, RX and TX of the `UART0 Debug` connector to an USB serial adapter (3.3V TTL).
Once connected a terminal can be used to connect with the settings `115200-8-N-1`.

In the default configuration the BPI-F3 tries to boot from SD card first and falls back to booting
from EMMC. However, on a new BPI-F3 the EMMC doesn't contain any data and so without an SD card the
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

So in the next step we need to prepare an SD card and/or write an image to the EMMC.

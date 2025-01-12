---
title: Building a custom firmware for the Banana Pi BPI-F3 
date: 2025-01-13T01:00:00+07:00
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
- opensbi
- uboot
- u-boot
- firmware
keywords:
- embedded
- riscv
- bananapi
- bpif3
- bpi-f3
- spacemit
- opensbi
- uboot
- u-boot
- firmware
showFullContent: true
readingTime: false
hideComments: false
toc: true
---

I started building the firmware components for booting the `BPI-F3` myself. While `SpacemiT` provides the source code
of the different components the official firmware is built from forks of older versions and without the
commit history. In my attempt to build a custom firmware I will use mainline versions where possible and
otherwise create my own forks from mainline sources and merge necessary changes into it.

Contents:
{{% toc %}}

# Status

| name     | build | test   |
|----------|:-----:|:------:|
| bootinfo | ✅    | ❌     |
| fsbl     | ❌    | ❌     |
| opensbi  | ✅    | ❌     |
| u-boot   | ✅    | ❌     |

# Bootinfo

The first part of the boot process we have control of is the `bootinfo` binary. In the `SpacemiT` `BSP`
[Bianbu Linux](https://gitee.com/bianbu-linux) the `bootinfo` is generated by a Python script during
the build of [u-boot](https://gitee.com/bianbu-linux/uboot-2022.10).

With [bpif3_gen_binary](https://github.com/sroemer/bpif3_gen_binary) I extracted and slightly modified this
script and according configuration files to be used independent from the `u-boot` source tree.

The content of the generated `bootinfo` is specified in a `JSON` file and could be modified.
However, currently I do not see a need for that.

# FSBL (U-Boot SPL)

The `FSBL` (First Stage Boot Loader) is generated by the same script as the `bootinfo` but requires `u-boot-spl.bin` at a path
specified in the according `JSON` configuration file.

While mainline `u-boot` already has support for the `BPI-F3`, it only generates a `u-boot proper` stage
but not a `u-boot spl` stage.

# OpenSBI

Mainline [OpenSBI](https://github.com/riscv-software-src/opensbi) does not have support for the `SpacemiT K1` (yet).
Luckily [cyyself](https://github.com/cyyself) already merged support for the `SoC` into a clean fork of `OpenSBI`.

With [bpif3_opensbi](https://github.com/sroemer/bpif3_opensbi) I merged [his work](https://github.com/cyyself/opensbi/tree/k1-opensbi)
on to the latest version of mainline `OpenSBI`.

# U-Boot

Mainline [u-boot](https://github.com/u-boot/u-boot) has support for the `BPI-F3`. It lacks support for the `u-boot spl`
stage but builds the `u-boot proper` main stage.
---
title: Reverse engineering a BLE body fat scale protocol
date: 2024-09-14T22:19:00+07:00
author: sroemer
categories:
- it
tags:
- bluetooth
- ble
- gatt
- bleak
- scale
- python
keywords:
- bluetooth
- ble
- gatt
- bleak
- scale
- python
showFullContent: true
readingTime: false
hideComments: false
---

I recently bought the '*smart*' body fat scale *Crénot Gofit S2*. As many of those modern '*smart*' devices it uses
`Bluetooth Low Energy` (`BLE`) and an App on the phone to connect to it. Due to privacy concerns I do not want to use this
cloud based App and share my fitness / health data.

Instead I found [openScale](https://github.com/oliexdev/openScale). An open source App which would give me the same
functionality while storing all data locally on my phone. Sadly it did not support my scale so I started investigating
if I can add support for it.

Initially I read the [How to reverse engineer a Bluetooth 4.x scale](https://github.com/oliexdev/openScale/wiki/How-to-reverse-engineer-a-Bluetooth-4.x-scale)
notes on the openScale Github page and also created a Bluetooth HCI snoop log as described there, but then ended up
following a different approach and wrote a small client in Python by using [Bleak](https://pypi.org/project/bleak/).

BLE devices use the `Generic Attribute Profile` (`GATT`) and provide services with different characteristics.
The [BLE scanner](https://play.google.com/store/apps/details?id=com.macdom.ble.blescanner) App mentioned in the notes
became handy to gather information about the scales provided services and characteristics.

With this information I quickly was able to reliably get the weight value from the scale but did not figure out yet how other
values - like body fat percentage for example -  are transferred.

My client can be found in my [scale-communication](https://github.com/sroemer/scale-communication) repo on Github and
with knowing the basic communication protocol it also will be possible to implement this in openScale. I also did not
give up yet on getting the missing values from my scale - but that's for another time. 

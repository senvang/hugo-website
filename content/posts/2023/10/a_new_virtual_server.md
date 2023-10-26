---
title: I ordered a new virtual server
date: 2023-10-14T00:00:00+07:00
author: sroemer
categories:
- it
tags:
- server
- linux
- devuan
keywords:
- server
- linux
- devuan
showFullContent: true
readingTime: false
hideComments: false
---

After running my own server for some years in the past I did move on to using an [Uberspace](https://uberspace.de/).
Uberspace as a hoster is great and offers a lot of flexibility but it has its limitations anyway (for example it is not
possible to run a VPN server). I am planning to run a VPN server and also like the admin work overall. So I ordered a
small virtual server (VPS 200 G10s) from [Netcup](https://www.netcup.de/) again.

#### Why did I choose Netcup?

The answer to that is easy. It's cheap and I did have some good experience with it already.
For 3.25 Euro/month I get a VPS with 2vCPUs, 2GB RAM and 40GB SSD. For the virtualization they use KVM and via the
web interface I can mount any ISO file and install whatever OS I like. The drawback of their cheap virtual servers
is the quite low minimum availability of just 99.6% but for my use case that's not an issue.

#### Which OS did I install?

After initially even considering [Gentoo](https://www.gentoo.org/) Linux I quickly dropped that idea due to the
limitations of the virtual hardware. I then - after some research - did a kind of test installation of Alpine Linux.
It was the first time ever I used [Alpine](https://www.alpinelinux.org/) and I must say that I am pretty impressed
with it - especially due to its low footprint. Anyway I decided to stick with something I know better and trust a bit
more in the long term.

I installed [Devuan](https://www.devuan.org) Linux - Devuan Daedalus 5.0.
For those who are not aware, Devuan is a fork of [Debian](https://www.debian.org) Linux which does not use systemd and
therfore I can count on the great selection of software available for Debian while still using the good old sysvinit.

As always I did a minimal installation and add what is required for my needs from there. My base installation with
[Nginx](https://nginx.org/), [Certbot](https://certbot.eff.org/) and unattended upgrades uses about 60MB of RAM.
A nice small footprint as well even if its not as small as Alpine Linux (but that's expected).

#### Further plans?

With Nginx and Certbot up and running I already moved the website on to the new server.
Additionally I want to install the following components:

- [Gitea](https://gitea.com/) git server, bug tracker, etc.
- [Radicale](https://radicale.org/) CalDAV and CardDAV server
- Mail server (probably [Postfix](https://www.postfix.org/) and [Dovecot](https://www.dovecot.org/))
- [WireGuard](https://www.wireguard.com/) VPN server
- ... and eventually more

Regarding all this I am not in a hurry. For now I am using the Uberspace and my virtual server in parallel and before
installing any of those components I want to do some research about [Linux Containers](https://linuxcontainers.org/).
So far I do not have experience with containers of any kind and I am not a huge fan of Docker and its approach of
isolating single applications. Linux Containers on the other hand are available on any Linux system, are OS-level
virtualization and do not rely on infrastructure controlled by a single company.
So my idea is to isolate some parts of my installation into Linux Containers (probably using Devuan images as well).
Therefore I should gain some flexibility when updating the main server OS and be able to move and update some parts
independently.

In any case containers are an interesting topic and digging deeper into it will be worth it ...

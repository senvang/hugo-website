---
title: A quick update
date: 2024-09-26T23:05:00+07:00
author: sroemer
categories:
- it
tags:
- server
- linux
- gentoo
- postfix
- dovecot
keywords:
- server
- linux
- gentoo
- postfix
- dovecot
showFullContent: true
readingTime: false
hideComments: false
---

It's a long time since I wrote something here so in this time things happened in the background.
Here a quick update:

- I moved the server to a more powerful one (VPS 1000) - still from [Netcup](https://www.netcup.de/).
The main reason for this upgrade was my decision to switch to [Gentoo Linux](https://www.gentoo.org/)
on the server as well after my desktop computer is running flawlessly with it since years now and I
wanted to make this switch before I started setting up a mail server.

- I got following services running on Gentoo now:
  1. Certbot to get certificates from [Let's Encrypt](https://letsencrypt.org/)
  2. [Nginx](https://nginx.org/) as web server / proxy
  3. [Radicale](https://radicale.org/) as CalDAV / CardDAV server
  4. Git via ssh access (I keep it simple and do not run [Gitea](https:/gitea.com) anymore)
  5. [Postfix](http://www.postfix.org/) / [Dovecot](https://www.dovecot.org/) as mail server
  6. [Wireguard](https://www.wireguard.com/) as VPN server

- The mail server is setup with [SPF](https://en.wikipedia.org/wiki/Sender_Policy_Framework),
[DKIM](https://en.wikipedia.org/wiki/DomainKeys_Identified_Mail), [DMARC](https://en.wikipedia.org/wiki/DMARC)
and [MTA-STS](https://en.wikipedia.org/wiki/Simple_Mail_Transfer_Protocol#SMTP_MTA_Strict_Transport_Security).
Regarding [DANE](https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities) I am still investigating
if I can/want to set this up with the frequently changing Let's Encrypt certificates.

So what the heck is all that? Here in short:
- SPF: Specifies the mail servers allowed to send mail for the domain by publishing an entry in the DNS
- DKIM: Sign outgoing mail and publish the public key in the DNS, so the receipient can verify the signature
- DMARC: Publish a DMARC DNS entry with instructions on how to handle mail which doesn't pass SPF/DKIM checks
- MTA-STS: A combination of DNS and HTTPS to instruct SMTP servers to encrypt communication and verify the used certificate
- DANE: A DNS entry containing the server certificate (usually just a hash of it) and therefore allowing the communication
partner to compare the servers certificate against the one published in DNS

That's what happened on here in the last months. More to come ...

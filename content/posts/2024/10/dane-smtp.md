---
title: DANE-SMTP enabled
date: 2024-10-01T14:27:00+07:00
author: sroemer
categories:
- it
tags:
- mail
- server
- dane
- dns
- tlsa
- certbot
- letsencrypt
- dnssec
keywords:
- mail
- server
- dane
- dns
- TLSA
- certbot
- letsencrypt
- dnssec
showFullContent: true
readingTime: false
hideComments: false
---

After setting up my mail server I already set up `SPF`, `DKIM`, `DMARC` and `MTA-STS` but was not yet sure about how to
deal with [DANE](https://en.wikipedia.org/wiki/DNS-based_Authentication_of_Named_Entities).

`DANE` uses a `TLSA` entry on the `DNS` server to publish a services certificate or key. This means that a client can
verify that it is talking to the correct server without relying on any kind of CA. All this is based on
[DNSSEC](https://en.wikipedia.org/wiki/Domain_Name_System_Security_Extensions) which ensures the authenticity and data
integrity of the `DNS` entries.

While this concept in general is pretty neat, it has the requirement of `DNS` entries to be updated when a certificate or
key changes. Additionally just simply updating an entry will no be enough, since other `DNS` servers still might have the
old key cached. Therefore a more sophisticated rollover mechanism would be required. Due to the short life of
[Let's Encrypt](https://letsencrypt.org/) certificates this is even more relevant.

Luckily [Certbot](https://certbot.eff.org/) provides the option `--reuse-key` to circumvent the requirement of updating
the `DNS` entries by reusing the existing key. Therefore a `TLSA` entry can be generated from the current public key and
will not require updating as well since the key does not change.

For generation of an according `TLSA` entry [Shumon Huque](https://www.huque.com/) has a very handy
[TLSA Record Generator](https://www.huque.com/bin/gen_tlsa) on his website.
The entry in this case must be for usage `DANE-EE` with selector `1 - SPKI`. I use a `SHA-256` hash and got following
result:
```
_25._tcp.vs.senvang.org. IN TLSA 3 1 1 (
                  5634d5e1ce5f1e4a8ab25cd8335c97ab76e1215e09c157568e5c9a3dc39a
                  a491
)
```

Alternatively you can generate the hash with `openssl`:
```
openssl x509 -in cert.pem -pubkey -noout | openssl ec -pubin --outform der | sha256sum
```

Obviously this does not resolve the need for a clean rollover when a key change happens, but it massively reduces the
frequency and therefore the risk and it gives me time to think about how to get this automated.

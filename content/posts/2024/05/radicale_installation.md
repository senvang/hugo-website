---
title: Radicale installation on Devuan Daedalus 5.0
date: 2024-05-30T23:31:00+07:00
author: sroemer
categories:
- it
tags:
- server
- linux
- devuan
- caldav
- carddav
- radicale
keywords:
- server
- linux
- devuan
- caldav
- carddav
- radicale
showFullContent: true
readingTime: false
hideComments: false
---


Since years I am running my own CalDAV / CardDAV server to synchronize my calendars and address books.
I use [Radicale](https://radicale.org) for this and on my Uberspace it was installed via Python's `pip`,
but now it was time to move this to my own server too and I chose to install the package directly from
the Devuan repository via `apt-get install radicale`.

The installation via `apt-get` just does the installation but some additional configuration was needed
to set everything up.

At first I modified the configuration file `/etc/radicale/config` and did set  
`type = htpasswd` and `htpasswd_encryption = bcrypt` (both in the `[auth]` section).

Then I created a new htpasswd file with my user:  
`htpasswd -c -B /etc/radicale/users <user>`

Finally I started the service and enabled it to be started automatically:  
`/etc/init.d/radicale start`  
`update-rc.d radicale enable`

After this the radicale server is bound to localhost and listening on port 5232.
For being accessible from the outside the following configuration must be added to the Nginx configuration (and Nginx being restarted once):
```
location /radicale/ { # The trailing / is important!
      proxy_pass        http://localhost:5232/; # The / is important!
      proxy_set_header  X-Script-Name /radicale;
      proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header  Host $http_host;
      proxy_pass_header Authorization;
}
```

Now I only had to copy contact and calendar entries from my Uberspace instance. Done!

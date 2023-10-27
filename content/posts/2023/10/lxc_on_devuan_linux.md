---
title: Linux Containers (LXC) on Devuan Daedalus 5.0
date: 2023-10-25T00:00:00+07:00
author: sroemer
categories:
- it
tags:
- server
- linux
- devuan
- lxc
keywords:
- server
- linux
- devuan
- lxc
- container
showFullContent: true
readingTime: false
hideComments: false
---

I started some investigation about how to run Linux Containers on Devuan Linux 5.0
(Daedalus), as I run it on my server. For this I first did set up a local VM which
replicates my virtual server. I use qemu for this and created a script called
[vm&#x2011;netcup](https://github.com/sroemer/dotfiles/blob/main/.local/bin/vm-netcup)
which is part of my [dotfiles](https://github.com/sroemer/dotfiles/) repository and
sets qemu parameters accordingly.

Installation via `apt-get install lxc` works as expected. So far so good - but things
should not go that smoothly for much longer.

#### cgroups

LXC relies on cgroups, a Linux kernel feature for isolation, limitation and accounting of
resource usage of processes. On distributions using systemd this is set up by systemd, but
with Devuan I chose to not use systemd.

You guessed it: `lxc-checkconfig` and `ls /sys/fs/cgroup` show that cgroups are not set up
by default (at least not with my minimal installation), but some research and serveral cups
of coffee later a solution appeared: run `apt-get install cgroupfs-mount` and this problem
is solved.

#### unpriviledged vs. priviledged containers

In short: We want unpriviledged containers whenever possible. Those map user ids inside the
container to a different range on the host and therefore are the safest option. For example
user id 0 (root) in an unpriviledged container would be mapped to user id 100000 on the host.
A user id 0 (root) on a priviledged container on the other hand also would be root on the
host.

`/etc/subuid` and `/etc/subgid` contain the ranges for mapping uid and gid on the host,
but those values also need to be reflected in the lxc configuration as described below.

#### configuration

For configuration LXC distinguishes between system and container configuration. See the
according man pages lxc.container.conf and lxc.system.conf for further information.

By default LXC on my system used `~/.local/share/lxc/` for container storage. This resulted
in LXC complaining about the users home directory not having x permission for the root user
of the container and starting the container failed.

Therefore I decided to move the container storage to `/var/lib/lxc/<user>`. This
directory needs to be created as root and owernship as well as permissions need to be set
as follows:  
`chown <user>:<group> /var/lib/lxc/<user>`  
`chmod 711 /var/lib/lxc/<user>`

Once this is done we can continue as regular user. We create a file `.config/lxc/lxc.conf`
and configure the container storage path in it:  
`lxc.lxcpath = /var/lib/lxc/<user>`

`/etc/lxc/default.conf` contains the system wide container default configuration. We copy
this file to `~/.config/lxc/default.conf` and modify it to create our user specific defaults.

Be aware that this file only contains the default configuration used during creation of a
new container. Once created every container has its own configuration file in its according
folder within the container storage path.

We now add the following lines for configuration of the uid and gid mapping. The values used
here must match the values defined in `/etc/subuid` and `/etc/subgid` for the according user:   
`lxc.idmap = u 0 100000 65536`  
`lxc.idmap = g 0 100000 65536`

Additionaly we change the AppArmor profile as follows:  
`lxc.apparmor.profile = unconfined`

##### creating a new container

For creation of a new container we use the `lxc-create` command. `lxc-create -t download -n <container-name>`
is a good start and provides an interactive way to create a new container. In my case I want to
install the `3.18` release for the `amd64` architecture of the `alpine` distribution.

This all also can be packed into the command directly:  
`lxc-create -t download -n <container-name> -- --dist alpine --release 3.18 --arch amd64`

##### network

With all this set up the container still fails to start because it cannot set up its network.
By default unprivileged containers cannot use any networking. We need to create an additional
configuration file `/etc/lxc/lxc-usernet` as root. This file must have the following entry
to allow the user to add 1 veth device to the lxcbr0 bridge:  
`<user> veth lxcbr0 1`

##### using the container

To start an already existing container the command `lxc-start -n <container-name>` is used.
With everything I described above done the container does start without issues. Previously
appearing `tmpfs: Bad value for 'uid'` error message were related to the Devuan container
image used. The errors disappeared after I switched to using the Alpine Linux image.

Once the container is running we can start a process in the container with `lxc-attach`.
For example `lxc-attach -n <container-name> bash` will provide shell access to the container.

For stopping the container again we use `lxc-stop -n <container-name>`.

In case the container isn't running we directly can run a command in it with `lxc-execute`.
For example `lxc-execute -n <container-name> bash` will provide shell access as before.

#### conclusion

I am not done with this topic yet but for now this is a good foundation and a playground to
further investigate if I want to run containers on the server. I might update this post at
some point if I have any points worth mentioning.

Especially the network setup was barely touched here and could be a topic for a separate post
at some point. For now that's it ...

---
title:		'Compiling Docker from source on Kali Linux'
author:		Stefan
createdate:	2017-01-23
modifydate:	2017-07-11
categories:	[Linux, Container]
tags:		[Docker, Kali, Linux, compiler]
slug:		compiling-docker-from-source-on-kali-linux
status:		draft
lang:		en
docid:		fe2b34b8d8fbc0d5
---

# Compiling Docker from source on Kali Linux

I noticed that there are no Docker packages for Kali Linux in the official Kali
Linux repository. So I prepared my Kali Linux machine to compile Docker from
source.

## Prerequisites

- Kali Linux (either native or as a virtual machine)
- git, Go and some libraries

Install the missing packages.

```
apt-get update && apt-get install golang btrfs-progs libdevmapper-dev libseccomp-dev
```

## Cloning the repositories

Add a new system user to compile the sources.

```
root@kali:~# useradd -m build
root@kali:~# su - build
build@kali:~$ 
```

The build user clones the repositories and compiles the sources.

```
git clone https://github.com/moby/moby
cd moby
git checkout v17.05.0-ce
AUTO_GOPATH=1 ./hack/make.sh dynbinary
```

You will see this warning because we're not running Docker.

```
# WARNING! I don't seem to be running in a Docker container.
# The result of this command might be an incorrect build, and will not be
# officially supported.
```

Build containerd.

```
mkdir -p ~/src/github.com/docker
cd ~/src/github.com/docker
git clone https://github.com/docker/containerd
cd containerd
git checkout v0.2.8
GOPATH=~ make
```

Build runc.

```
mkdir -p ~/src/github.com/opencontainers
cd ~/src/github.com/opencontainers
git clone https://github.com/opencontainers/runc
cd runc
git checkout v1.0.0-rc2
GOPATH=~ make
```

## Installation

As root install the binaries in `/usr/local/bin`.

```
cp -ai \
/home/build/src/github.com/docker/containerd/bin/containerd \
/home/build/src/github.com/docker/containerd/bin/containerd-shim \
/home/build/src/github.com/docker/containerd/bin/ctr \
/home/build/moby/bundles/17.05.0-ce/dynbinary-client/docker-17.05.0-ce \
/home/build/moby/bundles/17.05.0-ce/dynbinary-daemon/dockerd-17.05.0-ce \
/home/build/src/github.com/opencontainers/runc/runc \
/usr/local/bin/

cd /usr/local/bin
ln -s docker-17.05.0-ce docker
ln -s dockerd-17.05.0-ce dockerd
ln -s containerd docker-containerd
ln -s containerd-shim docker-containerd-shim
ln -s ctr docker-containerd-ctr
ln -s runc docker-runc
chown root.root *
```

Create a `docker` group.

```
groupadd docker
```

## Running Docker

Start the Docker daemon in screen and check the output.

```
screen -S dockerd dockerd
```

Detach from screen and pull a Docker image.

```
docker pull httpd:latest
```

We just downloaded the newest Apache httpd. Now you can run the image.

```
docker run -d --name webserver httpd
```

Find out the container IP address and connect to the webserver.

```
root@kali:~# docker exec webserver ip -4 a | grep inet
    inet 127.0.0.1/8 scope host lo
    inet 172.17.0.2/16 scope global eth0
root@kali:~# curl 172.17.0.2
<html><body><h1>It works!</h1></body></html>
```

We have an Apache httpd running on IP address 172.17.0.2.

---
title:		'Compiling Docker from source on Kali Linux'
author:		Stefan
createdate:	2017-01-23
modifydate:	2017-01-23
categories:	[Docker, Kali]
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

Install the missing packages

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
git clone https://github.com/docker/docker.git
cd docker
git checkout 1.13.x
AUTO_GOPATH=1 ./hack/make.sh dynbinary
```

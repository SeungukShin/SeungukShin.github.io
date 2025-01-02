---
title: Build Linux Kernel
parent: Linux Kernel
---

# Build Linux Kernel
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

This is tested on Ubuntu 18.04.1 LTS

## Build Kernel

Install dependencies
```sh
$ sudo apt-get install -y fakeroot build-essential crash \
    kexec-tools makedumpfile kernel-wedge flex bison git-core \
	libncurses5 libncurses5-dev libelf-dev asciidoc \
	binutils-dev libssh-dev
```

Clone linux kernel souce code
```sh
$ git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
$ cd linux
```

Copy configuration in previous version
```sh
$ mkdir -p ../out
$ cp /boot/config-<version> ../out/.config
$ make O=../out olddefconfig
```

Build linux kernel
```sh
$ make O=../out -j8
```

Build a package
```sh
$ make O=../out -j8 bindeb-pkg
```

## Build Module in a Separated Directory

Create a `Makefile`

```makefile
obj-m	+= test.o

all:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
	make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

Build a module
```sh
$ make -j8
```

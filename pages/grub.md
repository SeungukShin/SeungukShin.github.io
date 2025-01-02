---
title: Grub
---

# Grub
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Modify Grub Resolution

Check supported resolution
```sh
# reboot
# enter 'c' in grub screen
$ videoinfo
0x000  800x600x32
0x001  1024x768x32
0x002  1280x1024x32
0x003  1920x1080x32
```

Modify `/etc/default/grub`
```conf
GRUB_GFXMODE=1920x1080
```

Update the grub configuration
```sh
$ sudo update-grub
```

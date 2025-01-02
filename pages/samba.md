---
title: Samba
---

# Samba
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Install

Install samba package
```sh
$ sudo apt install samba
```

Edit `/etc/samba/smb.conf` to add a shared directory
```conf
[share]
    comment = share
    path = /home/share
    guest ok = no
    browseable = no
    create mask = 0600
    directory mask = 0700
```

Add a samba user
```sh
$ sudo smbpasswd -a <username>
```

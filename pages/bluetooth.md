---
title: Bluetooth
---

# Bluetooth
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Bluetooth on Display Manager

The bluetooth adapter does no power on after a reboot. Modify `/etc/bluetooth/main.conf` to power on after a reboot
```conf
[Policy]
AutoEnable=true
```

## References

* [Arch Wiki](https://wiki.archlinux.org/index.php/Bluetooth#Auto_power-on_after_boot)

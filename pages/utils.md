---
title: Utils
---

# Utils
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Remap Keys

Modify `/etc/udev/hwdb.d/*.hwdb`

Update hwdb
```sh
$ sudo systemd-hwdb update
```

Apply the modified hwdb
```sh
$ sudo udevadm trigger
```

## Dd

### Display progress

Use `pv`
```sh
$ dd if=<input> | pv | dd of=<output>
```

Use `kill`
```sh
$ dd if=<input> of=<output> & pid=$!
$ watch -n 10 kill -USR1 $pid
```

Use status option (GNU Coreutils 8.24+)
```sh
$ dd if=<input> of=<output> status=progress
```

## Gdm

### Scaling Setting

Change scaling setting on `/usr/share/glib-2.0/schemas/org.gnome.desktop.interface.gschema.xml`
```xml
<key name="scaling-factor" type="u">
<default>1</default>
```

Compile the setting
```sh
$ sudo glib-compile-schemas /usr/share/glib-2.0/schemas
```

## Certificate

### Install Certificate

Convert a certificate to pem format
```sh
$ openssl x509 -inform <input format> -in <input file> -outform pem -out file.pem
```

Copy the certificate to `/usr/local/share/ca-certificates/`
```sh
$ sudo cp file.pem /usr/local/share/ca-certificates/file.crt
```

Apply the certificate to system
```sh
$ sudo update-ca-certificates
```

The certificate file is symbolic linked to `/etc/ssl/certs/`

## References

* [Ask Ubuntu - dd](https://askubuntu.com/questions/215505/how-do-you-monitor-the-progress-of-dd)
* [Ask Ubuntu - certificate](https://askubuntu.com/questions/73287/how-do-i-install-a-root-certificate)

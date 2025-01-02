---
title: Debug Linux Kernel with GDB
parent: Linux Kernel
---

# Debug Linux Kernel with GDB
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Execute Qemu

`KGDB` should be enabled in the kernel
```conf
# CONFIG_STRICT_KERNEL_RWX is not set
CONFIG_FRAME_POINTER=y
CONFIG_KGDB=y
CONFIG_KGDB_SERIAL_CONSOLE=y
CONFIG_KGDB_KDB=y
CONFIG_KDB_KEYBOARD=y
```

Add following options when executing qemu
* Add `kgdboc=ttyS0,115200` in the command line for the kernel to send kgdb's output to ttyS0 with baud rate 115200
* Add `kgdbwait` in the command line for the kernel to wait gdb connection
* Add `-serial pty` in the command line for qemu to redirect serial output

```sh
$ sudo qemu-system-x86_64 -smp 8 -cpu host -m 4G -enable-kvm \
	-kernel bzImage -initrd initrd.img \
	-append "root=UUID=70ddb78e-d051-11e8-836f-525400123456 ro console=ttyS0 kgdboc=ttyS0,115200 kgdbwait" -nographic -serial pty \
	-drive file=ubuntu-18.04.1-server-amd64.qcow \
	-virtfs local,path=/home,mount_tag=host0,security_model=passthrough,id=host0 \
	-device vfio-pci,host=01:00.0 \
	-net nic,model=virtio -net user,hostfwd=tcp::2222-:22
```

Qemu shows char device that is redirected and waits for gdb connection
```sh
QEMU 3.0.0 monitor - type 'help' for more information
(qemu) qemu-system-x86_64: -serial pty: char device redirected to /dev/pts/5 (label serial0)
```

## Connect GDB

Execute gdb with vmlinux
```sh
$ gdb vmlinux
```

Set target in gdb and continue
```sh
(gdb) target remote /dev/pts/5
Remote debugging using /dev/pts/5
0xffffffffb2f64144 in ?? ()
(gdb) continue
```

## Debug Modules

Load modules and find out their address
```sh
$ sudo insmod <modules>
$ sudo cat /sys/module/<modules>/sections/.text
```

Break the execution in qemu
```sh
$ echo g | sudo tee /proc/sysrq-trigger
```

Load symbols form modules
```sh
(gdb) add-symbol-file <modules> <address>
```

Change path for source if it's changed
```sh
(gdb) set substitute-path <from> <to>
```

## References
* [The Linux Kernel Documentation](https://www.kernel.org/doc/html/v4.18/dev-tools/kgdb.html)
* [Linux 커널 디버거 사용법 (1) - kgdb](http://studyfoss.egloos.com/5490783)
* [Linux 커널 디버거 사용법 (2) - kgdb](http://studyfoss.egloos.com/5491083)
* [Linux 커널 디버거 사용법 (3) - kgdb](http://studyfoss.egloos.com/5491211)

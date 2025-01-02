---
title: Qemu
---

# Qemu
{: .no_toc}

## Table of Contents
{: .no_toc .text-delta}

* TOC
{:toc}

## Commands

### Execute without GUI

Serial console should be enabled on the guest kernel (add `console=ttyS0` on the kernel command line)

Execute qemu with `-nographic`
```sh
$ qemu-system-x86_64 -smp 8 -cpu host -m 4G -enable-kvm \
	-kernel bzImage -initrd initrd.img \
	-append "root=UUID=70ddb78e-d051-11e8-836f-525400123456 ro console=ttyS0" \
	-drive file=ubuntu-18.04.1-server-amd64.qcow -nographic
```

### Connect from Host to Guest by SSH (Port Forwarding)

Execute the qemu with `-net nic,model=virtio-net user,hostfwd=tcp::2222-:22`
```sh
$ qemu-system-x86_64 -smp 8 -cpu host -m 4G -enable-kvm \
	-kernel bzImage -initrd initrd.img \
	-append "root=UUID=70ddb78e-d051-11e8-836f-525400123456 ro console=ttyS0" \
	-drive file=ubuntu-18.04.1-server-amd64.qcow -nographic \
	-net nic,model=virtio -net user,hostfwd=tcp::2222-:22
```

Connect from host to guest by ssh
```sh
$ ssh localhost -p 2222
```

### Mount a Host Directory to the Guest

Execute the qemu with `-virtfs local,path=<host dir>,mount_tag=host0,security_model=assthrough,id=host0`
```sh
$ qemu-system-x86_64 -smp 8 -cpu host -m 4G -enable-kvm \
	-kernel bzImage -initrd initrd.img \
	-append "root=UUID=70ddb78e-d051-11e8-836f-525400123456 ro console=ttyS0" \
	-drive file=ubuntu-18.04.1-server-amd64.qcow -nographic \
	-virtfs local,path=/home,mount_tag=host0,security_model=passthrough,id=host0 \
	-net nic,model=virtio -net user,hostfwd=tcp::2222-:22
```

Mount a host directory on the guest
```sh
$ sudo mount -t 9p host0 /home/host
```

### PCI Device Passthrough

IOMMU should be enabled on the host kernel. The kernel log should contain `IOMMU` message.
```sh
$ sudo dmesg | grep -e IOMMU
```

If IOMMU is not enabled, enable it using one of following options.
* Enable this feature on kernel configuration - `CONFIG_DMAR_DEFAULT_ON`
* Enable this feature on kernel command line - `intel_iommu=on` for intel or `iommu=pt iommu=1` for amd

Find out pci bus, vendor and device ID for the target device (pci bus: 01:00.0, vendor ID: 8086, device ID: 10b9 in this example)
```sh
$ sudo lspci -nnk
```

Unbind the target device from the current driver
```sh
# echo <pci bus> | sudo tee /sys/bus/pci/devices/<pci bus>/driver/unbind
$ echo 0000:01:00.0 | sudo tee /sys/bus/pci/devices/0000:01:00.0/driver/unbind
```

Bind the target device with vfio driver
```sh
$ sudo modprobe vfio
$ sudo modprobe vfio_pci
# echo <vendor ID> <device ID> | sudo tee /sys/bus/pci/drivers/vfio-pci/new_id
$ echo 8086 10b9 | sudo tee /sys/bus/pci/drivers/vfio-pci/new_id
```

Execute qemu with `-device vfio-pci,host=<pci bus>`
```sh
$ sudo qemu-system-x86_64 -smp 8 -cpu host -m 4G -enable-kvm \
	-kernel bzImage -initrd initrd.img \
	-append "root=UUID=70ddb78e-d051-11e8-836f-525400123456 ro console=ttyS0" \
	-drive file=ubuntu-18.04.1-server-amd64.qcow -nographic \
	-virtfs local,path=/home,mount_tag=host0,security_model=passthrough,id=host0 \
	-device vfio-pci,host=01:00.0 \
	-net nic,model=virtio -net user,hostfwd=tcp::2222-:22
```

### Control Qemu with nographic Option

Change the frontend from the console to the monitor
```sh
Ctrl-a c
```

Use comamnd on the monitor
```sh
(qemu) system_reset	# reset the system
(qemu) q			# quit the emulator
```

Change `Ctrl-a` to `Ctrl-z` - `-echr 26`
```sh
$ sudo qemu-system-x86_64 -smp 8 -cpu host -m 4G -enable-kvm -echr 26 \
	-kernel bzImage -initrd initrd.img \
	-append "root=UUID=70ddb78e-d051-11e8-836f-525400123456 ro console=ttyS0" \
	-drive file=ubuntu-18.04.1-server-amd64.qcow -nographic
```

## References

* [Qemu version 3.0.93 User Documentation](https://qemu.weilnetz.de/doc/qemu-doc.html#mux_005fkeys)
* [How to assign devices with VT-d in KVM](http://www.linux-kvm.org/page/How_to_assign_devices_with_VT-d_in_KVM)

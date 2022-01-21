---
title: "Booting Qemu-KVM VM Without Grub (or any bootloader)"
date: 2022-01-21T16:36:17+01:00
draft: false
tags: ["system", "qemu"]
category: ["system", "virtualization"]
image: ""
description: "How to boot a linux VM without all the bootloader unnecessary shit."
---

I am looking more and more into [Cloud-Hypervisor](https://github.com/cloud-hypervisor/cloud-hypervisor) or [Firecracker](https://firecracker-microvm.github.io/). 
The other day I had this random thoughts: "These trendy new hypervisor can boot a kernel directly, why can't qemu do it?".

It turned out that it can _perfectly_ does that. Here is a random recipe to have a working Ubuntu, without having to use grub or any bootloader. 
Note that ubuntu first boot is _still_ utterly slow and bloated (snap and cloud-init, I'm looking at you !).

* First download ubuntu rootfs + initramfs + kernel [here](https://cloud-images.ubuntu.com/focal/current/). 
For our usage you will need :
  * `focal-server-cloudimg-amd64-root.tar.xz`: the rootfs that you will unpack into a brand new disks that we will create
  * `focal-server-cloudimg-amd64-initrd-generic` from the [`unpacked` folder](https://cloud-images.ubuntu.com/focal/current/unpacked/)
  * `focal-server-cloudimg-amd64-vmlinuz-generic`: a kernel

Here is the script to get it all

```bash
#!/bin/bash
wget https://cloud-images.ubuntu.com/focal/current/focal-server-cloudimg-amd64-root.tar.xz
wget https://cloud-images.ubuntu.com/focal/current/unpacked/focal-server-cloudimg-amd64-initrd-generic
wget https://cloud-images.ubuntu.com/focal/current/unpacked/focal-server-cloudimg-amd64-vmlinuz-generic
```

* We then need to create a disk to put our rootfs content into it:

```bash 
# create a 4G file
dd if=/dev/zero of=focal-rootfs.ext4 bs=1 count=0 seek=4G
# create an ext4 partition of this file, yes you can format in ext4 a simple file, how awesome.
mkfs.ext4 -F -L linuxroot focal-rootfs.ext4
# if not root you will need root access for this command
mount -o loop focal-rootfs.ext4 /mnt
# put the content of the rootfs, into the partition.
tar -C mnt/ -xJf focal-server-cloudimg-amd64-root.tar.xz
# putting our ssh public key in order to connect easily to the VM.
mkdir -p /mnt/home/ubuntu/.ssh/
cat ~/.ssh/ed_25519.pub >> /mnt/home/ubuntu/.ssh/authorized_keys
# and of course, unmount it.
umount mnt/
```

And finally our `qemu` command:

```
qemu-system-x86_64 \
-enable-kvm \
-kernel focal-server-cloudimg-amd64-vmlinuz-generic \
-m 4G \
-boot c \
-append 'root=/dev/vda rw console=ttyS0' \
-device e1000,netdev=net0 \
-netdev user,id=net0,hostfwd=tcp::5555-:22 \
-drive if=virtio,format=raw,file=focal-rootfs.ext4 \
-initrd focal-server-cloudimg-amd64-initrd-generic \
-nographic
```


* `-enable-kvm`: obviously, we want a VM that runs with the CPU virtual instructions.
* `-kernel` we specify which kernel to use. This means you can use whichever kernel you want (if you decide to compile your own).
* `-m 4G` 4Gio of memory allocated to this VM.
* `-boot c` boot to disk (don't try to boot to network or something fancy).
* `-append`: append the string to the kernel when booted. In our case, we are telling the kernel our disk is a `/dev/vda` and read-write (`rw`). And to output the console to serial 
S0 (`ttyS0`), qemu will foward ttyS0 to your terminal (stdout).
* `-device -netdev`: add a network interface to the system, with a shenigan to have the port 22 forwared to the port 5555 on localhost. Which will allow us to connect via ssh easily.
* `-drive ...`: add a virtio disks to the VM. This is for this reason that you need to pass `/dev/vda` and not `/dev/sda` to the kernel since it knows that the disks is a virtio device.
* `-initrd`: nothing to say about the ramfs, but you can have your own too.


I have no idea whatsoever if this will come handy one day but I struggle a bit with qemu and ubuntu since the documentation was sparse. The [invocation doc](https://www.qemu.org/docs/master/system/invocation.html) helps a lot though.

---
title: "How to Fix Grub (EFI boot) on Antergos"
date: 2018-09-16
tags: [linux, efi, boot, antergos]
header:
  image: "/images/background1.jpg"
excerpt: "Fix Grub, EFI boot, on Antergos Linux"
---

## The Problem
I have an HP laptop setup with dual boot between Windows 10 and Antergos (Arch Linux derivative).  Windows 10 updates as well as the Intel BIOS updates tend to remove grub from  the EFI partition.

While recently booted into Windows I saw that there was a BIOS update for my laptop. After installing the update I no longer had the grub menu at boot and could not select my Antergos install.  The laptop would boot directly into Windows 10.

I still had the usb flash drive with the live iso of Antergos I used for installation. I plugged in the usb flash drive and rebooted the laptop. On my HP laptop I need to press and hold the ESC key at startup to get to a menu (F9) that allows me to select which drive from which to boot.

I performed the steps listed in the next section to reinstall grub.

**WARNING**: The following solution will have you manipulating the partitions on your HDD or SSD.
Make sure you have backups. If you do not have backups, use the live iso to backup your linux partition, reboot into Windows, and make a backup. It's probably also a good time to make sure you have a way to reinstall Windows in case things go very wrong.

## The Solution

Open a terminal and use the following commands to list the devices and partitions.

```bash
sudo fdisk -l

Disk /dev/sda: 489.1 GiB, 525112713216 bytes, 1025610768 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 45DFB657-4D52-4F6A-8598-0B77B7295687

Device Start End Sectors Size Type
/dev/sda1 2048 534527 532480 260M EFI System
/dev/sda2 534528 567295 32768 16M Microsoft reserved
/dev/sda3 2758544 730187775 727429232 346.9G Microsoft basic data
/dev/sda4 998623232 1000630271 2007040 980M Windows recovery environment
/dev/sda5 1000630272 1025599487 24969216 11.9G Microsoft basic data
/dev/sda6 730187776 792687775 62500000 29.8G Linux filesystem
/dev/sda7 792687776 808312775 15625000 7.5G Linux filesystem
/dev/sda8 808312776 998623231 190310456 90.8G Linux filesystem

Partition table entries are not in disk order.
```

From the above results I can see that the EFI boot partition is /dev/sda1.
I also can recall that when I originally installed Antergos I created:
/dev/sda6 - root partition
/dev/sda7 - swap partition
/dev/sda8 - home partition

Next, the root partition should be mounted

```bash
	# change to su, be careful!
	sudo su

	# /dev/sda6 contains my root partition
	mount /dev/sda6 /mnt
```

Next, the boot partition, if you have one, should be mounted.
I do not have a boot partition but if I did I would execute the following
command, replacing /dev/sdXn with the actual partition.

```bash
# /dev/sdXn contains boot partition
mount /dev/sdXn /mnt/boot
```

Next, mount the EFI boot partition

```bash
# /dev/sda1 contains my EFI boot partition
mount /dev/sda1 /mnt/boot/efi
```

Now we need to change the mounted partitions to be our "real" partitions.

```bash
arch-chroot /mnt
```

Now we should run os-prober to identify the Windows 10 os.

```bash
os-prober
```

In my case, I received an error related to my usb drive (/dev/sdab) which was running the live iso.  I could not find a way to make os-prober ignore the usb drive so I continued and I believe this was the source of an issue I later had (more below).

Next, grub-mkconfig and grub-install are used to reginerate and install grub.

```bash
grub-mkconfig -o /boot/grub/grub.cfg

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Antergos-grub
```

If you receive any errors, go back and check that you have the correct partitions mapped.  You have backups, right?

## Success?

Almost. When I rebooted I received the grub menu but my Windows install was not listed in the grub menu.

Momentary PANIC!

Voice in my head: "You have a backup, right?"
Me: "Yes... when did it last run?"
Voice in my head: "You have a way to reinstall Windows, right?"
Me: "Uh... Yes! I created the USB installer earlier this year when installing a new SSD."

Then I remembered the issue I had when running os-prober.

I selected Antergos from grub and once in Linux, I opened a terminal and ran the following commands again.

```bash
sudo su

os-prober

grub-mkconfig -o /boot/grub/grub.cfg

grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Antergos-grub
```

The above commands completed without error and when I rebooted the grub menu appeared and both Antergos and Windows were listed.

Success!

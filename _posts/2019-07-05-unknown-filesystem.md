---
title: "Unknow Filesystem"
date: 2019-07-05
tags: [linux, grub, efi, antergos, arch]
excerpt: "unknown filesystem at grub rescue prompt"
---

# Boot What? 
If you have read any of my earlier posts, you may know that I have a dual booting laptop (Windows 10 and Antergos Linux).
You may have also seen my previous post about issue with grub being removed after certain Windows updates (<a href='../efi-boot-antergos'>"How to Fix Grub (EFI boot) on Antergos"</a>)
After a recent Windows major update, to Windows 10 1903, I had a new problem.  At startup I would get a black screen with the text:
```bash
unknow filesystem
grub rescue >
```

# The Solution
Luckily there are already people who have had this problem and posted solutions in blog posts and on Youtube.
I'm posting my own solution, copied from others, so that I can find it more easily when this happens again.
At the "grub rescue >" prompt enter the "ls" command.
```bash
ls
(hd0,gpt0) (hd0,gpt1) (hd0,gpt2) ... 
```
You will see a series of disks and partitions listed. I believe mine went up to (hd0,gpt11).
Next, do an "ls" on each item that was listed.
```bash
ls (hd0,gpt0)
ls (hd0,gpt1)
ls (hd0,gpt2)
...
ls (hd0,gpt9)
filesystem is ext2
```
Keep performing "ls" on each until you get a response of "filesystem is ext2".
Once found, this is the hard drive/partition you should use with the following commands:
```bash
set boot=(hdo,gpt9)
set prefix=(hd0,gpt9)/boot/grub
insmod normal
normal
```
After entering the last command "normal", the screen should go blank and then the grub boot menu will appear.

## Almost done
The good news is that the /boot/grub/grub.cfg has not been removed or modified.
After getting back to the grub boot menu, boot into Linux.
Open a terminal and run the follwing command to re-install the grub boot menu:
```bash
sudo grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=Antergos-grub
```
Afterwards, I was able to boot my laptop and get the grub boot menu as normal.
If you do not run the grub-install command the next time the computer restarts you will be back at the "grub rescue >" prompt.

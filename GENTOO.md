# Gentoo Linux

## Network

First off, you'll need access to the internet!

Find your IP address:
```
ip addr
```

Choose one - if you want to use ethernet, use the one that starts with an `e`. If you want to use WiFi, choose the one that starts with a `w`.

If you're using WiFi:
```
net-setup wlp3s0
```

And `net-setup` will ask you some connections about your network. Your network *probably* has DHCP, do choose automatically generate with DHCP when prompted.


Now, test your connection:
```
ping 8.8.8.8
```
If you get some sort of error, try re-doing these steps (or set it up manually).

## Partitioning

```
parted -a optimal /dev/sda
```

This will open up a command console for modifying disks. See your existing partitions with:

```
p
```

Switch to a more sensible unit system for our current process (megabytes):
```
unit mib
```

First, delete all partitions by creating a new label:
```
mklabel gpt
```

Create our first, 2MB partition (don't change this number!):
```
mkpart primary 1 3
```

This means start at the first megabyte, end at the 3rd. `3 - 1` = 2 MB partition.

Now, set the name to `grub` (this will be holding grub, the BIOS boot):
```
name 1 grub
set 1 bios_grub on
```

Feel free to print now to see the first partition completed:
```
p
```
(File System should be blank)

Now make your EFI Partition (don't change this either!):
```
mkpart primary 3 131
name 2 boot
```

Now, create a swap partition (recommended start is 512MB, but go as high as you'd like! ~10gb is a lot.):
```
mkpart primary 131 643
name 3 swap
```

And finally, set the final, full partition size (see notes below code blocks):
```
mkpart primary 643 -1
name 4 rootfs
```
`-1` means to the end of your disk. Adjust as needed!

Now, some touchups:
```
set 2 boot on
p
```

You should get output like this (numbers may differ):
```
Number   Start      End      Size     File system  Name   Flags
 1       1.00MiB    3.00MiB  2.00MiB               grub   bios_grub
 2       3.00MiB    131MiB   128MiB                boot   boot
 3       131MiB     643MiB   512MiB                swap
 4       643MiB     20479MiB 19836MiB              rootfs
```

Done? Type:
```
quit
```

## Partition formatting

Time to format! We'll use `ext*` formats in this tutorial. Also, we'll activer our `swap` storage partition.
```
mkfs.ext2 /dev/sda2
mkfs.ext4 /dev/sda4
mkswap /dev/sda3
swapon /dev/sda3
```

And finally, mount your main partition to /mnt/gentoo so we can edit it!
```
mount /dev/sda4 /mnt/gentoo
```

## Grabbing the install files

First, check on the date of the system:
```
date
```

It would be generally preffered that this is set to UTC time. **If it's not**, set it by running:

```
date MMDDhhmmYYYY
```

Now, let's grab the actual files:
```
cd /mnt/gentoo
```

**Now**, head over to the [Gentoo downloads page](https://www.gentoo.org/downloads/#other-arches), right click on the first Stage 3, select "Copy Link Address", and open a new tab to paste the text you just copied. Don't go to the link, just paste it.

Now, type:

```
wget <LINK YOU COPIED>
```


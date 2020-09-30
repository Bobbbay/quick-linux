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

Validating your files is currently not supported in this guide, so check out [the official instructions](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage#Verifying_and_validating) if you'd like to do it (optional).

Now, let's 'unzip' the file we just got:
```
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```
## Configuration
Now, we need to configure `portage`, which manages your packages/apps.

```
nano /mnt/gentoo/etc/portage/make.conf
```

Will open a text editor to the configuration file.

Modify the following line:
```
COMMON_FLAGS="-march=native -O2 -pipe"
```

And add the following line:
```
MAKEOPTS="-j2"
```
**Replacing** the `2` with your RAM divided by 2. For example, if I have 8GB of RAM, write `-j4`.

**Now, save and quit** by pressing `Ctrl+S` and `Ctrl+X`.

## Grabbing the mirrors

Run the following to select mirrors. It will open a visual box. Press space to select mirrors in countries near you - 5-10 is generally enough. Quick tip: select mirrors with `https` or `rsync` more to have a more "secure" connection.

```
mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf
```

Once you're done, press `Enter`. You'll be taken out of the visual box and back to the terminal (see the bottom of the screen).

## Grabbing the repository

```
mkdir --parents /mnt/gentoo/etc/portage/repos.conf
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

## Touchups
A small bit remains before running the `chroot` command.

```
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
```


## Entering the `chroot` environment
```
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(chroot) ${PS1}"
mount /dev/sda2 /boot
```

Now, this is your actual system-to-be!

## Syncing

Now, while we still have an internet connection, we need to sync with the rest of the web.

```
emerge-webrsync
```

Now, let's sync `emerge`, Gentoo's package manager.
```
emerge --sync --quiet
```

... for slow terminals/devices, or...

```
emerge --sync
```

This will take some time, but don't worry - it's syncing information about every single application/package that's available on Gentoo!

## Reading the news

You may have noticed a message prompting you about news. It's generally recommended to read through the news, in case there is an important piece of information.

```
eselect news list
```

Read a specific piece of news:
```
eselect news read 1
```

Read all of the content:
```
eselect news read
```

Delete your news items:
```
eselect news purge
```

## Setting a `profile`
Profiles are the building blocks of your Gentoo system.

```
eselect profile list
```

Here, we will be selecting the raw desktop profile - this gives us access to installing any graphical Desktop Environment or Window Manager later on.

The magic number we need at the moment is in that list. You want to choose the newest version number (for us, 17.1). Now, find the line that says this:

```
[##]  default/linux/amd64/17.1/desktop
```

If you have a newer version number than `17.1`, use that. Now, our magic number is the number in `[##]`.

Now, we select what we want:
```
eselect profile set {magic number}
```

In our case, (maybe not yours), `20`:
```
eselect profile set 20
```

Make sure this number is correct!

```
eselect profile list
```

And there should be a blue asterisk beside the profile you chose.

## Updating @world
```
emerge --ask --verbose --update --deep --newuse @world
```

You'll be asked if you want to proceed - press `y` then enter, or just immediately `enter` - they do the same thing!

This will take quit a while, so sit back! You're re-compiling every package that you have installed!

## Setting a timezone
All done updating `@world`? Now, let's set a timezone.

```
ls /usr/share/zoneinfo
```

Will output all available timezones.

To get more in-depth, you can keep going, for example:

```
ls /usr/share/zoneinfo/Canada
```

Will give us all of the available timezones in Canada!

Once you know which one you're using, run the following command:

```
echo "Canada/Eastern" > /etc/timezone
```

Obviously replacing "Canada/Eastern" with your timezone.
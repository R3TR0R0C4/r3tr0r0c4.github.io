---
title: "Installing Gentoo Linux on an iMac (2014 A1418)"
date: 2026-02-08
categories: [Linux, Gentoo, Apple] 
tags: [gentoo, imac, installation, budgie]
---

## What is this guide?

This guide details the process of installing Gentoo Linux on a 2014 iMac (Model A1418). In simple terms, this installation will:

* Replace the existing macOS (or other OS) with a highly optimized Linux system.
* Configure specific hardware support for Intel-based Apple hardware.
* Set up the Budgie Desktop Environment for a modern, lightweight GUI.

Instead of a generic install, this guide focuses on specific tweaks required for the iMac hardware, such as the Broadcom Wi-Fi and Intel graphics.

## Why Gentoo on an old iMac?

### Performance

* The 2014 iMac hardware is aging; Gentoo allows you to compile software specifically for the Intel CPU architecture, squeezing out maximum performance.
* No bloatware; you only install exactly what you need.

### Customization

* Choose your own init system (systemd in this guide) and desktop environment.
* Granular control over compilation flags (USE flags).

### Learning

* Great way to understand how Linux works from the kernel up to the display manager.

---

## Preparations

Before installing, we must prepare the system. This involves ensuring system time is correct, partitioning the drive, and formatting filesystems.


#### Time Synchronization

Ensure the system clock is accurate to avoid https certification errors.

```bash
chronyd -q
```

#### Disk Partitioning

We will use `parted` to create a GPT partition table with three partitions: EFI Boot, Swap, and Root (xfs).

```bash
parted /dev/sda --script \
  mklabel gpt \
  mkpart ESP fat32 1MiB 513MiB \
  set 1 esp on \
  mkpart primary linux-swap 513MiB 8705MiB \
  mkpart primary xfs 8705MiB 100%
```

#### Formatting

Apply the filesystems to the newly created partitions:

```bash
mkfs.fat -F32 /dev/sda1
```

```bash
mkswap /dev/sda2
swapon /dev/sda2
```

```bash
mkfs.xfs /dev/sda3
```

## Base System Installation


#### Mounting and Stage3

Mount the root and boot partitions to prepare for the installation.

```bash
mount /dev/sda3 /mnt/gentoo
```

```bash
mkdir -p /mnt/gentoo/boot
mount /dev/sda1 /mnt/gentoo/boot
```

Check the current filesystem, using `lsblk`, Should look similar to:

```bash
sda      8:0    0 931.5G  0 disk
├─sda1   8:1    0   512M  0 part /mnt/gentoo/boot
├─sda2   8:2    0     8G  0 part [SWAP]
└─sda3   8:3    0   923G  0 part /mnt/gentoo
```

Navigate to the mount point and download the [**Stage3 tarball**](https://wiki.gentoo.org/wiki/Stage_file) (the base system files) using `links`.

```bash
cd /mnt/gentoo
links https://www.gentoo.org/downloads/
```

Extract the stage3 archive ensuring attributes are preserved:

```bash
tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
```

#### Chrooting

Now that the stage3 archive is decompressed, we've got the base of our system, that we'll start to install on with a chroot.

First copy the DNS configuration to the stage:

```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

And enter the new environment, we must mount the necessary system filesystems and change the root.

```bash
mount --types proc /proc /mnt/gentoo/proc
mount --rbind /sys /mnt/gentoo/sys
mount --make-rslave /mnt/gentoo/sys
mount --rbind /dev /mnt/gentoo/dev
mount --make-rslave /mnt/gentoo/dev
```

Enter the environment:

```bash
chroot /mnt/gentoo /bin/bash
source /etc/profile
export PS1="(gentoo) ${PS1}"
```

## Core Configuration Concepts

Gentoo relies heavily on configuration files to determine how software is compiled and installed.

* Main config: `/etc/portage/make.conf`
* Package specific config: `/etc/portage/package.use/`


#### Configuring make.conf

Edit the main configuration file:

```bash
nano /etc/portage/make.conf
```

Add the following content to optimize for the iMac and the intended desktop environment:

```conf
# Compilation flags
MAKEOPTS="-j4 -l4"

# Hardware Support
VIDEO_CARDS="intel i915"
INPUT_DEVICES="libinput"

# License acceptance
ACCEPT_LICENSE="*"

# Global USE flags (Features to enable/disable system-wide)
USE="X wayland gtk gnome vaapi systemd dbus udev udisks pipewire bluetooth wifi screencast threads zstd -cups -nvidia -amdgpu -elogind"
```

#### Syncing and Profiling

Update the Gentoo repositories:

```bash
emerge --sync
```

Install `mirrorselect` to choose the best mirror location-wise and `cpuid2cpuflags` to optimize USE compilation flags. ([Learn more](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Optional:_Selecting_mirrors))

```bash
emerge --ask --quiet app-portage/mirrorselect app-portage/cpuid2cpuflags
```

Select the best download mirrors and set CPU flags automatically:

```bash
mirrorselect -i -o >> /etc/portage/make.conf
echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```

Select the system profile (ensuring you select a systemd desktop profile):

```bash
eselect profile list
eselect profile set X
```

(Where X is something similar to `default/linux/amd64/23.0/desktop/systemd`)

Update the entire system to match the new profile and flags:

```bash
emerge --ask --update --deep --newuse --quiet @world
```

#### Locale and Timezone

Set the timezone (Example: Madrid):

```bash
ln -sf ../usr/share/zoneinfo/Europe/Madrid /etc/localtime
emerge --config sys-libs/timezone-data
```

Generate locales by editing `/etc/locale.gen` and running generation:

Uncomment keyboard your language of choice something similar to: `en_US.UTF-8` (Always use UTF-8 if posible, as it gives the best compatibility)

```bash
nano /etc/locale.gen
locale-gen
```

List currently available locales and select it (Using the number on the left):

```bash
eselect locale list
eselect locale set X
```

Where X is your desired language


Update yout currently running config:

```bash
env-update && source /etc/profile
```

Add the hostname of the computer to `/etc/hostname`

```bash
echo "imac-gentoo" > /etc/hostname
```

---

## Kernel & Bootloader

#### Microcode And Firmware


It's very important to install the Intel microcode for our iMac, or else the grub installation might include some AMD microcode (If you are compiling the kernel from makemenu this wouldn't be necessary.)

```bash
emerge --ask --quiet sys-firmware/intel-microcode
```

Install the gentoo kernel and linux firmware, and dracut (to create initramfs).

```bash
emerge --ask --quiet sys-kernel/linux-firmware sys-kernel/gentoo-kernel sys-kernel/dracut
```

#### Fstab Configuration

Configure the `/etc/fstab` file to mount the `/boot` and `/` partitions.

Using `lsblk` find out the UUID of each partition.

```bash
lsblk -o PATH,TYPE,MOUNTPOINT,UUID
nano /etc/fstab
```

Add them to the `fstab` Like shown:

```conf
# EFI System Partition (/boot)
UUID=XXXX-XXXXX...                            /boot       vfat    defaults,noatime      0 2

# Root Partition (/)
UUID=XXXX-XXXXX... /           xfs     defaults,noatime      0 1
```

#### Bootloader (GRUB)

Install and configure GRUB for EFI booting:

```bash
emerge --ask --quiet sys-boot/grub
```

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Gentoo
grub-mkconfig -o /boot/grub/grub.cfg
```

> 
> *Note for Intel Systems*: Dracut may include *AMD microcode* by default which can be problematic. 
> You can remove it and regenerate the initramfs if needed:
> `rm /boot/amd-uc.img`
> `dracut --force --kver 6.12.63-gentoo-dist` (Keep in mind this is specific to kernel version 6.XX.XX, change to your version's kernel)
> `grub-mkconfig -o /boot/grub/grub.cfg`
{: .prompt-danger }


## Desktop Environment: Budgie

We will use the Budgie desktop environment.

#### Repository and Base Install

Enable the overlay required for Budgie:

```bash
emerge --ask --quiet app-eselect/eselect-repository dev-vcs/git

eselect repository enable SarahMiaOverlay
emerge --sync SarahMiaOverlay
```


You have two options for installation:

##### Option A: Minimal Installation

```bash
emerge --ask --quiet budgie-base/budgie-meta
```

##### Option B: Full Desktop Suite 

Add the flag to `/etc/portage/package.use/budgie-meta`:

If the user wishes to use the full desktop env:

Set the USE flag on the `/etc/portage/package.use/budgie-meta` file to:

```bash
buddgie-base/budgie-meta all-packages
```

And then install:

````bash
emerge --ask --quiet budgie-base/budgie-meta
````

#### Display Manager (LightDM)

Install and enable LightDM:

```bash
emerge --ask x11-misc/lightdm
systemctl enable lightdm
```

Configure it to auto-start with budgie, edit `/etc/lightdm/lightdm.conf` and on the `[Seat:*]` section add:

```conf
user-session=budgie-desktop
```

Edit `/etc/pam.d/lightdm` to unlock the keyring on login:

```bash
auth     optional        pam_gnome_keyring.so #keyring
session  optional        pam_gnome_keyring.so auto_start #keyring
```


#### Network & Utilities

Install essential networking and system utilities before rebooting.

```bash
emerge --ask --quiet net-misc/dhcpcd net-wireless/iw net-wireless/wpa_supplicant net-misc/networkmanager
systemctl enable dhcpd
systemctl enable NetworkManager

# System Utilities
emerge --ask --quiet sys-apps/mlocate app-shells/bash-completion sys-fs/xfsprogs sys-block/io-scheduler-udev-rules app-admin/sudo
systemd-machine-id-setup
systemctl preset-all
systemctl enable sshd
```

##### Drivers

Install display drivers for the iris pro 5200:

```bash
emerge --ask --quiet media-libs/libva-intel-driver
```

Add a new user to not use root:

```bash
useradd -m -G wheel,audio,video,plugdev -s /bin/bash youruser
passwd youruser
```

#### Final Steps:

Exit the chroot, unmount, and reboot into your new system.

```bash
exit
cd
umount -l /mnt/gentoo/dev{/shm,/pts,}
umount -R /mnt/gentoo
reboot
```


You now should reboot automatically insto a working gentoo desktop.

<br>

## Troubleshooting
Here are common errors encountered during installation on this hardware.

* Snapshot Timestamp Error

  Symptom: When running emerge --sync, it fails claiming the local timestamp is newer than the snapshot. Fix:

  1. Remove the timestamp file: `rm /var/db/repos/gentoo/metadata/timestamp.x`
  2. Re-run `emerge --sync`
  3. Reason: This usually happens if the command was run twice in quick succession (lock file issue) or if the system time is incorrect. Check time with `chronyd -q`.

<br>

* Config File Needs Updating

  Symptom: Installation pauses with "IMPORTANT: config file XXX needs updating". Fix:

  1. Allow the install command to finish.
  2. Run `etc-update`.
  3. Select option `-3` (Auto-merge).
  4. Reason: Changes in USE flags or package updates require configuration file changes. Option `-3` intelligently merges these updates.


* AMD microcode in the `/boot` partition.

  Symptom: Finding AMD microcode in the `/boot` partition after using dracut and grub.

  1. Remove the AMD microcode: `rm /boot/amd-uc.img`
  2. Install the Intel Microcode: `emerge --ask sys-firmware/intel-microcode`
  3. Install initramfs forcinf the gentoo-kernel version (Use your kernel version) `dracut --force --kver 6.XX.XX-gentoo-dist`
  4. Install the new grub configuration: `grub-mkconfig -o /boot/grub/grub.cfg`
  5. Reason: Installing the `gentoo-kernel` includes AMD and INTEL Microcode, and the installer usually finds AMD's microcode first and installs it. Usually installing the Intel microcode first fixes it.
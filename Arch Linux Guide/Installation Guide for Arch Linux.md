# Installation Guide for Arch Linux

## Archive and burn the installation media

Head to [Arch's official website](https://endeavouros.com/#Download) to choose an ISO and archive

It's recommended to make use of [Etcher](https://etcher.balena.io/) to burn the ISO image into a USB flash drive.

## Manipulate UEFI BIOS settings to enter the Live ISO

> What we are trying to address through the term "Arch Linux Live ISO" is its environment, which means you can always use others to get the job done. But we recommend you to use the EndeavourOS Live ISO to finish the task if you will have to configure your live environment such as connecting to an education network.

### For PCs from ASUS:

Hit `Escape` when the ASUS or ROG logo was displayed on the screen to enter the boot menu, then select `Enter UEFI Setup`.

After entering, please hit `F7` to access to the "Advanced Menu", then go to the "Security" column and switch off "Secure Boot" or alternate it from `Windows UEFI` to `Other OS` or `Microsoft & 3rd Party`.

Apply settings and reset, then enter the boot menu again and choose `UEFI: <Your Device Name>` to load the `grub` bootloader, and select `Arch Linux install medium (x86_64, x64 UEFI)` to enter the live environment.

## Preparation before installation

### Select your keymap

> We'll be using the standard US keyboard in this guide.

To list all keymaps, execute

```bash
localectl list-keymaps
```

> You could use `|` (pipe) to redirect the output to `grep` to have it filtered. For example, if we want to filter US keyboards and its variations:

```bash
localectl list-keymaps | grep us
```

The default keymap is the standard English (US) keyboard, please adjust according to your need.

### Select what font to be used in console

Letters might be too tiny to see in tty on a Hi-DPI display, that's when you can change your font.

> Please note that console fonts are stored in `/usr/share/kbd/consolefonts/`.

To change your font, simply execute:

```bash
setfont <font that you want>
```

For example:

```bash
setfont <ter-132b>
```

### Connect to the Internet

To check all network interfaces, execute:

```bash
ip link
```

Then check the state of each interface, `UP` indicates it's enabled.

To use WLAN or WWAN interfaces, check its status via `rfkill` first.

> `rfkill` is a tool that can be used to manage network interfaces in GNU/Linux.

To check all network interfaces, run:

```bash
rfkill list
```

You can enable an interface by executing:

```bash
rfkill enable <ID>
```

Take this output as an example:

```
0: phy0: Wireless LAN
Soft blocked: yes
Hard blocked: no
2: hci0: Bluetooth
Soft blocked: yes
Hard blocked: no
```

It's applicable to execute the following command to unblock the WLAN interface:

```sh
rfkill unblock 0
```

Now, we are able to try to connect to the Internet.

- For ethernets, please check if your cable was plugged in firmly.
  
- For WLAN, please use `iwctl` or `nmtui` to authorise and join a network.

> `iwctl` is a CLI programme to manage `iwd` (iNet Wireless Daemon) interactively; `nmtui` is a TUI programme to manage `NetworkManager` and it's more user friendly.

To use `iwctl` to check all available interfaces, execute:

```sh
iwctl
[iwd] > device list
```

To obtain a list of available WLAN SSIDs, run:

```sh
[iwd] > station <Name> get-networks
```

Execute the following command to connect to your desired WLAN access point:

```sh
[iwd] > station <Name> connect <SSID>
```

> `<Name>` is the interface that you want to use, `<SSID>` is the name of an access point.

Enter your passcode to connect when `Passphrase:` was prompted.

To use `nmtui` to connect to an access point, execute:

```bash
nmtui
```

Then select `Activate a connection` from the menu and choose a WLAN SSID to connect and exit the programme.

- For WWAN modem, you can use `mmcli`

> Given that WWAN modems are pretty rare these days, we will not give instructions here. (Related link [from Arch Wiki](https://wiki.archlinux.org/title/Mobile_broadband_modem#ModemManager))

#### Verify Internet access

Execute:

```sh
ping www.archlinux.org
```

Typically, if the output shows `time=<latency> ms`, then we are good to complete the installation since this indicates we have access to the Arch Linux repository.

### Choose a timezone for the operating system

Execute the following command to check the timezone and time of your current OS:

```bash
timedatectl
```

To list all available timezones, execute:

```bash
timedatectl list-timezone
```

Also, it's applicable to use `|` (pipe) and `grep` to filter the output.

To choose a timezone, please run:

```bash
timedatectl set-timezone <Timezone>
```

For instance, running this will set your timezone to Hong Kong and use HKT (UTC+8):

```bash
timedatectl set-timezone Asia/Hong_Kong
```

For CST (UTC+8), run:

```bash
timedatectl set-timezone Asia/Chongqing
```

### Prepare your disk

1. Check all disks

To list all disk partitions and differentiate, run:

```bash
lsblk
```

> `lsblk` means "list all block devices"

If you disks have the same capacity or even model, execute this to further investigate:

```bash
fdisk -l
```

> `fdisk` is a CLI programme to manage disk partition table, we are calling to "list all partition tables of all disks".

2. Alternate your partition table 

Execute this in your terminal:

```bash
cfdisk <target disk>
```

> `cfdisk` is a more user-friendly utility compared with `fdisk`, it provides with a Terminal User Interface (or Curses-based User Interface).

For example, to edit the partition table of `/dev/sda` with the programme, execute:

```bash
cfdisk /dev/sda
```

> Note that Arch Linux will store all vmlinuz kernel images (or bzImage) to `/boot`, so it's recommended that the partition should be no less than 500M. If you are about to use multiple kernels, adjust the capacity according to you demand.

Take a 60G disk as an example, after alternating its partition table, it might look like this:

| `/dev/sda` | Size | Type |
|-|-|-|
`/dev/sda1` | 0.5G | EFI System |
`/dev/sda2` | 40G | Linux root (x86-64) |
`/dev/sda3` | 19.5G | Linux home |

1. Initialise partitions

Please do run `lsblk` again to refer to the output

We will continue with the our instance above to make file systems:

```bash
mkfs.fat -F32 /dev/sda1 -n ARCH
mkfs.btrfs /dev/sda2 -f
mkfs.f2fs /dev/sda3 -f -l Home
```

> `mkfs` is a programme to make file systems on a partition.


4. Check your mirror list

Execute the following command to inspect and edit the mirror list:

```bash
nano /etc/pacman.d/mirrorlist
```

Add whichever mirror server you want and press `Control+O` to write in, then `Enter` to confirm to use the original file name, hit `Control+X` to exit.

For example, you can write in this on top to prioritise this mirror:

```
Server = https://mirrors.cernet.edu.cn/archlinux/$repo/os/$arch
```

To learn more about this mirror, head to its [official website](https://mirrors.cernet.edu.cn/about)

**Now, you are ready to proceed with the installation**

## Perform the installation in the Live ISO environment

### Install using `archinstall` helper

> To customise more, please refer to the manual installation section below.

To ensure your smooth installation, please always use the latest version of `archinstall`.

Execute this to get `archinstall` up-to-date:

```bash
pacman -Sy archinstall
```

> `pacman` is the package manager for Arch Linux, `-Sy` means synchronise with remote database and update local package index, you can interpret this as "update local databse and install `archinstall`".

`archinstall` is an automatic installer for Arch Linux, which gives us a "quiz" to collect necessary information to perform the installation.

> Pay attention to the *Mirrors* of `archinstall`, it would be better to select local mirror servers to speed up the process.

Also, select *Manual Partitioning* in *Disk configuration* and hit `Space` to the partition table you have just created, then `Enter` to proceed.

We will stick to our example given above:

| Status | Device | Size | FS type | Mountpoint | Mount options | Flags | Btrfs vol |
|-|-|-|-|-|-|-|-|
| existing |`/dev/sda1` | 524 MB | fat32 | `/boot` | | Boot, ESP | |
| modify | `/dev/sda2` | 42 GB | btrfs | | compress=zstd | | 2 subvolume |
| existing | `/dev/sda3` | 20 GB | f2fs | `/home` | | | |

As for btrfs subvolumes, you can consult to the following configuration:

| name | mountpoint | compress | nodatacow |
|-|-|-|-|
`@` | `/` | True | False |
`@log` | `/var/log` | False | True |
`@pkg` | `/var/cache/pacman/pkg` | False | True |

The rest can be altered to fit your preferrence.

| Options | Description |
|-|-|
| Mirrors | *Custom* |
| Locales | *Custom* |
| Disk configuration | Manual Partitioning |
| Disk encryption | |
| Bootloader | Grub |
| Swap | False |
| Hostname | *Custom* |
| Root password | ****** |
| User account | 1 User(s) |
| Profile | Desktop |
| Audio | Pipewire |
| Kernels | linux |
| Additional packages | *Custom* |
| Network configuration | Use NetworkManager |
| Timezone | *Custom* |
| Automatic time sync (NTP) | True |
| Optional repositories | multilib |

As for *Additional packages*, you could add:

`firefox` `nano` `fish`

In *Profile*, it is recommended to use KDE Plasma or Gnome if you are new to GNU/Linux.

Eventually, hit *Install* to start.

### Installing Arch in the manual way

> While `archinstall` provides us with an automatic installation, the manual approach gives us more flexibility. We will use LVM and `systemd-boot` to boot Unified Kernel Image in this case.

#### Prepare your LVM

> LVM stands for Linux Volume Manager, which relies on the device mapper of the Linux kernel to virtualise storage devices to a storage pool where you can create virtual partitions called Logical Volumes. This deals with the inflexibility of manipulating partitions on a disk that requires you to assign consecutive sectors.

Back to partition table editing via `cfdisk`, we can create only two partitions to keep it simple. One for `/boot`, the other for LVM physical volume.

So, we will still have to make `/dev/sda1` a FAT32:

```bash
mkfs.fat -F32 /dev/sda1
```

As for `/dev/sda2`, we will have to make it a LVM2 physical volume:

```bash
pvcreate /dev/sda2
```

> `pvcreate` stands for "Create Physical Volumes", simply assign block devices to get it done.

After that, create a volume group:

```bash
vgcreate Arch /dev/sda2
```

> `vgcreate` stands for "Create Volume Group" and give it a name (we named it "Arch" here). You can specify only one or multiple LVM physical volumes to create a group.

Finally, create logical volumes:

```bash
lvcreate -L 40G Arch -n root
lvcreate -l 100%FREE Arch -n home
```

> Easy to know that `lvcreate` stands for "Create Logical Volume" in a volume group and name it. You can allocate the capacity for volumes by simply implying size or by percentage of redundance.

#### Create file systems in LVM

> Logical volumes are partitions mapped under a volume group.

We will use btrfs for the root directory an f2fs for home here.

1. Create file systems

```bash
mkfs.btrfs /dev/Arch/root
mkfs.f2fs /dev/Arch/home
```

2. Mount and create btrfs subvolumes

Mount the whole btrfs container to `/mnt` by executing:

```bash
mount /dev/Arch/root /mnt
```

Create subvolumes in the container by running:

```bash
cd /mnt
btrfs subvolume create @
btrfs subvolume create @log
btrfs subvolume create @pkg
cd && umount /mnt
```

3. Mount neccessary partitions

First, mount the subvolume to be used as root directory:

```bash
mount /dev/Arch/root -o subvol=@,compress=zstd /mnt
```

Then, create mounting points for other partitions and subvolumes:

```bash
cd /mnt
mkdir boot home
mkdir -p ./var/log
mkdir -p ./var/cache/pacman/pkg
```

Now, mount all partitions and subvolumes:

```bash
mount /dev/sda1 /mnt/boot
mount /dev/Arch/home /mnt/home
mount /dev/Arch/root -o subvol=@log,nodatacow /mnt/var/log
mount /dev/Arch/root -o subvol=@pkg,nodatacow /mnt/var/cache/pacman/pkg
```

Run `lsblk` to double check and make sure all mounted correctly.

#### Install basic packages

> `pacstrap` is a seeder of Arch Linux, it can be used to create the whole tree structured hierarchy root directory and install some basic packages such as a package manager.

Execute the following command to install:

```bash
 pacstrap -K /mnt base base-devel linux-firmware linux linux-headers nano fish networkmanager btrfs-progs f2fs-tools dosfstools lvm2
```

For Intel CPUs, you will need `intel-ucode`, and `amd-ucode` for AMD CPUs.

> To use the `linux-zen` kernel, please replace `linux` `linux-headers` with `linux-zen` `linux-zen-headers`.

#### Create a file system table for the new installation

> You can use `genfstab` to generate a file system table automatically in the Arch Linux Live ISO.

To generate, run:

```bash
cd
genfstab -U /mnt > /mnt/etc/fstab
```

Then we'll have to inspect and edit `/mnt/etc/fstab` if needed.

```bash
nano /mnt/etc/fstab
```

- If you would like to use `fstrim.timer` for periodic trim, disable `discard` for desired file systems.

  For btrfs and f2fs, change `discard` to `nodiscard` in the options column.

#### Enter chroot environment to configure the system

> Arch Linux scripted `arch-chroot` to foster changing root directory, to login as root to the new system.

```bash
arch-chroot /mnt
```

1. Set local timezone

```bash
ln -sf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
```

> `ln -sf` will create a symbolic link named `/etc/localtime` that points to `/usr/share/zoneinfo/Asia/Hong_Kong`.

2. Set hostname

```bash
nano /etc/hostname
```

Enter your hostname in the first line, you could try to name in the `<owner>-<operating system>-<device model or brand>` pattern.

For example:

```
Jimmy-Arch-SP5
```

3. Set locales

```bash
nano /etc/locale.gen
```

> You MUST uncomment `en_US.UTF-8 UTF-8`. On this basis, you can uncomment whatever you want to have multilingual support.

Now, generate locales by executing:

```bash
locale-gen
```

4. Install neccessary packages, drivers and KDE Plasma
   
```bash
pacman -Sy openssh htop wget iwd wireless_tools wpa_supplicant smartmontools xdg-utils plasma-meta plasma-workspace ark dolphin konsole egl-wayland pipewire pipewire-alsa pipewire-jack pipewire-pulse gst-plugin-pipewire libpulse wireplumber sddm
```

For laptops that have hybrid configuration with Intel GPU and NVIDIA GPU:

```bash
pacman -S mesa intel-media-driver libva-intel-driver vulkan-intel xorg-server xorg-xinit nvidia-dkms
```

5. Configure Unified Kernel Image

> Arch Linux authors coded `mkinitcpio` and use it to create initramfs by default.

Create a directory for Unified Kernel Images:

```bash
mkdir -p /boot/EFI/Linux
```

Change settings for `mkinitcpio`:

```bash
nano /etc/mkinitcpio.conf
```

Add `lvm2` to the HOOKS line that have been uncommented to support mounting a root directory that resides in a LVM logical volume.

Now, change kernel initramfs generation settings:

```bash
nano /etc/mkinitcpio.d/<package name>.preset
```

Comment `default_image` and `fallback_image`.

Uncomment `default_uki` and `fallback_uki`, change the stored location from `/efi/EFI/Linux/<package name>.efi` to `/boot/EFI/Linux/<package name>.efi`.

Finally, set kernel parameters:

```bash
mkdir /etc/cmdline.d
```

> `mkinitcpio` will source `/etc/cmdline.d` for kernel parameters when building an image. You can use snippets in the folder.

To specify root directory mounting options:

```bash
nano /etc/cmdline.d/root.conf
```

Write the following content into this configuration file:

```
root=/dev/Arch/root rootflags=subvol=@ rootfstype=btrfs rw
```

Execute this to build all Unified Kernel Images for all kernels:

```bash
mkinitcpio -P
```

6. Install the `systemd-boot` bootloader

To install, execute:

```bash
bootctl install
```

Then change settings by running:

```bash
nano /boot/loader/loader.conf
```

It's recommended to uncomment `timeout 3` and `console-mode keep`.

Also, append these two lines:

```
auto-entries yes
editor yes
```

At last, run this command to check all boot entries:

```bash
bootctl list
```

7. Install packages

> If you are using the EndeavourOS Live ISO to install Arch Linux, please DO delete the lines about the EndeavourOS repository in `/etc/pacman.conf`.

Execute the following command to perform package installation:

```bash
pacman -S fcitx5 fcitx5-chinese-addons kcm-fcitx5 fcitx5-qt fcitx5-gtk ttf-sarasa-gothic snapper snap-pac power-profiles-daemon
```

> Just letting you know that `power-profiles-daemon` is a GUI power plan manager with great integration with Plasma. You could replace it with `tlp` if you like, but remember to enable services.

8. User management

Execute the following command as root to change the passcode for root:

```bash
passwd
```

To add a user and create a home directory, and add the user to wheel privileged group, run:

```bash
useradd -m -G wheel <username>
```

9. Allow a user to execute `sudo` for escalation

> `sudo` is the most common administration tool in GNU/Linux, it gives a user temporal root permission without switching user.

To edit the allowlist, run:

```bash
nano /etc/sudoers
```

Locate the line:

```
root ALL=(ALL:ALL) ALL
```

and insert this line below to permit a user to use `sudo`:

```
<username> ALL=(ALL:ALL) ALL
```

Then save the file and quit.

10. Unmount all disks

Back to the shell and execute `exit` or press `Control+D` to exit the chroot environment.

Execute `cd` to change directory to the user's home directory.

To unmount all disks, execute:

```bash
umount -Rl /mnt
```

Then reboot into Arch Linux to check if everything works fine.


**Now, you have finished the basic installation of Arch Linux**
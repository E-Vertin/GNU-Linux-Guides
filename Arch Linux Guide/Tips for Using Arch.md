# Tips for Using Arch

## Setting the same hardware clock as Windows (Local to HW)

Run the following command to set local time as hardware time:

```bash
timedatectl set-local-rtc 1 --adjust-system-clock
```

> Note that you could also change Windows settings via `regedit` to match the standard of GNU/Linux operating systems.

## Enable some `systemd` services

- `fstrim.timer` will activate `fstrim` once a week to perform `discard` on all supported file systems mounted on the system. This will tell the NAND flash controller on your SSD to run a set of algorithm to balance all written data across the whole flash.

  Execute this to have the timer enabled:

  ```bash
  systemctl enable --now fstrim.timer
  ```

- `nvidia-powerd.service` can dynamically manage NVIDIA GPU power consumption to reach its optimal state, you can enable the service on laptops.

  Simply run:

  ```bash
  systemctl enable nvidia-powerd.service
  ```

  > Avoid passing the `--now` parameter as NVIDIA Unix/Linux Driver might cause the kernel to panic!

## Add the archlinuxcn repository

We will have to add archlinuxcn repository manually as it is not verified by Arch Linux officially.

Edit the configuration file of `pacman` to add the repo:

```bash
nano /etc/pacman.conf
```

Move to the end of the file by pressing `Control+End` and append the following content:

```
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

> If you are in Chinese Mainland, try to use a local mirror server such as CERNET.

For example:

```
[archlinuxcn]
Server = https://mirrors.cernet.edu.cn/archlinuxcn/$arch
#Server = https://repo.archlinuxcn.org/$arch
```

Now, execute this to manually trust farseerfc's key:

```bash
pacman-key --lsign-key "farseerfc@archlinux.org"
```

> `pacman` will not pull in packages in a untrusted repository or by untrusted maintainers.

Then, run this command to obtain the PGP keys for archlinuxcn repository:

```bash
pacman -Sy archlinuxcn-keyring
```

(Optional) Install `archlinuxcn-mirrorlist-git` to acquire a copy of archlinuxcn repository mirror list for the convenience of importing it into `/etc/pacman.conf`.

## Enable the AUR (Arch Users' Repository)

`paru` is a feature-focused AUR helper

- Install via archlinuxcn repo
  
  Simply execute this command to install:

  ```bash
  pacman -S paru
  ```

- Also, you can compile and install it on your own

  First, you'll need these packages for this, install them by running:

  ```bash
  pacman -S --needed base-devel git
  ```

  > It's recommended to run the following commands in `~` -- your home directory.

  To get a copy of source code of `paru` from AUR:

  ```bash
  git clone https://aur.archlinux.org/paru.git
  cd paru
  ```

  Now compile and install `paru`:

  ```bash
  makepkg -si
  ```

  > `makepkg` is a programme developed by Arch Linux authors to build and pack binary packages. It's something like Gentoo's `portage` and FreeBSD's `ports`.

## Taking snapshots to back up the system with `snapper`

**Install `snapper` and utilities**

Run the following command to proceed:

```bash
pacman -S snapper snap-pac btrfs-assistant
```

> `snapper` was developed by Arvin Schnell from openSUSE, it can be used to manage btrfs snapshots and LVM Thin-provisioned volume snapshots. By taking and comparing snapshots, you can roll back to the state you desire. Also, it supports automatic snapshots creation in timeline.

> `snap-pac` is a script for `pacman` that runs before and after transactions to call `snapper` to take a snapshot.

**Configure `snapper` to take snapshots for `/`**

Execute `btrfs-assistant` to configure

> `btrfs-assistant` is a tool with intuitive GUI to manipulate `snapper` configuration and btrfs subvolumes.

- Create a configuration for `/` in *Snapper Settings*

- One-click to enable `systemd` services for timeline automatic snapshot taking

- Modify *Timeline* settings and apply

Setup `snap-pac` as a `pacman` hook

The configuration file for `snap-pac` is `/etc/snap-pac.ini`

Execute this to modify its settings:

```bash
nano /etc/snap-pac.ini
```

Please DO read the self-explanatory configuration file through and change whatever you need!

## Setting up a firewall

`firewalld` is a dynamic firewall implementation with a feature-rich GUI frontend that supports applying administration templates for interfaces and network connections via "zones". Runtime and permanent configurations are isolated for enhanced security and flexibility.

**Install `firewalld`**

Execute this command to install:

```bash
pacman -S firewalld
```

**Enable the service**

```bash
systemctl enable --now firewalld
```

Now you can turn to *System Settings -> Wi-Fi & Networking -> Firewall* to access `firewalld` settings.

## Enabling `apparmor`

`apparmor` is a Linux security module that does almost the same thing as `SELinux` that supervises and prohibits processes behaviour.

**Install `apparmor`**

Proceed to install with this command:

```bash
pacman -S apparmor
```

**Enable the service**

Simply run this to configure `apparmor` to start at boot:

```bash
systemctl enable apparmor
```

**Communicate with the kernel to load security module**

- Using traditional initramfs and kernel images
  - For `grub`
  
    Modify `/etc/default/grub`

    Locate the line:

    ```
    GRUB_CMDLINE_LINUX_DEFAULT=
    ```

    and append

    ```
    lsm=lockdown,landlock,yama,integrity,apparmor,bpf
    ```

    (DO NOT change the sequence!)

  - For `systemd-boot`

    Edit `/<ESP>/loader/entries/<your-entry>`

    Add this after `option`

    ```
    lsm=lockdown,landlock,yama,integrity,apparmor,bpf
    ```

    (DO NOT change the sequence!)

- Using UKI

  For EFI executable UKIs created by `mkinitcpio`

  Create `/etc/cmdline.d/apparmor.conf` and write:

  ```
  lsm=lockdown,landlock,yama,integrity,apparmor,bpf
  ```

  (DO NOT change the sequence!)

## Xorg process having NVIDIA GPU engaged on laptops

If you do want to have it runs on Intel GPU only to completely shutdown your NVIDIA GPU to conserve power, add this to `/etc/X11/xorg.conf.d/20-force-intel.conf`:

```
Section "ServerFlags"
  Option "AutoAddGPU" "off"
EndSection
```

Then save your works and reset the laptop to apply changes.


**This guide is for referrence only.**

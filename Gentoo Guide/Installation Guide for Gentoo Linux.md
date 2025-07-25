# Installation Guide for Gentoo Linux

> This passage only applies to users who have experience using GNU/Linux and are moving forward to perform a simple installation of Gentoo Linux, so articulation limited.

> Anyway, "copy - paste - run" is not my intention to compile this article, but to give readers a glimpse to the installing procedure of Gentoo Linux thus they can apply what they have learned here to other desktop environment and file system table configurations.

> Gentoo Linux is based on source code, so it's compatible the most hardware as long as they can execute a C/C++ toolchain. Here we'll take amd64 (x86_64) with UEFI BIOS as an example.

## 1. Archive and burn the installation media

Head to the [Gentoo Linux download page](https://www.gentoo.org/downloads/) and download the latest installation ISO image.

Burn the ISO image to a USB flash drive or DVD. If you are using a USB flash drive, you can use tools like `dd` on Linux or Rufus on Windows to create a bootable USB. [Etcher](https://etcher.balena.io/) is cross-platform and user-friendly for this purpose.

## 2. Manipulate the UEFI BIOS settings

> What we're trying to imply here is a *Live ISO* environment that allows you to install Gentoo Linux. Live ISOs ditributed by Gentoo Linux is not a must. If you want to configure your installation environment, connecting to the Internet via an education network plan for example, it's recommended to use EndeavourOS Live ISO or any others that you familiar with.

> Please note that Gentoo Linux provides Live ISOs with `OpenRC` init, use it if you like.

## 3. Prepare for installation

### Prepare you disk

#### Alternate your disk partition table

> The example for this step is based on a GPT partition table, which is recommended for UEFI systems. If you need MBR and GPT dual booting, you can refer to related articles.

You should have at least two partitions: one for the root filesystem and one for the EFI System Partition (ESP). The ESP is required for UEFI booting.

> To make `/home` `/usr` `/opt` independent partitions, you can create additional partitions as needed.

> Consider using subvolumes with btrfs is you are to create storage pools across multiple disks.

- EFI System Partition (ESP) will need at least 500 MB

  Two options for using the same and only ESP on a single disk:

  > If there's not only one GNU/Linux installation on the single disk, let's take Windows as an example, please be cautious to proceed!

  | Mount Point | Pros | Cons |
  |-------------|------|------|
  | `/boot` | Executable EFI images and kernel images reside in the same partion for compatibility and convenient maintainance | May interfere with other GNU/Linux installations, especially if they use different bootloaders and takes more space on disk |
  | `/efi` | Seperate kernel images and files crutial for UEFI booting, avoiding interference with other GNU/Linux installations | *It just works* |

  > According to [Gentoo Wiki](https://wiki.gentoo.org/wiki/EFI_System_Partition#Optional:_autofs), it's not recommended to mount your ESP to `/boot/efi`.

  > If you are to create different ESPs for different operating systems, you can do both as illustrated above.

- Root file system partition can be adjusted according to your needs, but at least 40 GB is recommended.

- Create `swap` according to your needs, but at least 2 GB is recommended. If you have enough RAM (64 GB and above), you can skip this step.

  Recommendations in the Gentoo Handbook:

  | RAM size | With suspend support | With hibernation support |
  | - | - | - |
  | 2 GB or less | 2*RAM | 3*RAM |
  | 2 to 8 GB | RAM amount | 2*RAM |
  | 8 to 64 GB | 8 GB minimum, 16 GB maximum | 1.5*RAM |
  | 64 GB or greater | 8 GB minimum | Hibernation **not RECOMMENDED**! |

#### Create file systems

- For ESP, please use FAT32 file system.

- For root file system, it's recommended to use btrfs and seperate `/var/log` as an independent subvolume.
  
  To spilt up further, you can create subvolumes for `/var/cache` and `/var/tmp` as well, where Gentoo portage stores its temporary files and cache.

  > If you have enough RAM, you can use tmpfs for `/var/tmp/portage` to speed up the installation process.
  >
  > Append the following line to `/etc/fstab` to mount it automatically:
  >
  > ```
  > tmpfs   /var/tmp/portage    tmpfs   size=<insert value>G,uid=portage,gid=portage,mode=775    0 0
  > ```

- As for other independent partitions, you can use F2FS on SSDs and XFS on HDDs, or any other file systems you prefer.

### Connect to Internet and update the system clock

This is for archiving and extracting the stage3 tarball.

#### System clock update for `systemd` users

Execute the following command to check the system clock:

```bash
timedatectl status
```

Execute the following command to set the system clock to list supported time zones:

```bash
timedatectl list-timezones
```

> Hints: You can use `grep` to filter the output, for example, `timedatectl list-timezones | grep Asia`.

Execute the following command to set the system clock to your desired time zone:

```bash
timedatectl set-timezone <Your_Time_Zone>
```

For example, if you are in Hong Kong, you can set it to:

```bash
timedatectl set-timezone Asia/Hong_Kong
```

Then, you can manually restart `systemd-timesyncd` to synchronize the system clock:

```bash
systemctl restart systemd-timesyncd
```

#### System clock update for `OpenRC` users

Execute the following command to set the system clock to your desired time zone:

```bash
ln -sf /usr/share/zoneinfo/<Your_Time_Zone> /etc/localtime
```

Restart your time synchronisation service if needed, `chrony` for example:

```bash
rc-service chronyd restart
```

## Get started with installing Gentoo Linux

### Install the stage3 tarball

> stage3 tarball is a pre-compiled base system that you can use to start building your Gentoo installation, it contains the essential files and directories needed to run a basic Gentoo system.

> Be cautious to download the correct stage3 tarball for your architecture and desired profile, especially for init systems.

- Mount partitions

  1. Create the mounting point in `/mnt/gentoo`

  2. Mount the root file system partition or subvolume to `/mnt/gentoo`, and create `/mnt/gentoo/efi` `/mnt/gentoo/home` etc. to mount other partitions.

  3. Mount all partitions or subvolumes to their respective mount points.

- Install the stage3 tarball

  1. Change your directory to `/mnt/gentoo`.

  2. Download the stage3 tarball via `links` or `wget` to download it directly.

For example, to download the latest stage3 tarball for amd64 architecture:

```bash
links https://mirrors.cernet.edu.cn/gentoo/releases/amd64/autobuilds/
```

- Navigate to choose a folder:

  For `systemd` users, you can download tarballs reside in `current-stage3-amd64-desktop-systemd`.

  For `OpenRC` users, you can download tarballs reside in `current-stage3-amd64-desktop-openrc`.

- After archiving the tarball, you can extract it by executing the following command in `/mnt/gentoo`:

```bash
tar xpf stage3-amd64-desktop-<init 程式>-<時間戳>.tar.xz --xattrs-include='*.*' --numeric-owner
```

### Configure `portage`

> `portage` is the package management system used by Gentoo Linux, the heart of Gentoo's flexibility and customisation.

#### Manipulating `make.conf`

> Consult https://wiki.gentoo.org/wiki//etc/portage/make.conf for detailed configuration options.

By executing the following command, you can edit the `make.conf` file:

```bash
nano /mnt/gentoo/etc/portage/make.conf
```

1. Set compiler flags

Look for 

```
COMMON_FLAGS="-O2 -pipe"
```

and change it to:

```
COMMON_FLAGS="-march=x86-64 -O2 -pipe"
```

> If your CPU support `AVX2`, you can consider using `-march=x86-64-v3`, and `AMX` or `AVX512` for `-march=x86-64-v4`.

> To use native optimisations detected by compiler, you can use `-march=native`, but this may cause compatibility issues with other systems that have different micro architecture.

> According to Gentoo Handbook, `-O2` is a good balance between performance and stability, but you can experiment with other optimisation levels like `-O3` if you are comfortable with it, but Gentoo indicates that if *may cause problems*.

2. Set threads and jobs to be used by `portage`

For example, we have 12 threads and 32 GB or above of RAM available, we can set:

```
MAKEOPTS="-j12 -l12"
```

Simply compare half the number of threads with half the size of RAM and take the smaller one.

3. Set the priority of the `portage` process and paralled jobs

Append the following line to the `make.conf` file to set the idle scheduling policy for `portage` to avoid hogging CPU resources:

```
PORTAGE_SCHEDULING_POLICY="idle"
```

and this for 4 parallel jobs:

```
EMERGE_DEFAULT_OPTS="--jobs 4"
```

4. Set default Gentoo repository mirrors

You can set the default Gentoo repository mirrors by appending the following line to the `make.conf` file if you are in China:

```
GENTOO_MIRRORS="https://mirrors.tuna.tsinghua.edu.cn/gentoo"
```

> You can also use other mirrors like `https://mirrors.cernet.edu.cn/gentoo` or `https://mirrors.aliyun.com/gentoo` according to your location.

5. Set graphics cards

`portage` can automatically pull the correct drivers for your graphics card, so you can set it via `VIDEO_CARDS` variable.

- For 4th Gen and later Intel graphics, you can set:

  ```
  VIDEO_CARDS="intel"
  ```

- For NVIDIA graphics with proprietary drivers, you can set:

  ```
  VIDEO_CARDS="nvidia"
  ```

- For AMD Radeon graphics, you can set:

  ```
  VIDEO_CARDS="amdgpu radeonsi"
  ```

- If you are using Gentoo in a QEMU/KVM client, you can set:

  ```
  VIDEO_CARDS="virtgl"
  ```

Please be aware that you can set multiple values for `VIDEO_CARDS` by separating them with spaces.

> Here is an example of `VIDEO_CARDS` settings:
>
> A laptop with Radeon Integrated GPU and NVIDIA discrete GPU, you can set:
>
> ```
> VIDEO_CARDS="amdgpu radeonsi nvidia"
> ```

6. Set default USE flags

Append the following line to the `make.conf` file to set default USE flags:

```
USE=""
```

For example, we are to use X Server, Wayland, KDE Plasma, Pipewire Sound Server, fcitx, and always use ditributed binaries and Gentoo distro kernel, we can set:

```
USE="X wayland kde qt pipewire pulseaudio fcitx bindist dist-kernel -gnome"
```

7. Set accepted licenses

Recommended to set the following line for global use, and create `/etc/portage/package.license/` directory to set exceptions for certain packages:

```
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"
```

#### Configure the `portage` tree

1. Create the `/etc/portage/repos.conf` directory if it does not exist for repositories configuration:

```bash
mkdir /mnt/gentoo/etc/portage/repos.conf
```

This directory is used to store the configuration files for different repositories, including those set by `eselect`.

1. Copy the default repository configuration file for the official repo (gentoo) to the `/etc/portage/repos.conf` directory:

```bash
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

You can change the repo URL in the `gentoo.conf` file if you want to use a different mirror.

For example, change this line

```
sync-uri = rsync://rsync.gentoo.org/gentoo-portage
```

to 

```
sync-uri = rsync://mirrors.cernet.edu.cn/gentoo-portage
```

#### Add binary packages repository for `portage`

Edit `/mnt/gentoo/etc/portage/binrepos.conf/gentoobinhost.conf` and change this line

```
sync-uri = https://distfiles.gentoo.org/releases/amd64/binpackages/23.0/x86-64/
```

to

```
sync-uri = http://mirrors.sustech.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64/
```

> Again, if you have a CPU that supports `AVX2`, you can change the URL to `https://mirrors.sustech.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64-v3/`.

### Enter the chroot environment to continue the installation

#### `chroot` into Gentoo Linux

1. Copy DNS resolution information to the new environment:

> This step is necessary to ensure that the new environment can resolve domain names to access the Internet.

```bash
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

2. Mount necessary filesystems:

- If you are using Arch Linux Live ISO or its derivatives, or you have `arch-chroot` installed, you can execute the following command to mount the necessary filesystems:

  ```bash
  arch-chroot /mnt/gentoo
  ```

  then you can change your shell prompt to indicate that you are in the chroot environment:

  ```bash
  export PS1="(chroot) ${PS1}"
  ```

- If you don't have `arch-chroot` installed, you can execute the following commands to mount the necessary filesystems:

  ```bash
  mount --types proc /proc /mnt/gentoo/proc
  mount --rbind /sys /mnt/gentoo/sys
 mount --make-rslave /mnt/gentoo/sys
  mount --rbind /dev /mnt/gentoo/dev
  mount --make-rslave /mnt/gentoo/dev
  mount --bind /run /mnt/gentoo/run
  mount --make-slave /mnt/gentoo/run 
  ```

  > This is essential for the chroot environment to function properly, as pseudo file systems like `/proc` `/sys` `/dev` `/run` allows the new environment to access the system's resources.

  Then you can enter the chroot environment by executing:

  ```bash
  chroot /mnt/gentoo /bin/bash
  source /etc/profile
  export PS1="(chroot) ${PS1}"
  ```

#### Synchronise the `portage` tree

Execute the following command to synchronise the `portage` tree to the latest version:

```bash
emerge-webrsync
```

Then get keyrings for binary packages:

```bash
getuto
```

#### Choose a profile

> A profile is a set of default USE flags, package sets, and other configuration options that define the behaviour of the system. You can choose a profile that suits your needs.

Execute the following command to list available profiles:

```bash
eselect profile list
```

> Filter the output by using `grep` to find the profile you want, for example, `eselect profile list | grep plasma`.

Then execute the following command to set the desired profile:

```bash
eselect profile set <ID>
```

#### Set `CPU_FLAGS`

Gentoo provides a tool to help you set the correct `CPU_FLAGS` for your system. 

```bash
emerge --ask --oneshot app-portage/cpuid2cpuflags
```

> To shorten the command, you can use `emerge -a1 cpuid2cpuflags`.

> `--oneshot` means that the package will not be added to the world file, and `--ask` will prompt you for confirmation before proceeding.

Execute the following command to generate and import the `CPU_FLAGS`:

```bash
echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```

#### Perform a global system update (Optional)

> This step is optional, because the stage3 tarball already contains a basic system. However, if you run into trouble installing packages, you can perform a global system update.

Update the system by executing the following command:

```bash
emerge --ask --verbose --update --deep --newuse @world
```

> To shorten the command, you can use `emerge -avuDN @world`.

> Consider using `--getbinpkg` or `-g` to get binary packages, and that's `emerge -agvuDN @world`.

#### Set system locale

> It's similar to setting the system locale in Arch Linux, but Gentoo provides a more flexible way to do it.

Edit the `/etc/locale.gen` file to uncomment or append your desired locale(s).

Execute the following command to generate the locale(s):

```bash 
locale-gen
```

List the default locale by executing the following command:

```bash
eselect locale list
```

Then set the default locale by executing the following command:

```bash
eselect locale set <ID>
```

Finally, you can update current environment by executing:

```bash
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

#### Set system time zone

> It's also familiar to setting the system time zone in Arch Linux.

Execute the following command to set the system time zone:

```bash
ln -sf /usr/share/zoneinfo/<地區>/<城市> /etc/localtime
```

#### Install the Gentoo Linux distro kernel and firmware

- Take a system with `systemd` init that boots from a Unified Kernel Image (UKI) of `gentoo-kernel-bin` created by `dracut` with `systemd-boot` as an example:

  1. Add support for `dracut`, `systemd-boot` and UKI to `installkernel`:

     Create a file named `/etc/portage/package.use/installkernel` if it does not exist:

     ```bash
     nano /etc/portage/package.use/installkernel
     ```

     Append the following line to the file:
    
     ```
     sys-kernel/installkernel dracut uki systemd-boot systemd
     ```

  2. Add support for `systemd-boot` to `systemd`:

     Create a file named `/etc/portage/package.use/systemd` if it does not exist:

     ```bash
     nano /etc/portage/package.use/systemd
     ```

     Append the following line to the file:

     ```
     sys-apps/systemd boot
     ```

  3. Install kernel and firmware:

     Execute the following command to install the kernel and firmware:

     ```bash
     emerge -a gentoo-kernel-bin linux-firmware
     ```

     > Use `gentoo-kernel-bin` to install the latest stable pre-compiled kernel to shorten time. 

     > Be sure to install `intel-microcode` if you have an Intel CPU, while AMD's is included in `linux-firmware`.

  4. Change `dracut` configuration:

        Edit the `/etc/dracut.conf` file to configure
    
        ```bash
        nano /etc/dracut.conf
        ```
    
        Append the following lines to the file:

        ```
        uefi="yes"  # To support UEFI booting, necessary for UKIs
        hostonly="yes"  # To create a minimal initramfs without unnecessary modules
        kernel_cmdline="<command line options>"  # Typically specifies root block device and file system type
        ```

        > Alternatively, you can use snippet files in `/etc/dracut.conf.d/` to configure `dracut`.

        > For a system with btrfs, you can set the `kernel_cmdline` to:
        >
        > `root=UUID=<your-root-uuid>`
        >
        > `rootflags=subvol=<your-subvolume>`
        >
        > `rootfstype=btrfs`
        >
        > `rw`
        >
        > Seperate them with spaces, and replace `<your-root-uuid>` and `<your-subvolume>` with your actual root UUID and subvolume name.

        **Be aware that DO NOT USE UUID to specify your root device if resides in a LVM logical volume**

        **It's applicable to use `root=/dev/mapper/<your-vg-name>/<your-lv-name>`**

- For a system with `OpenRC` init that boots from a Unified Kernel Image (UKI) of `gentoo-kernel-bin` created by `dracut` with `systemd-boot` as the bootloader:

  > Step 2 above needs your attention

  2. Add support for `systemd-boot` `udev` `kernel-install` to `sys-apps/systemd-utils`:

     Create a file named `/etc/portage/package.use/systemd-utils` if it does not exist:

     ```bash
     nano /etc/portage/package.use/systemd-utils
     ```

     Append the following line to the file:

     ```
     sys-apps/systemd-utils boot kernel-install udev
     ```

#### Alternate your `/etc/fstab`

- If you are using Arch Linux Live ISO or its derivatives, you can execute the following command to generate a default `/etc/fstab` file:

```bash
genfstab -U /mnt/gentoo > /mnt/gentoo/etc/fstab
```

Then back to the chroot environment to manipulate the `/etc/fstab` file as needed.

- If you don't have `genfstab` installed, you can create the `/etc/fstab` file manually:

> A system with its ESP mounted to `/efi` and a btrfs root file system and a seperate `/home` with f2fs should be like this:

  | UUID | Mount point | Filesystem type | Options | Dump | Filesystem check |
  | - | - | - | - | - | - |
  | UUID=<your-root-UUID> | / | btrfs | defaults,subvol=<your-root-subvolume> | 0 | 1 |
  | UUID=<your-root-UUID> | /var/log | btrfs | defaults,subvol=<your-log-subvolume> | 0 | 2 |
  | UUID=<your-esp-UUID> | /efi | vfat | defaults | 0 | 2 |
  | UUID=<your-home-UUID> | /home | f2fs | defaults | 0 | 2 |

  **Please note that DO NOT specify mounting points for LVM logical volumes with UUID**

  **Instead, use `/dev/mapper/<your-vg-name>-<your-lv-name>`**

#### Set the root password and add a user

1. Set the root password by executing the following command as root:

```bash
passwd
```

Execute the command to add a user and append the user to the `wheel` privileged group:

```bash
useradd -m -G wheel <username>
```

Set the password for the user by executing the following command:

```bash
passwd <username>
```

2. Configure `sudo` to allow the user to execute commands as root:

Execute the following command to install `sudo`:

```bash
emerge -a sudo
```

Edit the `/etc/sudoers` file and append user name.

#### Setup your init

- For `systemd` users:

  Execute the following command and follow the prompt to perform the initial setup:

  ```bash
  systemd-machine-id-setup
  systemd-firstboot --prompt
  ```

  Load preset configurations for all systemd services:

  ```bash
  systemctl preset-all --preset-mode=enable-only
  systemctl preset-all
  ```

- For `OpenRC` users:

  Create `/etc/hostname` file and set your hostname

  Configure as needed in `/etc/rc.conf` `/etc/conf.d/keymaps` `/etc/conf.d/hwclock`

  > It's recommended to set `rc_logger="YES"` in `/etc/rc.conf`.

#### Configure network interfaces

> We'll use `networkmanager` to manage network interfaces, which is a user-friendly tool for network configuration.

Execute the following command to install `networkmanager`:

```bash
emerge -ag networkmanager
```

Then enable its service to start at boot:

- For `systemd` users:

  ```bash
  systemctl enable NetworkManager
  ```

- For `OpenRC` users:

  ```bash
  rc-update add NetworkManager default
  ```

#### Install utilities

1. Disk scheduling optimisation

   Install `io-scheduler-udev-rules`

2. File indexing

   Install `mlocate`

3. Command completion for `bash`

   Install `bash-completion`

4. File system utilities

   Install what you need

   > For instance, `btrfs-progs` for btrfs file system, `f2fs-tools` for f2fs file system, `dosfstools` for FAT32 file system, and `lvm2[lvm]` for LVM, etc.
   
5. WLAN utilities

   Install `wpa_supplicant` and `iw` for wireless network management.

   > For better compatibility, you can install `wpa_supplicant` with `tkip` USE flag enabled.

- For `OpenRC` users, you will need the following tools:

  `app-admin/sysklogd`
  `sys-process/cronie`
  `net-misc/chrony`

  Enable them respectively:

  ```bash
  rc-update add sysklogd default
  rc-update add cronie default 
  rc-update add chronyd default
  ```

#### Install the bootloader

> We are to use `system-boot` as the bootloader.

Execute the following command to install `system-boot`:

```bash
bootctl install
```

To check if the installation is successful and your boot entries, you can execute:

```bash
bootctl list
```

#### Reboot into your new Gentoo Linux installation

Exit the chroot environment by executing `exit` or pressing `Ctrl+D`.

Back to the Live ISO environment, unmount the mounted filesystems:

```bash
cd
umount -Rl /mnt/gentoo
```

Then reboot your system and choose Linux Boot Manager or UEFI OS from the boot menu to enter `systemd-boot` and boot into your new Gentoo Linux installation.

### Setup GUI for your new Gentoo Linux installation

> We will use KDE Plasma as an example, but you can choose any desktop environment you prefer.

- Install X Server and NVIDIA Proprietary drivers:

  ```bash 
  emerge -a xorg-server nvidia-drivers
  ```

  > ATTENTION: If your NVIDIA GPU is the only GPU to output video, please add this configuration to `/etc/X11/xorg.conf.d/nvidia.conf`:

  >  ```
  >  Section "Device"
  >     Identifier  "nvidia"
  >     Driver      "nvidia"
  >  EndSection
  >  ```

- Install KDE Plasma:

  ```bash
  emerge -ag plasma-meta sddm kde-apps/dolphin ark konsole
  ```

- Enable SDDM to start at boot:

  - For `systemd` users:

    ```bash
    systemctl enable sddm
    ```

  - For `OpenRC` users:

    Edit `/etc/conf.d/display-manager` to specify SDDM

    ```
    ......
    DISPLAYMANAGER="sddm"
    ```

    Then enable `display-manager` and `elogind` to start at boot:

    ```bash
    rc-update add display-manager default
    rc-update add elogind boot
    ```

- Enable Pipewire sound server:

  Append a user to the `pipewire` group:

  ```bash
  usermod -aG pipewire <username>
  ```

  - For `systemd` users, execute the following command as a regular user:

    ```bash
    systemctl --user enable pipewire
    systemctl --user enable pipewire-pulse
    systemctl --user enable wireplumber
    ```
  
  - For `OpenRC` users, please double-check if there is a `/bin/gentoo-pipewire-launcher` exist.

    For desktop environments that honour auto-start files, such as KDE Plasma, you don't need any extra steps.

    For others, please refer to https://wiki.gentoo.org/wiki/PipeWire#OpenRC

- Allow a user to use removable media and graphics hardware acceleration:

  ```bash
  usermod -aG plugdev <username>
  usermod -aG video <username>
  ```


**Now, you have accomplished the basic installation of Gentoo Linux**
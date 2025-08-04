# Gentoo on LUKS

## Preface

Articles that talk about encrypting your root file system via the LUKS (Linux Unified Key Setup) as the bottom layer container to harden your GNU/Linux are of abundence, but they are just a bit confusing as people trying to tell you the whole story thus making them lengthy.

By encrypting your root file system, your data will not be accessed easily, either for the others or yourself. Doing so will surely enhance security.

Just like the *Installation Guide for Gentoo Linux*, I will try to keep the artical simple and focus on practising.

**To align with the *Guide*, I will take a PC with amd64 architecture and UEFI BIOS as an example, and it is designed to boots a Unified Kernel Image created by `dracut` with `systemd-boot`.**

## Prepare your disk

The disk should contain at least two partitions, one for your root file system and the other fo EFI System Partition.

> Please be aware that the ESP CANNOT be encrypted otherwise the system is unbootable.

> Whether to share the ESP with other operating systems and which mounting point to choose are discussed in the [Installation Guide](Gentoo%20Guide/Installation%20Guide%20for%20Gentoo%20Linux.md).

Here I will seperate the ESP and mount it to `/efi`.

### Modify the partition table

1. Check your disk

To list all local disks, run:

```bash
lsblk
```

> Mark down the path to your desired disk, such as `/dev/sda` and `/dev/nvme0n1`.

2. Modify your partition table with `cfdisk` 

> Encrypting a partition will not affect data that reside in unencrypted partitions, if you are to encrypt the whole disk, skip this step.

Execute the following command to start editing your partition table:

```bash
cfdisk /dev/<your disk>
```

- Create a partition with 1 GiB and change its type to EFI System Partition

- Create a partition for the LUKS container which your root file system will reside in, change its type to help you distinguish between others

### Create a LUKS container

Check your disk again and mark your partitions down via `lsblk`.

- Creating a regular LUKS container

  Execute the following command to create a container:

  ```bash
  cryptsetup luksFormat /dev/<partition>
  ```

  > You can add the `--key-size 512` parameter for a bit additional security.

To shorten the passage, I will skip advanced ways to create a LUKS container. If you do interest in, please visit [related articles](https://wiki.gentoo.org/wiki/Full_Disk_Encryption_From_Scratch).

- **Backup your header**

  > DO NOT skip this step as your passcode is only for unlocking the LUKS container and if the header was damaged somehow, all data will be irrecoverable!

  Execute the following command to backup your header:

  ```bash
  cryptsetup luksHeaderBackup /dev/<partition> --header-backup-file <path to your backup>
  ```

### Make file systems

1. Create your ESP

Execute this to make an ESP:

```bash
mkfs.vfat -F32 -n <file system label> /dev/<partition>
```

2. Unlock the LUKS container

Execute this to unlock the LUKS container:

```bash
cryptsetup open /dev/<partition> <name to be mapped>
```

> For instance, `cryptsetup open /dev/nvme0n1p2 rootfs` will unlock `/dev/nvme0n1p2` and map it under `/dev/mapper/rootfs`.

1. Create file systems inside the LUKS container

> You can use any file systems in the LUKS container, even LVM. Adjust to match you demand.

- To avoid additional complexity in mapping, you can make a file system directly

  > You can use any file systems if you like, but we will go with btrfs here.

  Run this command to create a btrfs file system:

  ```bash
  mkfs.btrfs /dev/mapper/<mapped name> -L <file system label>
  ```

  Mount the btrfs container and create subvolumes:

  ```bash
  mount /dev/mapper/<mapped name> /mnt/gentoo
  cd /mnt/gentoo
  btrfs subvolume create @
  btrfs subvolume create @log
  btrfs subvolume create @tmp
  btrfs subvolume create @cache
  btrfs subvolume create @home
  cd && umount -Rl /mnt/gentoo
  ```

- For flexibility in allocating disk space, you can create a LVM

  > Attention: This could lead to worse performance.

  Setup a LVM physical volume:

  ```bash
  pvcreate /dev/mapper/<mapped name>
  ```

  Create a LVM group:

  ```bash
  vgcreate <group name> /dev/mapper/<mapped name>
  ```

  > For example:
  >
  > ```bash
  > vgcreate sys /dev/mapper/rootfs
  > ```

  To create a LVM logical volume:

  ```bash
  lvcreate -L <capacity> <group name> -n <logical volume name>
  ```

  or

  ```bash
  lvcreate -l <percentage>FREE <group name> -n <logical volume name>
  ```

  > For instance:
  >
  > ```bash
  > lvcreate -L 40G sys -n root
  > lvcreate -l 100%FREE sys -n home
  > ```

  Then create file systems in logical volumes:

  ```bash
  mkfs.btrfs /dev/<group name>/<logical volume name> -L <file system label>
  ```

  To create subvolumes, please refer to the former section.

## Perform the installation

After mounting all file systems, you are good to proceed with the standard Gentoo Linux installation. All the steps that needs your attention will be implied below.

## Steps that different from standard installation

### File system utilities

On the basis of installing all the file system libraries you need, you must install `sys-fs/cryptsetup` as well.

> If you are using LVM, make sure you have `sys-fs/lvm2` with the `lvm` USE Flag.

### Configuring the init

- `OpenRC` users can skip this step.

- `systemd` users must add the `cryptsetup` USE Flag for `sys-apps/systemd`.

### Configuring `dracut`

- `dracut` modules

  Please add the following manually:

  ```bash
  add_dracutmodules+=" crypt dm rootfs-block "
  ```

  > Add `lvm` if you are using it.

- `dracut` kernel parameters

  Assign these at least:

  ```bash
  kernel_cmdline+=" root=UUID=<partition UUID> rd.luks.uuid=<LUKS container UUID> rootflags=<options for mounting root> rootfstype=<root file system> rw"
  ```

  > To use `discard` in LUKS, add `rd.luks.allow-discards` as kernel parameter.

  > To use `discard` in LVM, search and change the option to `issue_discards = 1` in `/etc/lvm/lvm.conf`.

  > If using LVM, add `rd.lvm.vg=<group name>` as well, and avoid specifying root with UUID, use `root=/dev/mapper/<group name>-<logical volume name>` instead.

Now, your Gentoo Linux should boot successfully after your rebuilding your UKI.

## (Optional) Unlock LUKS automatically with TPM on startup

> Alike to unlocking BitLocker volumes via TPM during system startup on Windows, the TPM will check UEFI settings and hardware for integrity. If modification detected, there will be a prompt for you to enter your passcode; otherwise, TPM will unlock LUKS automatically.

### Install `app-crypt/clevis` from `guru`

1. Enable the `guru` repository

   Install `app-eselect/eselect-repository` `dev-vcs/git` first:

   ```bash
   emerge -ag eselect-repository dev-vcs/git
   ```

   Enable the `guru` repository:

   ```bash
   eselect repository enable guru
   ```

   Synchronise local database:

   ```bash
   emerge --sync
   ```

2. Install `app-crypt/clevis`

   Execute this to install:

   ```bash
   emerge -ag app-crypt/clevis
   ```

### Add a TPM key for the LUKS container

1. Check the `keyslots` of a LUKS container

   Run this command and check the `keyslots` section:

   ```bash
   cryptsetup luksDump /dev/<LUKS partition>
   ```

2. Add a TPM key

   Execute this to add a key:

   ```bash
   clevis luks bind -d /dev/<LUKS partition> tpm2 '{"pcr_bank":"sha256","pcr_ids":"0,2,3,5,6,7"}'
   ```

   Check the `keyslots` again:

   ```bash
   cryptsetup luksDump /dev/<LUKS partition>
   ```

3. Rebuild the UKI

   `dracut` will pull in related modules automatically after installing `app-crypt/clevis`, manual intervention is not needed typically.

   Check the output if you can see:

   ```
   ...
   dracut: *** Including module: clevis ***
   dracut: *** Including module: clevis-pin-sss ***
   dracut: *** Including module: clevis-pin-tpm2 ***
   dracut: *** Including module: crypt ***
   dracut: *** Including module: dm ***
   ...
   ```

   If not, please add them into `add_dracutmodules+="  "`


**The End**
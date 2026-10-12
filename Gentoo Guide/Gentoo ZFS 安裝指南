# Gentoo ZFS 根目錄安裝

**特別注意：Gentoo Live GUI ISO 不提供 ZFS 支援，請使用 Gentoo Minimal Install ISO**

## 前言

ZFS 最初由 SUN Microsystems 開發，於 Sun Solaris 中使用，後來於 2005 年被開源，並於 2007 年被移植到 FreeBSD 中。2008 年，Brian Behlendorf 等人嘗試將 ZFS 移植到 GNU/Linux 作業系統中，但因授權條款不相容而無法併入 Linux 源碼

若閣下正閱讀本文，想必對 ZFS 有過了解並有意願嘗試，但務必注意 ZFS 對硬體的要求較高，I/O 效能與記憶體容量將會直接影響其作業效能

ZFS 有兩個重要概念：`zpool` 與 `dataset`，理解清楚將有助於閣下稍後的實作

`zpool` 可以理解為一個由虛擬磁碟組成的大容器，而 `dataset` 則為容器中的「檔案系統」

**為與安裝教程相契，本文將使用 `systemd-boot` 啟動由 `dracut` 建立的 Unified Kernel Images，且 `/efi` 獨立掛載，`/` 位於 ZFS 分割**

## 準備磁碟

磁碟至少包含兩個分割，分別用於挂載根目錄和 EFI System Partition

> 注意：EFI System Partition 不能納入 ZFS 分割，否則無法透過 `systemd-boot` 啟動

### 編輯分割表

1. 檢查磁碟

使用

```bash
lsblk
```

以列出本機磁碟

> 請記下磁碟檔案的位置，例如 `/dev/sda` 或 `/dev/nvme0n1`

2. 使用 `cfdisk` 編輯分割表

執行 

```bash
cfdisk /dev/<磁碟>
```

以編輯分割表

- 為 ESP 建立一個 1 GiB 的分割，並賦予 EFI System Partition 類型

- 為 ZFS 建立一個分割，並賦予 Linux filesystem 類型

### 建立 `zpool` 容器

**`zpool` 根據 `/etc/hostid` 辨識主機，建立容器前請先執行 `zgenhostid`**

> 貼士：使用 `lsblk` 再次檢查磁碟，記下磁碟分割

- 建立一般 `zpool` 儲存池

  執行

  ```bash
  zpool create -o ashift=12 -o autotrim=on -O acltype=posix -O xattr=sa -O normalization=formD -m none <zpool 名稱> /dev/<磁碟分割>
  ```

  以建立 `zpool` 儲存池

  > 例如：`zpool create -o ashift=12 -o autotrim=on -O acltype=posix -O xattr=sa -O normalization=formD -m none root_zpool /dev/nvme1n1p2`

  > `zpool` 支援相當多引數，此處僅對以上命令中的作說明：
  >  - `ashift=12`，手動設定磁區大小為 4096 bytes（適用於絕多數磁碟）
  >  - `autotrim=on`，啟用 TRIM 功能，於 SSD 上使用
  >  - `acltype=posix`，設定使用 POSIX 的存取控制清單格式
  >  - `xattr=sa`，使用系統屬性格式（通常啟用以最佳化效能）
  >  - `normalization=formD`，設定檔案名「正規化」，以免 NFS/SMB 共享時於其他作業系統中呈現亂碼
  >  - `-m`，設定掛載點，此處指派的是容器頂層，通常不掛載

  > 注意：意欲使用 LUKS 加密，請於磁碟分割首先建立 LUKS 容器，隨後解鎖容器並將其作為 `zpool` 的磁碟

### 建立檔案系統及 ZFS Dataset

1.  建立 ESP
  
    執行

    ```bash
    mkfs.vfat -F32 -n <檔案系統標簽> /dev/<磁碟分割>
    ```

    以建立 ESP，此分割將被掛載於 `/efi`

2.  建立 `Dataset`

    > 注意：建立 `zpool` 容器的同時，將會建立一個同名的頂層 `Dataset`，且無法刪除，除非刪除整個 `zpool`

    執行

    ```bash
    zfs create -o mountpoint=none <zpool 名稱>/<dataset 名稱>
    ```

    以建立子 `Dataset`，此次頂層 `Dataset` 將不會被掛載

    > 例如：`zfs create -o mountpoint=none root_zpool/gentoo`

    再次執行以建立下層 `Dataset` 以便管理樹狀根目錄

    ```bash
    zfs create -o mountpoint=/ -o canmount=noauto <zpool 名稱>/<次頂層 dataset 名稱>/<dataset 名稱>
    zfs create -o mountpoint=/home <zpool 名稱>/<次頂層 dataset 名稱>/<dataset 名稱>
    zfs create -o mountpoint=/var/log <zpool 名稱>/<次頂層 dataset 名稱>/<dataset 名稱>
    (以此類推)
    ```

    > 例如：
    >  ```bash
    >  zfs create -o mountpoint=/ -o canmount=noauto root_zpool/gentoo/rootfs`
    >  zfs create -o mountpoint=/home root_zpool/gentoo/home
    >  zfs create -o mountpoint=/var/log root_zpool/gentoo/log
    >  (以此類推)
    >  ```

    設定 `Dataset` 屬性，使根目錄作為 `bootfs`

    ```bash
    zpool set bootfs=<zpool 名稱>/<次頂層 dataset 名稱>/<根目錄 dataset 名稱> <zpool 名稱>
    ```

    > 例如：
    >
    >  ```bash
    >  zpool set bootfs=root_zpool/gentoo/rootfs root_zpool
    >  ```

## 執行安裝

  - 掛載 `Dataset`

    執行以下命令以重新匯入 `zpool`，並設定臨時根目錄為 `/mnt/gentoo` 並暫時不掛載：

    ```bash
    zpool export <zpool 名稱>
    zpool import <zpool 名稱> -R /mnt/gentoo -N
    ```

    由根目錄開始掛載檔案系統：

    ```bash
    zfs mount <zpool 名稱>/<次頂層 dataset 名稱>/<dataset 名稱>
    mount --mkdir /dev/<ESP 分割> /mnt/gentoo/efi
    ```

## 有別於常規安裝的步驟

### `chroot` 進入新系統之前

  為確保 DNS 解析正常，Gentoo Handbook 中要求將 `/etc/resolv.conf` 複製至新系統中

  同樣，為確保 `zpool` 容器正常運作，請將 `/etc/hostid` 複製至新系統中

### 檔案系統工具組

  請確保 `sys-fs/zfs` 已加入 `dist-kernel-cap` 與 `modules` 的 USE Flag，通常預設啟用

### `dracut` 設定

  為確保正常掛載，請於 `dracut` 配置檔案中加入：

  ```bash
  nofsck="yes"
  add_dracutmodules+=" zfs "
  kernel_cmdline+=" root=ZFS=AUTO rw zfs.force=1 "
  ```

  > 若有多個 `zpool` 設定過 `bootfs` 屬性，請將 `root=ZFS=AUTO` 改爲 `root=ZFS=<zpool 名稱>/<次頂層 dataset>/<根目錄 dataset 名稱`

  **為使設定生效，請重建 initramfs**

  > 注意：若 `zpool` 位於 LUKS 容器內，請同樣於 `add_dracutmodules` 中加入 `crypt`，並於 `kernel_cmdline` 中加入 `rd.luks.uuid=<LUKS 容器物理分割 UUID>`

### `init` 設定

  - `OpenRC` 使用者：

    請將 `zfs-import` 與 `zfs-mount` 加入 `boot` runlevel

    ```bash
    rc-update add zfs-import boot
    rc-update add zfs-mount boot
    ```

  - `systemd` 使用者：

    ```bash
    systemctl enable zfs-import.target
    systemctl enable zfs-import-cache
    systemctl enable zfs-mount
    ```

## 完成安裝

  至此，請根據 Gentoo Handbook 完成標準安裝，其餘流程並無二致
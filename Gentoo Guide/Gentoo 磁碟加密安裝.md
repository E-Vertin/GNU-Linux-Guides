# Gentoo 磁碟加密安裝

## 前言

有非常多系統加固 (Hardening) 相關文章提及“加密的根目錄”，即使用 LUKS (Linux Unified Key Setup) 作爲底層容器加密整個磁碟或一個分割區

此舉可使安全性更上一層樓，但不少文章講解得過於透徹，令人不能一目瞭然，因此本文將延續安裝教程的行文風格，强調實際操作的同時輔以理解

像安裝教程一樣，本文以使用 UEFI BIOS 的 amd64 架構的計算機爲例，執行一次帶有根目錄加密的 Gentoo Linux 安裝

**為與安裝教程相契，本文將使用 `systemd-boot` 啓動由 `dracut` 建立的 Unified Kernel Images**

## 準備磁碟

磁碟至少包含兩個分割，分別用於挂載根目錄和 EFI System Partition

> 注意：EFI System Partition 不能被加密，否則無法啓動

> 對於 ESP 挂載點與是否獨立，請參考[安裝指南](Gentoo%20Guide/Gentoo%20Linux%20安裝指南.md)

本文將使用獨立的 ESP 並挂載於 `/efi`

### 編輯分割表

1. 檢查磁碟

使用

```bash
lsblk
```

以列出本機磁碟

> 請記下磁碟檔案的位置，例如 `/dev/sda` 或 `/dev/nvme0n1`

2. 使用 `cfdisk` 編輯分割表

> 加密一個分割將不會影響其他未加密的分割内的資料，若需要加密整個磁碟，可以跳過此步驟

執行 

```bash
cfdisk /dev/<磁碟>
```

以編輯分割表

- 為 ESP 建立一個 1 GiB 的分割，並賦予 EFI System Partition 類型

- 為 LUKS 容器磁碟建立一個分割，根目錄將位於此處，可賦予類型以便區分

### 建立 LUKS 容器

使用 `lsblk` 再次檢查磁碟，記下磁碟分割

- 建立常規 LUKS 容器

  執行

  ```bash
  cryptsetup luksFormat /dev/<磁碟分割>
  ```

  以建立 LUKS 容器

  > 可以加入 `--key-size 512` 引數增加安全性

為縮短行文，此處將略去建立 LUKS 容器的進階做法，請參閲[相關文章](https://wiki.gentoo.org/wiki/Full_Disk_Encryption_From_Scratch)

- **建立 Header 備份**

  > 請不要略過此步驟！密碼只用於解鎖 LUKS 容器，若 Header 損毀，全部資料將丟失！

  執行

  ```bash
  cryptsetup luksHeaderBackup /dev/<磁碟分割> --header-backup-file <轉儲檔案路徑>
  ```

### 建立檔案系統

1. 建立 ESP

執行

```bash
mkfs.vfat -F32 -n <檔案系統標簽> /dev/<磁碟分割>
```


2. 解鎖 LUKS 容器

執行

```bash
cryptsetup open /dev/<磁碟分割> <映射名稱>
```

> 例如 `cryptsetup open /dev/nvme0n1p2 rootfs`，將會解鎖 `/dev/nvme0n1p2` 並映射至 `/dev/mapper/rootfs`

1. 於 LUKS 容器内建立檔案系統

> 可於 LUKS 容器中使用任何檔案系統，甚至是 LVM，請按需取用

- 為避免繁雜的映射，可以直接建立檔案系統

  > 可以使用任何檔案系統，但本文將使用 btrfs 及其子卷

  執行以建立 btrfs 檔案系統：

  ```bash
  mkfs.btrfs /dev/mapper/<映射名稱> -L <檔案系統標簽>
  ```

  挂載 btrfs 容器並建立子卷：

  ```bash
  mount /dev/mapper/<映射名稱> /mnt/gentoo
  cd /mnt/gentoo
  btrfs subvolume create @
  btrfs subvolume create @log
  btrfs subvolume create @tmp
  btrfs subvolume create @cache
  btrfs subvolume create @home
  cd && umount -Rl /mnt/gentoo
  ```

- 為靈活使用磁碟空間，可以於 LUKS 中建立 LVM

  > 注意：此舉可能導致效能下降

  建立 LVM 物理卷：

  ```bash
  pvcreate /dev/mapper/<映射名稱>
  ```

  建立 LVM 組：

  ```bash
  vgcreate <組名> /dev/mapper/<映射名稱>
  ```

  > 例如：
  > 
  > ```bash
  > vgcreate sys /dev/mapper/rootfs
  > ```

  建立 LVM 邏輯卷：

  ```bash
  lvcreate -L <具體容量> <組名> -n <邏輯卷名稱>
  ```

  或

  ```bash
  lvcreate -l <百分比>FREE <組名> -n <邏輯卷名稱>
  ```

  > 例如：
  >
  > ```bash
  > lvcreate -L 40G sys -n root
  > lvcreate -l 100%FREE sys -n home
  > ```

  隨後於 LVM 邏輯卷中建立檔案系統：

  ```bash
  mkfs.btrfs /dev/<組名>/<邏輯卷名稱> -L <檔案系統標簽>
  ```

  此處省略建立子卷，請參閲上文

## 執行安裝

挂載所有檔案系統並執行常規的 Gentoo Linux 安裝，需要特殊處理的步驟將在稍後説明

## 有別於常規安裝步驟

### 檔案系統工具組

在安裝所使用的檔案系統工具的基礎上，請確保已安裝 `sys-fs/cryptsetup`

> 若使用 LVM，則應加上 `sys-fs/lvm2` 並加入 `lvm` 的 USE Flag

### init 設定

- `OpenRC` 使用者可以略過此步驟

- `systemd` 使用者請為 `sys-apps/systemd` 加入 `cryptsetup` 的 USE Flag

### `dracut` 設定

- `dracut` 模塊

  請手動加入：

  ```bash
  add_dracutmodules+=" crypt dm rootfs-block "
  ```

  > 若使用 LVM，則再加入 `lvm`

- `dracut` 内核引數

  請至少指定：

  ```bash
  kernel_cmdline+=" root=UUID=<磁碟分割 UUID> rd.luks.uuid=<LUKS 容器分割 UUID> rootflags=<根目錄挂載選項> rootfstype=<根目錄檔案系統類型> rw "
  ```

  > 慾於 LUKS 中使用 `discard`，請加入 `rd.luks.allow-discards` 的內核引數

  > 慾於 LVM 中使用 `discard`，請於 `/etc/lvm/lvm.conf` 中尋找並改爲 `issue_discards = 1`

  > 若使用 LVM，則再加入 `rd.lvm.vg=<組名>`，且應避免使用 UUID 指派根目錄，而使用 `root=/dev/mapper/<組名>-<邏輯卷名稱>`

至此，重新建立 UKI 后 Gentoo Linux 應可正常啓動

### 爲 SSD 上的 LUKS 分割啓用 TRIM 及效能優化

> 注意：此步驟建議 SSD 使用者執行

1. 檢查 LUKS 分割功能標籤

   執行

   ```bash
   cryptsetup luksDump /dev/<LUKS 分割>
   ```

   並檢查 `Flags` 的内容

   > 若爲 `none` 則表示未啓用

2. 變更 LUKS 分割功能標籤

   執行

   ```bash
   cryptsetup --perf-no_read_workqueue --perf-no_write_workqueue --allow-discards --persistent refresh <device mapper 名稱>
   ```

   > 其中，`<device mapper 名稱>` 可於 `/dev/mapper` 中查詢，一般爲 `luks-<UUID>`

   再次執行 `cryptsetup luksDump /dev/<LUKS 分割>` 以檢查 `Flags` 的内容
 

## (可選) 使用 TPM 在開機時自動解鎖 LUKS

> 類似 Windows 在啓動中使用 TPM 自動解鎖 BitLocker，TPM 將在啓動時檢查本機 UEFI 設定與硬體變動。若 TPM 偵測到變更，則需要人為輸入密碼；若否，則自動解鎖 LUKS 容器

### 於 `guru` 倉庫安裝 `app-crypt/clevis`

1. 啓用 `guru` 倉庫

   首先安裝 `app-eselect/eselect-repository` `dev-vcs/git`：

   ```bash
   emerge -ag eselect-repository dev-vcs/git
   ```

   啓用 `guru` 倉庫：

   ```bash
   eselect repository enable guru
   ```

   同步化處理本機資料庫：

   ```bash
   emerge --sync
   ```

2. 安裝 `app-crypt/clevis`

   執行以安裝：

   ```bash
   emerge -ag app-crypt/clevis
   ```

### 為 LUKS 容器加入 TPM 密籥

1. 檢查 LUKS 容器的“鎖孔”

   執行

   ```bash
   cryptsetup luksDump /dev/<LUKS 分割>
   ```

   並檢查 `Keyslots` 的内容

2. 將密籥加入 TPM

   執行

   ```bash
   clevis luks bind -d /dev/<LUKS 分割> tpm2 '{"pcr_bank":"sha256","pcr_ids":"0,2,3,5,6,7"}'
   ```

   再次執行以檢查“鎖孔”

   ```bash
   cryptsetup luksDump /dev/<LUKS 分割>
   ```

3. 重新建立 UKI

   安裝 `app-crypt/clevis` 后 `dracut` 將會自動加入相關模塊，一般不需要人為指定

   檢查輸出結果

   ```
   ...
   dracut: *** Including module: clevis ***
   dracut: *** Including module: clevis-pin-sss ***
   dracut: *** Including module: clevis-pin-tpm2 ***
   dracut: *** Including module: crypt ***
   dracut: *** Including module: dm ***
   ...
   ```

   若沒有以上模塊，請手動加入至 `add_dracutmodules+="  "`


**至此，本文結束**
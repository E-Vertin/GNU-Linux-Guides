# Installation Guide for Gentoo Linux, ADVANCED USER ONLY

> Gentoo Linux 是基於源碼構建的作業系統，因此其對於絕大多數的硬體都有極好的相容性，此處以 amd64 (x86_64) 架構的計算機的安装为例。

## 1. 歸檔並燒錄映像檔

前往 [官方網站](https://www.gentoo.org/downloads/) 選擇鏡像並歸檔

使用 [Etcher](https://etcher.balena.io/) 燒錄映像檔至 USB Disk

## 2. 變更 UEFI BIOS 設定並載入 ISO

(TO BE LINKED to Basic)

> 此處要强調的是 “Live ISO” 環境，使用 Gentoo Linux Live ISO 并不是强制條件。慾使用 GUI 對安裝作業環境進行一定配置，如連線至校園網，請使用 EndeavourOS Live ISO 或任意一個您熟悉的 ISO。

> 請注意，Gentoo Linux 提供了使用 `OpenRC` init 程式的 Live ISO，請按需取用。

## 3. 執行安裝作業前的準備

### 準備磁碟

#### 調整磁碟分割表

磁碟至少包含兩個分割，分別掛載於 `/boot` 和 `/`

> 慾獨立 `/home` `/usr` `/opt` 分割，請按需調整

- `/boot` 用作 EFI system 分割時至少需要 1 GB

- `/` 可以按需調整，建議 50 GB

- `swap` 按需建立，但建議安裝的 RAM 较大 (64 GB 及以上) 的計算機不建立

此爲 Gentoo Handbook 中對 `swap` 的建議

| RAM size | With suspend support | With hibernation support |
| - | - | - |
| 2 GB or less | 2*RAM | 3*RAM |
| 2 to 8 GB | RAM amount | 2*RAM |
| 8 to 64 GB | 8 GB minimum, 16 GB maximum | 1.5*RAM |
| 64 GB or greater | 8 GB minimum | Hibernation **not RECOMMENDED**! |

#### 建立檔案系統

- 對於 `/boot` 作爲 EFI system 分割，請使用 FAT32

- 對於 `/`，建議使用 btrfs，並將 `/var/log` 獨立掛載於一個子卷

  慾進一步細分，還可以將 `/var/cache` 與 `/var/tmp` 獨立掛載

  > 若您有足夠的 RAM，可以考慮將 `/var/tmp/portage` 掛載爲 tmpfs
  >
  > 於 `/etc/fstab` 中加入
  >
  > ```
  > tmpfs           /var/tmp/portage                tmpfs   size=<大小>G,uid=portage,gid=portage,mode=775    0 0
  > ```

- 對於其他獨立的分割，建議使用 f2fs （機械硬碟可以使用 XFS）

### 連線至 Internet 並校準時間

此舉是爲歸檔並解壓 stage3 檔案做準備

(TO BE LINKED to Arch Guide)

## 開始 Gentoo Linux 的安裝作業

### 安裝 stage3 檔案

> stage3 檔案是 Gentoo Linux 的 “種子”，是基於某些作業系統設定檔自動構建的，幾乎包含了整個基礎作業系統的檔案。

> 在選擇 stage3 檔案時，必須考慮清楚慾使用的 init 程式是 `systemd` 還是 `OpenRC`，這會省下不必要的麻煩。

- 掛載磁碟分割

1. 建立 `/mnt/gentoo` 資料夾
   
2. 掛載作爲根目錄的磁碟分割或 btrfs 子卷至 `/mnt/gentoo` 並建立 `/mnt/gentoo/boot` `/mnt/gentoo/home` 以及其他您獨立掛載的作爲掛載點的資料夾
   
3. 掛載所有磁碟分割或子卷

- 安裝 stage3 檔案

1. 切換目錄至 `/mnt/gentoo`
   
2. 使用 `links` 或 `wget` 歸檔 stage3 檔案
   
例如

```sh
links https://mirrors.cernet.edu.cn/gentoo/releases/amd64/autobuilds/
```

選擇一個資料夾，例如 `current-stage3-amd64-desktop-systemd`

選擇歸檔 `stage3-amd64-desktop-systemd-<time>.tar.xz`

歸檔完成後，請按需執行檔案校驗

3. 解壓 stage3 檔案

於 `/mnt/gentoo` 目錄中執行

```sh
tar xpf stage3-amd64-desktop-systemd-<time>.tar.xz --xattrs-include='*.*' --numeric-owner
```

以解壓縮 stage3 檔案至根目錄

### 調整 `portage` 設定

> `portage` 是一個極其強大的包管理員，其靈活程度令人難以置信

#### 變更 `make.conf` 設定

執行

```sh
nano /mnt/gentoo/etc/portage/make.conf
```

1. 變更編譯器設定

找到行

```
COMMON_FLAGS="-O2 -pipe"
```

變更爲

```
COMMON_FLAGS="-march=x86-64 -O2 -pipe"
```

> 若您的 CPU 是 x86-64-v3 的微架構，可以考慮使用 `-march=x86-64-v3`；若支援 AVX512，可以考慮使用 `-march=x86-64-v4`

> 根據 Gentoo Handbook，`-O2` 是最合適的等級，`-O3` 可以稍微加快速度，但 “某些時候會出現問題”

2. 變更編譯時的執行緒數量及負載

例如本機的 CPU 有 12 條執行緒，RAM 的大小也大於或等於 24 GB，則

加入行

```
MAKEOPTS="-j12 -l12"
```

兩個數值均爲 CPU 的執行緒數量，或 RAM 的一半，在兩者中取最小的值。

3. 加入 Gentoo 倉庫鏡像

加入行

```
GENTOO_MIRRORS="https://mirrors.tuna.tsinghua.edu.cn/gentoo"
```

以使用清華 TUNA 鏡像伺服器

> 鏡像可以根據需求調整，例如廣東省的使用者可以使用 SUSTech CRA 鏡像
>
> ```
> GENTOO_MIRRORS="https://mirrors.sustech.edu.cn/gentoo"
> ```

4. 設定顯示卡
   
`portage` 會根據 `VIDEO_CARDS` 變數編譯安裝所需的包

例如，本機使用的 4th Gen 或更新的 Intel Integrated GPU 與“現代的” NVIDIA Discrete GPU 且慾使用專用驅動程式，則應該寫入

```
VIDEO_CARDS="intel nvidia"
```

5. 設定全域 `USE` 變數

加入行

```
USE=""
```

並與引號內加入

例如，本機慾使用 X Server, Wayland, KDE Plasma, KVM, Pipewire，且優先使用預先編譯的二進制包，對於需要編譯安裝的使用 `jumbo-build` 加速編譯

可以寫入

```
USE="X wayland kde qt kvm pipewire pulseaudio bindist dist-kernel -gnome"
```

6. 設定接受的條款

推薦對全域使用如下設定，並於 `portage/package.license/` 資料夾中對某些包進行例外設定

```
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"
```

#### 添加 `portage` 的倉庫

1. 建立倉庫設定檔案的資料夾

執行

```sh
mkdir /mnt/gentoo/etc/portage/repos.conf
```

此資料夾用於存放 gentoo 官方倉庫以及 `eselect` 中啓用的倉庫的設定檔案

2. 拷貝 gentoo 官方倉庫的設定檔案

執行

```sh
cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf
```

替換倉庫位址爲鏡像位址

使用 `nano` 編輯 `/mnt/gentoo/etc/portage/repos.conf/gentoo.conf`

將行

```
sync-uri = rsync://rsync.gentoo.org/gentoo-portage
```

替換爲

```
sync-uri = rsync://mirrors.cernet.edu.cn/gentoo-portage
```

#### 添加 `portage` 二進制包倉庫

使用 `nano` 編輯 `/mnt/gentoo/etc/portage/binrepos.conf/gentoobinhost.conf` 

將行

```
sync-uri = https://distfiles.gentoo.org/releases/amd64/binpackages/23.0/x86-64/
```

替換爲慾使用的鏡像，例如：

```
sync-uri = http://mirrors.sustech.edu.cn/gentoo/releases/amd64/binpackages/23.0/x86-64/
```

### 進入 Gentoo Linux 執行後續安裝及設定

#### 使用 `chroot` 進入 Gentoo Linux

1. 拷貝 DNS 解析檔案

執行

```sh
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

2. 掛載必要的檔案系統並進入 Gentoo Linux

- 若您使用的是 Arch Linux 及其衍生版的 Live ISO 且有 `arch-chroot` 程式，這一步非常簡單

  執行

  ```sh
  arch-chroot /mnt/gentoo
  ```

- 若您沒有該程式，請執行

  ```sh
  mount --types proc /proc /mnt/gentoo/proc
  mount --rbind /sys /mnt/gentoo/sys
  mount --make-rslave /mnt/gentoo/sys
  mount --rbind /dev /mnt/gentoo/dev
  mount --make-rslave /mnt/gentoo/dev
  mount --bind /run /mnt/gentoo/run
  mount --make-slave /mnt/gentoo/run 
  ```

  > 這一步非常重要，`/proc` `/sys` `/dev` `/run` 都是安裝作業系統必要的

  隨後，使用 `chroot` 進入 Gentoo Linux

  執行

  ```sh
  chroot /mnt/gentoo /bin/bash
  source /etc/profile
  export PS1="(chroot) ${PS1}"
  ```

#### 同步化處理 `portage` 資料庫並設定二進制包認證

執行

```sh
emerge-webrsync
```

以取得最新的資料庫

執行

```sh
getuto
```

以取得二進制包密鑰串

#### 選擇本機設定檔

> `eselect` 的設定檔賦予 `USE` `CFLAGS` 以及其他重要變數預設值，並屏蔽了某些包及某些包的版本，這是作爲整個作業系統的基石，因此請謹慎選擇

執行

```sh
eselect profile list
```

以檢視設定檔列表，同樣，可以輸出至 `grep` 進行篩選，例如：

執行

```sh
eselect profile list | grep plasma
```

以篩選使用 KDE Plasma 桌面的設定檔

慾選擇設定檔

執行

```sh
eselect profile set <ID>
```

#### 設定 `CPU_FLAGS`

Gentoo Linux 提供了一個自動化的程式用於加入 CPU 支援的命令集與演算法

執行

```sh
emerge --ask --oneshot app-portage/cpuid2cpuflags
```

> `--oneshot` 是表示不要將該包錄入 world 集合中，因爲該程式只需要使用一次

以安裝

再執行

```sh
echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```

以匯入所有 `CPU_FLAGS`

#### 更新 world 集合中的所有包

> 這一步並非必要，因爲安裝完成且進入作業系統後再更新亦無妨

執行

```sh
emerge --ask --verbose --update --deep --newuse @world
```

> 簡便表達，可以使用 `-avuDN`

即可

> 可以考慮加入 `--getbinpkg` 以使用二進制包；簡便表達爲 `-g`，也就是 `emerge -agvuDN @world`

#### 設定 locale

> 這一步與手動安裝 Arch Linux 的非常相似

使用 `nano` 於 `/etc/locale.gen` 反註釋或加入 locale

執行

```sh
locale-gen
```

以生成 locale

執行

```sh
eselect locale list
```

以檢視 locale 列表

使用

```sh
eselect locale set <ID>
```

以設定全域 locale

最後執行

```sh
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

以套用 locale 變更

#### 設定時區

> 這一步也與手動安裝 Arch Linux 的非常相似

執行

```sh
ln -sf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
```

以將 HKT (UTC+8) 設定爲本地時間

#### 安裝 Linux 內核及韌體

> 以將 `systemd` 作爲 init 程式並使用 `systemd-boot` 啓動 `dracut` 建立的 `gentoo-kernel-bin` 的 Unified Kernel Image 爲例

1. 加入 `installkernel` 包對 `dracut` `systemd-boot` 以及 UKI 的支援
   
   執行

   ```sh
   nano /etc/portage/package.use/installkernel
   ```

   並寫入

   ```
   sys-kernel/installkernel dracut uki systemd-boot systemd
   ```

2. 加入 `systemd` 包的 `systemd-boot` 支援

   執行

   ```sh
   nano /etc/portage/package.use/systemd
   ```

   並寫入

   ```
   sys-apps/systemd boot
   ```

3. 安裝 Linux 韌體及內核
   
   執行

   ```sh
   emerge -a linux-firmware gentoo-kernel-bin intel-microcode
   ```

   > 使用 `gentoo-kernel-bin` 是爲了節省安裝的時間，對於 Intel CPU ，必須於此安裝 `intel-microcode`，AMD CPU 的微碼已包含於 `linux-firmware` 中

4. 變更 `dracut` 設定
   
   使用 `nano` 編輯 `/etc/dracut.conf`

   寫入

   ```
   uefi="yes"
   hostonly="yes"
   kernel_cmdline="所需執行的命令"
   ```

   > 同樣的，`dracut` 也支援分類管理配置檔案，建立 `/etc/dracut.conf.d/` 資料夾並於此加入配置檔案

   > 對於一個使用 btrfs 檔案系統和 NVIDIA GPU 的計算機而言， `kernel_cmdline` 通常包括
   >
   > `root=UUID=<分割的UUID>`
   >
   > `rootflags=subvol=<根目錄所在子卷>`
   >
   > `rootfstype=btrfs`
   >
   > `rw`
   >
   > `nvidia_drm.modeset=1`
   >
   > 填入時請以空格分隔

#### 撰寫 `/etc/fstab`

- 若使用的是 Arch Linux 及其衍生版的 Live ISO，這一步也非常簡單

  在 Live ISO 環境中以 root 登入

  執行

  ```sh
  genfstab -U /mnt/gentoo > /mnt/gentoo/etc/fstab
  ```

  隨後使用 `nano` 按需求變更掛載設定

  (TO BE LINKED to Arch Manually Install)

- 若使用的 Live ISO 沒有 `genfstab`

  使用 `nano` 撰寫 `/etc/fstab`

  > 一個以 `/boot` 爲 EFI 分割，`/` 使用 btrfs 且獨立 `/home` 使用 f2fs 的設定應該是這樣的

  | UUID | Mount point | Filesystem type | Options | Dump | Filesystem check |
  | - | - | - | - | - | - |
  | UUID=<分割UUID> | / | btrfs | defaults,subvol=<根目錄所在子卷> | 0 | 1 |
  | UUID=<分割UUID> | /var/log | btrfs | defaults,subvol=<日誌目錄所在子卷> | 0 | 2 |
  | UUID=<分割UUID> | /boot | vfat | defaults | 0 | 2 |
  | UUID=<分割UUID> | /home | f2fs | defaults | 0 | 2 |

#### 設定 root 密碼及加入使用者賬戶

1. 設定密碼及加入使用者賬戶

執行

```sh
passwd
```

以設定 root 密碼

再執行

```sh
useradd -m -G wheel <使用者名稱>
```

以加入使用者賬戶並將其加入 wheel 羣組

執行

```sh
passwd <使用者名稱>
```

以設定使用者密碼

2. 設定 `sudo`
   
執行

```sh
emerge -a sudo
```

以安裝 `sudo`

使用 `nano` 編輯 `/etc/sudoers`

(TO BE LINKED to Arch Manually Install)

#### 設定 `systemd`

執行

```sh
systemd-machine-id-setup
systemd-firstboot --prompt
```

並按提示進行設定

```sh
systemctl preset-all --preset-mode=enable-only
systemctl preset-all
```

以載入所有服務的預設設定

#### 設定 Internet

> 以使用 `networkmanager` 爲例

執行

```sh
emerge -ag networkmanager
```

> `-a` 即 `--ask`，要求二次確認；`-g` 即 `-getbinpkg`，優先使用二進制包

再執行

```sh
systemctl enable NetworkManager
```

以啓用服務

#### 安裝作業系統工具

1. 效能優化
   
   安裝 `io-scheduler-udev-rules`

2. 檔案索引
   
   安裝 `mlocate`

3. `bash` 補全

   安裝 `bash-completion`

4. 檔案系統工具

   根據所使用的檔案系統安裝

   > 例如，本機使用的是 btrfs, f2fs, FAT32 以及 LVM

   > 則應安裝 `btrfs-progs` `f2fs-tools` `dosfstools` `lvm2`

5. WLAN 工具
   
   安裝 `iw` `wpa_supplicant`

#### 安裝啓動載入器

> 以使用 `systemd-boot` 爲例

執行

```sh
bootctl install
```

隨後執行

```sh
bootctl list
```

檢查啓動項目

> 注意：若使用了 LVM，則應該爲包 `sys-fs/lvm2` 加入 `lvm` 的 `USE Flags`，並在 `dracut` 中要求加入 `lvm` 模塊

> 使用 `nano` 編輯 `/etc/dracut.conf`
>
> 加入
>
> ```
> add_dracutmodules+=" lvm "
> ```
>
> 執行
>
> ```sh
> emerge -a --config <所安裝的內核>
> ```
>
> 最後，檢查啓動項目

#### 重新開機進入 Gentoo Linux

於 `chroot` 環境中執行 `exit`

執行

```sh
cd
umount -l /mnt/gentoo/dev{/shm,/pts,} 
umount -R /mnt/gentoo 
reboot
```

於開機選單中選擇 Linux Boot Manager 或 UEFI OS 以進入 Gentoo Linux

### 設定 Gentoo Linux 的 GUI

> 以 KDE Plasma 爲例

- 安裝 X Server 以及 NVIDIA GPU 驅動程式

  執行

  ```sh
  emerge -ag xorg-server nvidia-drivers
  ```

  > 請注意，若您使用的是 NVIDIA GPU 作爲唯一視訊輸出 GPU，請加入該配置檔案
  >
  > `/etc/X11/xorg.conf.d/nvidia.conf`
  >
  > ```
  > Section "Device"
  >    Identifier  "nvidia"
  >    Driver      "nvidia"
  > EndSection
  > ```


- 安裝 KDE Plasma
  
  執行

  ```sh
  emerge -ag plasma-meta sddm kde-apps/dolphin ark konsole
  ```

- 啓用 SDDM 登入

  執行

  ```sh
  systemctl enable sddm
  ```

- 啓用 Pipewire 音訊

  使用者權限執行

  ```sh
  systemctl --user enable pipewire
  systemctl --user enable pipewire-pulse
  systemctl --user enable wireplumber
  ```

- 允許使用者使用音訊裝置以及圖形硬體加速

  執行

  ```sh
  usermod -aG audio <使用者名稱>
  usermod -aG video <使用者名稱>
  ```


**至此，您已完成 Gentoo Linux 的基本安裝作業**
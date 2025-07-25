# Gentoo Linux 安裝指南

> 本文僅適用於對 GNU/Linux 有一定使用經驗且意欲執行 Gentoo Linux 的簡單安裝的讀者，因此並不會涉及每一步驟的原理解釋。

> 簡而言之，雖然“複製 - 粘貼 - 執行”並非本文之目的，但讀者可以藉此簡要地瞭解 Gentoo Linux 的安裝流程並推廣至其他桌面環境以及分割表配置。

> Gentoo Linux 是基於源碼構建的作業系統，因此其對於絕大多數的硬體都有極好的相容性，此處以 amd64 (x86_64) 架構且使用 UEFI BIOS 的計算機的安装为例。

## 1. 歸檔並燒錄映像檔

前往 [官方網站](https://www.gentoo.org/downloads/) 選擇鏡像並歸檔映像檔

使用 [Etcher](https://etcher.balena.io/) 燒錄映像檔至 USB Disk

## 2. 變更 UEFI BIOS 設定並載入 ISO

> 此處要强調的是 “Live ISO” 環境，使用 Gentoo Linux Live ISO 并不是强制條件。慾使用 GUI 對安裝作業環境進行配置，如連線至校園網，建議使用 EndeavourOS Live ISO 或任意一個您熟悉的 ISO。

> 請注意，Gentoo Linux 提供了使用 `OpenRC` init 程式的 Live ISO，請按需取用。

## 3. 執行安裝作業前的準備

### 準備磁碟

#### 調整磁碟分割表

> 本文僅以支援 UEFI BIOS 的裝置爲例，若需要實現同時支援從 MBR 和 GUID 分割表啓動，請自行參閲相關文章。

磁碟至少包含兩個分割，分別用於挂載根目錄和 EFI System Partition

> 慾獨立 `/home` `/usr` `/opt` 分割，請按需調整。

> 使用 btrfs 檔案系統建立多磁碟容器時，可以考慮使用其子卷機制代替獨立分割。

- EFI System Partition 分割至少需要 500 MB

  關於在單個磁碟上使用同一個 ESP 分割，其挂載點有兩個建議的選擇：

  > 若磁碟上存在非 GNU/Linux 作業系統，如 Windows，請謹慎考慮使用單個 ESP ！

  | 挂載點 | 優勢 | 劣勢 |
  | - | - | - |
  | `/boot` | 可執行 EFI 檔案與内核映像位於同一分割，便於管理維護 ESP 且擁有最佳相容性 | 可能會篡改其他 GNU/Linux 作業系統的相關檔案且所需空間較大 |
  | `/efi` | 分離内核映像與實現 UEFI 啓動所需檔案，避免篡改其他 GNU/Linux 作業系統的相關檔案 | *冇得彈* |
  
  > 根據 [Gentoo Wiki](https://wiki.gentoo.org/wiki/EFI_System_Partition#Optional:_autofs)，不推薦將 ESP 挂載於 `/boot/efi` 

  > 若為不同作業系統建立不同的 ESP，以上兩者均可

- 根目錄可以按需調整，建議至少 40 GB

- `swap` 按需建立，但建議安裝的 RAM 较大 (64 GB 及以上) 的計算機不建立

  此爲 Gentoo Handbook 中對 `swap` 的建議

  | RAM size | With suspend support | With hibernation support |
  | - | - | - |
  | 2 GB or less | 2*RAM | 3*RAM |
  | 2 to 8 GB | RAM amount | 2*RAM |
  | 8 to 64 GB | 8 GB minimum, 16 GB maximum | 1.5*RAM |
  | 64 GB or greater | 8 GB minimum | Hibernation **not RECOMMENDED**! |

#### 建立檔案系統

- 對於 EFI System Partion，請使用 FAT32

- 對於 `/`，建議使用 btrfs，並將 `/var/log` 獨立掛載於一個子卷

  慾進一步細分，還可以將 `/var/cache` 與 `/var/tmp` 獨立掛載爲子卷 (此兩者分別為 Gentoo portage 的包快取與編譯工作資料夾)

  > 若您有足夠的 RAM，可以考慮將 `/var/tmp/portage` 掛載爲 tmpfs 以減少編譯過程中的磁碟讀寫
  >
  > 於 `/etc/fstab` 中加入
  >
  > ```
  > tmpfs           /var/tmp/portage                tmpfs   size=<大小>G,uid=portage,gid=portage,mode=775    0 0
  > ```

- 對於其他獨立的分割區，建議使用 F2FS (機械硬碟可以使用 XFS)

### 連線至 Internet 並校準時間

此舉是爲歸檔並解壓 stage3 檔案做準備

#### 對於 `systemd`

執行
```sh
timedatectl 
```
以檢查作業系統時間狀態。

執行
```sh
timedatectl list-timezone
```
以列出時區，同樣的，使用 `|` (pipe) 和 `grep` 以篩選輸出結果。

欲選取作業系統時區，請執行
```sh
timedatectl set-timezone <時區>
```
例如
```sh
timedatectl set-timezone Asia/Hong_Kong
```

之後，可以重新啓動 `systemd-timesyncd.service` 以手動同步化時間。

```sh
systemctl restart systemd-timesyncd
```

#### 對於 `OpenRC`

執行
```sh
ln -sf /usr/share/zoneinfo/<地區>/<城市> /etc/localtime
```
以設定時區

根據時間同步化服務按需重啓服務

例如 `chrony`

```sh
rc-service chronyd restart
```

## 開始 Gentoo Linux 的安裝作業

### 安裝 stage3 檔案

> stage3 檔案是 Gentoo Linux 的 “種子”，是基於某些作業系統設定檔自動構建的，幾乎包含了整個基礎作業系統的檔案。

> 在選擇 stage3 檔案時，必須考慮清楚慾使用的 init 程式是 `systemd` 還是 `OpenRC`，這會省下不必要的麻煩。

- 掛載磁碟分割

  1. 建立 `/mnt/gentoo` 資料夾
   
  2. 掛載作爲根目錄的磁碟分割或 btrfs 子卷至 `/mnt/gentoo` 並建立 `/mnt/gentoo/efi` `/mnt/gentoo/home` 以及其他您欲獨立掛載的掛載點
   
  3. 掛載所有磁碟分割或子卷

- 安裝 stage3 檔案

  1. 切換目錄至 `/mnt/gentoo`
   
  2. 使用 `links` 或 `wget` 歸檔 stage3 檔案
   
例如

```sh
links https://mirrors.cernet.edu.cn/gentoo/releases/amd64/autobuilds/
```

- 選擇一個資料夾

  對於 `systemd` 請使用 `current-stage3-amd64-desktop-systemd` 中的檔案

  對於 `OpenRC` 請使用 `current-stage3-amd64-desktop-openrc` 中的檔案

- 歸檔完成後，請按需執行檔案校驗並解壓 stage3 檔案

  於 `/mnt/gentoo` 目錄中執行

  ```sh
  tar xpf stage3-amd64-desktop-<init 程式>-<時間戳>.tar.xz --xattrs-include='*.*' --numeric-owner
  ```

  以解壓縮 stage3 檔案至目標根目錄

### 調整 `portage` 設定

> `portage` 是 Gentoo Linux 的包管理體系，是 Gentoo 的核心

#### 變更 `make.conf` 設定

> 詳情請參閲 https://wiki.gentoo.org/wiki//etc/portage/make.conf

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

> 慾使用編譯器檢測的原生微架構，可使用 `-march=native`，此舉會讓本機編譯安裝的二進制檔案無法在別的微架構的 CPU 上如期運作

> 根據 Gentoo Handbook，`-O2` 是最合適的等級，`-O3` 可以稍微加快速度，但 “某些時候會出現問題”

2. 變更編譯時的執行緒數量及負載

例如本機的 CPU 有 12 條執行緒，RAM 的大小也大於或等於 32 GB，則

加入行

```
MAKEOPTS="-j12 -l12"
```

比較 CPU 的執行緒數量與 RAM 的一半大小，兩者中取最小值填入。

3. 調整 `portage` 進程的優先級及並行作業數量

加入行
```
PORTAGE_SCHEDULING_POLICY="idle"
```

以規定 `portage` 自身及其建立的進程的優先級爲 idle，減少安裝程式時對電腦日常使用的影響

加入行
```
EMERGE_DEFAULT_OPTS="--jobs 4"
```

以規定 `portage` 最大並行作業數量爲 4

4. 加入 Gentoo 倉庫鏡像

加入行

```
GENTOO_MIRRORS="https://mirrors.tuna.tsinghua.edu.cn/gentoo"
```

以使用清華 TUNA 鏡像伺服器

> 鏡像可以根據地理位置調整，例如廣東省的使用者可以使用 SUSTech CRA 鏡像
>
> ```
> GENTOO_MIRRORS="https://mirrors.sustech.edu.cn/gentoo"
> ```

5. 設定顯示卡
   
`portage` 會根據 `VIDEO_CARDS` 設定編譯安裝所需的包

- 對於 4th Gen 或更新的 Intel Integrated GPU 以及 Intel ARC GPU

  ```
  VIDEO_CARDS="intel"
  ```

- 對於慾使用專有驅動程式的 NVIDIA GPU

  ```
  VIDEO_CARDS="nvidia"
  ```

- 對於 Radeon Integrated GPU 和 Radeon Discrete GPU，請寫入

  ```
  VIDEO_CARDS="amdgpu radeonsi"
  ```

- 若使用的是 QEMU/KVM 虛擬機，請啓用
  ```
  VIDEO_CARDS="virgl"
  ```

  以使用 VirtIO 的 OpenGL 加速

請根據自己的硬體配置組合使用上述例子

> 例如，對於一部使用帶有 Radeon Integrated GPU 的 Ryzen CPU 和 NVIDIA Discrete GPU 的筆電，請寫入
>
> ```
> VIDEO_CARDS="amdgpu radeonsi nvidia"
> ```


6. 設定全域 `USE` 變數

加入行

```
USE=""
```

並與引號內加入

例如，本機慾使用 X Server, Wayland, KDE Plasma, Pipewire Sound Server, fcitx，並優先使用二進制分發的包，內核使用 Gentoo 發行版標準內核

可以寫入

```
USE="X wayland kde qt pipewire pulseaudio fcitx bindist dist-kernel -gnome"
```

7. 設定所接受的“許可證”

推薦對全域使用如下設定，並建立 `/etc/portage/package.license/` 資料夾，於其對某些包進行例外設定

```
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"
```

#### 添加 `portage` 的倉庫

1. 建立倉庫設定檔案的資料夾

執行

```sh
mkdir /mnt/gentoo/etc/portage/repos.conf
```

此資料夾用於存放 gentoo 倉庫以及 `eselect` 中啓用的倉庫的設定檔案

1. 拷貝 Gentoo Linux 官方倉庫 (gentoo) 的設定檔案

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

> 若您的 CPU 支援 x86-64-v3，可以考慮啓用，將結尾替換爲 x86-64-v3 即可

### 進入 Gentoo Linux 執行後續安裝及設定

#### 使用 `chroot` 進入 Gentoo Linux

1. 拷貝 DNS 解析檔案

> 這一步至關重要，否則有可能無法存取 Internet

執行

```sh
cp --dereference /etc/resolv.conf /mnt/gentoo/etc/
```

2. 掛載必要的檔案系統並進入 Gentoo Linux

- 若您使用的是 Arch Linux 及其衍生版的 Live ISO 或有 `arch-chroot` 程式，這一步非常簡單

  執行

  ```sh
  arch-chroot /mnt/gentoo
  ```

  隨後可以變更 Prompt 加以區分 chroot 環境

  ```sh
  export PS1="(chroot) ${PS1}"
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

  > 這一步非常重要，`/proc` `/sys` `/dev` `/run` 都是安裝作業系統必要的“偽檔案系統”

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

> 注意：此處的設定檔必須與 stage3 檔案的一致，例如 `systemd` 與 `OpenRC` 的選擇

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

> 簡便表達，使用 `-a1`

> `--oneshot` 是表示不要將該包錄入 @world 集合中，因爲該程式只需要使用一次

以安裝

再執行

```sh
echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
```

以匯入所有 `CPU_FLAGS`

#### 更新 @world 集合中的所有包（可選步驟）

> 這一步並非必要，因爲安裝完成且進入作業系統後再更新亦無妨，若後續出現編譯或安裝問題可以返回執行這一步

執行

```sh
emerge --ask --verbose --update --deep --newuse @world
```

> 簡便表達，可以使用 `-avuDN`

> 可以考慮加入 `--getbinpkg` 以使用二進制包；簡便表達爲 `-g`，也就是 `emerge -agvuDN @world`

即可

#### 設定區域格式

> 這一步與手動安裝 Arch Linux 的非常相似

使用 `nano` 於 `/etc/locale.gen` 反註釋或手動寫入 locale

執行

```sh
locale-gen
```

以生成區域格式

執行

```sh
eselect locale list
```

以檢視區域格式列表

使用

```sh
eselect locale set <ID>
```

以設定全域區域格式

最後執行

```sh
env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
```

以套用區域格式變更

#### 設定時區

> 這一步也與手動安裝 Arch Linux 的非常相似

執行

```sh
ln -sf /usr/share/zoneinfo/<地區>/<城市> /etc/localtime
```

#### 安裝 Linux 內核及韌體

- 以將 `systemd` 作爲 init 程式並使用 `systemd-boot` 啓動 `dracut` 建立的 `gentoo-kernel-bin` 的 Unified Kernel Image (UKI) 爲例

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
   uefi="yes"    # 支援 UEFI 啓動，UKI 必須啓用
   hostonly="yes"    # 建立適用於本機的精簡 initramfs，節約磁碟空間
   kernel_cmdline="所需執行的命令"    # 一般而言需要指定根目錄所在分割及其挂載選項
   ```

   > 同樣的，`dracut` 也支援分類管理配置檔案，建立 `/etc/dracut.conf.d/` 資料夾並於此加入配置檔案

   > 對於一個使用 btrfs 檔案系統的計算機而言， `kernel_cmdline` 通常包括
   >
   > `root=UUID=<分割的UUID>`
   >
   > `rootflags=subvol=<根目錄所在子卷>`
   >
   > `rootfstype=btrfs`
   >
   > `rw`
   >
   > 填入時請以空格分隔

   **請注意，根目錄位於 LVM 邏輯卷中的讀者請不要在 KERNEL_CMDLINE 使用 UUID 指定根目錄**

   **可以使用 `root=/dev/mapper/<VG 名稱>-<LV 名稱>`**

   > 注意：若使用了 LVM，則應該爲包 `sys-fs/lvm2` 加入 `lvm` 的 USE Flag，並在 `dracut` 中要求加入 `lvm` 模塊

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
   > emerge -a --config <所安裝的內核包>
   > ```
   >
   > 以重新建立 initramfs


- 以將 `OpenRC` 作爲 init 程式並使用 `systemd-boot` 啓動 `dracut` 建立的 `gentoo-kernel-bin` 的 Unified Kernel Image 爲例

  > 上述的第二步需要注意

  2. 加入 `sys-apps/systemd-utils` 的 `systemd-boot`, `udev` 以及 `kernel-install` 支援
    
    執行
    ```sh
    nano /etc/portage/package.use/systemd-utils
    ```

    寫入

    ```
    sys-apps/systemd-utils boot kernel-install udev
    ```

#### 撰寫 `/etc/fstab`

- 若使用的是 Arch Linux 及其衍生版的 Live ISO，這一步也非常簡單

  於 Live ISO 環境中以 root 登入

  執行

  ```sh
  genfstab -U /mnt/gentoo > /mnt/gentoo/etc/fstab
  ```

  回到 chroot 環境使用 `nano` 按需求修改 `/etc/fstab` 以變更掛載設定

- 若使用的 Live ISO 沒有 `genfstab`

  使用 `nano` 撰寫 `/etc/fstab`

  > 一個以 `/efi` 爲 EFI 分割，`/` 使用 btrfs 且獨立 `/home` 使用 f2fs 的設定應該是這樣的

  | UUID | Mount point | Filesystem type | Options | Dump | Filesystem check |
  | - | - | - | - | - | - |
  | UUID=<分割UUID> | / | btrfs | defaults,subvol=<根目錄所在子卷> | 0 | 1 |
  | UUID=<分割UUID> | /var/log | btrfs | defaults,subvol=<日誌目錄所在子卷> | 0 | 2 |
  | UUID=<分割UUID> | /efi | vfat | defaults | 0 | 2 |
  | UUID=<分割UUID> | /home | f2fs | defaults | 0 | 2 |

  **請注意，使用 LVM 的讀者請不要在 `/etc/fstab` 使用 UUID 指派挂載點**

  **可以使用 `/dev/mapper/<VG 名稱>-<LV 名稱>`**

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

以加入使用者賬戶並將其加入 wheel 特權組

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

使用 `nano` 編輯 `/etc/sudoers` 以允許使用者使用 `sudo`

#### 設定 init 程式

- 對於 `systemd`

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

- 對於 `OpenRC`

  建立 `/etc/hostname` 並寫入主機名

  請分別閱讀並按需修改 `/etc/rc.conf` `/etc/conf.d/keymaps` `/etc/conf.d/hwclock`

  > 建議於 `/etc/rc.conf` 中啓用 `rc_logger="YES"`

#### 設定網路介面卡

> 以使用 `networkmanager` 爲例

執行

```sh
emerge -ag networkmanager
```

> `-a` 即 `--ask`，要求二次確認；`-g` 即 `-getbinpkg`，優先使用二進制包

再啓用服務以開機載入

- 對於 `systemd`

  ```sh
  systemctl enable NetworkManager
  ```

- 對於 `OpenRC`

  ```sh
  rc-update add NetworkManager default
  ```

#### 安裝作業系統工具

1. 磁碟排程效能優化
   
   安裝 `io-scheduler-udev-rules`

2. 檔案索引
   
   安裝 `mlocate`

3. `bash` 命令補全

   安裝 `bash-completion`

4. 檔案系統工具

   根據所使用的檔案系統安裝

   > 例如，本機使用的是 btrfs, f2fs, FAT32 以及 LVM

   > 則應安裝 `btrfs-progs` `f2fs-tools` `dosfstools` `lvm2[lvm]`

5. WLAN 工具
   
   安裝 `iw` `wpa_supplicant`

   > 爲了更好的相容性，`net-wireless/wpa_supplicant` 還可以加入 `tkip` USE Flag

- 對於 `OpenRC` init，還需要安裝如下工具
  
  `app-admin/sysklogd`
  `sys-process/cronie`
  `net-misc/chrony`

  並分別啓用服務

  ```sh
  rc-update add sysklogd default
  rc-update add cronie default 
  rc-update add chronyd default
  ```

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

#### 重新開機進入 Gentoo Linux

於 chroot 環境中執行 `exit` 或同時按下 `Control` 和 `D`

返回 Live ISO 環境後執行

```sh
cd
umount -Rl /mnt/gentoo/
```

以取消挂載新系統的分割區

現在，重啓計算機並於開機選單中選擇 Linux Boot Manager 或 UEFI OS 以進入 `systemd-boot` 以啓動 Gentoo Linux

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
  >  ```
  >  Section "Device"
  >     Identifier  "nvidia"
  >     Driver      "nvidia"
  >  EndSection
  >  ```


- 安裝 KDE Plasma
  
  執行

  ```sh
  emerge -ag plasma-meta sddm kde-apps/dolphin ark konsole
  ```

- 啓用 SDDM 登入

  - 對於 `systemd`

  執行

  ```sh
  systemctl enable sddm
  ```

  - 對於 `OpenRC`
  
  修改 `/etc/conf.d/display-manager` 並指定 SDDM

  ```
  ......

  DISPLAYMANAGER="sddm"
  ```

  啓用 `display-manager` `elogind` 服務

  ```sh
  rc-update add display-manager default
  rc-update add elogind boot
  ```

- 啓用 Pipewire 音訊

  將使用者加入 `pipewire` 組

  ```sh
  usermod -aG pipewire <使用者名稱>
  ```

  - 對於 `systemd`

  以使用者權限執行

  ```sh
  systemctl --user enable pipewire
  systemctl --user enable pipewire-pulse
  systemctl --user enable wireplumber
  ```

  - 對於 `OpenRC`
  
  > 請檢查 `/bin/gentoo-pipewire-launcher` 是否已存在

  對於支援自動啓動的桌面環境，如 KDE Plasma，不需要額外操作

  其他桌面環境請參閱 https://wiki.gentoo.org/wiki/PipeWire#OpenRC

- 允許使用者使用可移動裝置以及圖形硬體加速

  執行

  ```sh
  usermod -aG plugdev <使用者名稱>
  usermod -aG video <使用者名稱>
  ```


**至此，您已完成 Gentoo Linux 的基本安裝作業**
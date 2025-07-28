# Arch Linux 安裝指南

## 歸檔並燒錄映像檔

前往 [官方網站](https://endeavouros.com/#Download) 選擇鏡像並歸檔

使用 [Etcher](https://etcher.balena.io/) 燒錄映像檔至 USB Disk

## 變更 UEFI BIOS 設定並載入 ISO

> 此處要强調的是 Arch 的 “Live ISO” 環境，使用 Arch Linux Install Medium ISO 并不是强制條件。慾使用 GUI 對安裝作業環境進行一定配置，如連線至校園網，亦可使用 EndeavourOS Live ISO。

### ASUS:

顯示 ASUS/ROG 徽標時按下 `Escape` 開啓啓動選單，選擇 `Enter UEFI Setup`。

按下 `F7` 進入「進階選單」，檢查「安全性」欄，關閉「安全啓動」或將類型變更爲 `Other OS` 或 `Microsoft & 3rd Party`。

套用變更並重設電源，於啓動選單中選擇 `UEFI: <Your Device Name>` 以載入 `grub` 載入程式，選擇載入 `Arch Linux install medium (x86_64, x64 UEFI)`，進入 Live ISO 環境。

## 執行安裝作業前的準備

### 選取作業系統的鍵盤格式

> 注意：此處以 English (US) 鍵盤格式爲例。

欲列出鍵盤格式，請執行

```sh
localectl list-keymaps
```

> 注意：您可以使用 `|` (pipe) 輸出至 `grep` 以篩選輸出結果，例如執行此命令以篩選美式鍵盤及其變種。

```sh
localectl list-keymaps | grep us
```

預設的鍵盤格式是 English (US)，請按需變更設定。

### 選取 Console 所使用的字體

在 Hi-DPI 熒幕上的 tty 可能顯示得非常小，此時可以變更字體

> 注意：終端字體是儲存在 `/usr/share/kbd/consolefonts/` 的。

執行

```sh
setfont <欲選取的字體>
```

例如

```sh
setfont ter-132b
```

### 連接至 Internet

欲檢查網路介面卡，執行

```sh
ip link
```

並檢查輸出項目中的 state，`UP` 即已啓用。

欲使用 WLAN 或 WWAN 介面卡，請首先透過 `rfkill` 檢查介面卡的封鎖狀態。

> `rfkill` 是 GNU/Linux 用於管理網絡介面卡的程式。

執行

```sh
rfkill list
```

以檢查所有介面卡

執行

```sh
rfkill enable <介面卡編號>
```

以 `rfkill list` 輸出結果如下爲例：

```
0: phy0: Wireless LAN
Soft blocked: yes
Hard blocked: no
2: hci0: Bluetooth
Soft blocked: yes
Hard blocked: no
```

此時我們可以執行

```sh
rfkill unblock 0
```

以解除對 WLAN 介面卡的封鎖。

至此，我們可以嘗試連接至 Internet。

- 對於乙太網路，請檢查綫纜是否穩定地插入 LAN 埠

- 對於 WLAN，請使用 `iwctl` 或 `nmtui` 進行認證

> `iwctl` 是 `iwd` (iNet Wireless Daemon) 的 CLI 程式；`nmtui` 是 `NetworkManager` 的 TUI 程式，使用起來更友好

使用 `iwctl`，例如：

執行

```sh
iwctl
[iwd] > device list
```

以檢查可用的介面卡。

執行

```sh
[iwd] > station <Name> get-networks
```

以取得 WLAN SSID 列表。

執行

```sh
[iwd] > station <Name> connect <SSID>
```

以選取欲連接的 WLAN 接入點。

> `<Name>` 即介面卡的名稱， `<SSID>` 即 WLAN 接入點的名稱。

提示 `Passphrase:` 時輸入密碼即可連接。

使用 `nmtui`，例如：

執行

```sh
nmtui
```

於選單中選取 `Activate a connection`

隨後，於次級選單中選擇一個 WLAN SSID 並輸入密碼進行認證

最後退出

- 對於流動寬頻調解器，請使用 `mmcli`

> 注意：鑒於流動寬頻調節器在當今相對罕見，此處不再贅述。(附上 [Wiki 連結](https://wiki.archlinux.org/title/Mobile_broadband_modem#ModemManager))

#### 驗證 Internet 存取

執行

```sh
ping www.archlinux.org
```

通常，若輸出結果有 `time=<latency> ms` 即可存取 Arch Linux 倉庫以完成安裝作業。

### 選取作業系統的時區

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
timedatectl set-timezone <Timezone>
```

例如

```sh
timedatectl set-timezone Asia/Hong_Kong
```

此時選取的是 HKT (UTC+8)

```sh
timedatectl set-timezone Asia/Chongqing
```

此時選取的是 CST (UTC+8)

### 準備磁碟

1. 檢查磁碟裝置

```sh
lsblk
```

此處可以檢查整個磁碟的大小以便區分

> `lsblk` 即“列出本機的塊裝置”

若您使用的是相同大小的甚至是相同型號的磁碟

請執行

```sh
fdisk -l
```

以仔細檢查

> `fdisk` 是一個 CLI 磁碟分割表管理程式，此處是“列出所有裝置的分割表”

2. 編輯磁碟分割表

與終端中執行

```sh
cfdisk <目標磁碟>
```

> `cfdisk` 是一個對使用者更友好的 `fdisk`，其使用了 Terminal User Interface (亦稱爲 Curses-based User Interface)

例如

```sh
cfdisk /dev/sda
```

即可進入 `cfdisk` 實用程式以編輯 `/dev/sda` 裝置的磁碟分割表

> 注意：由於 Arch Linux 會將 vmlinuz (亦稱 bzImage) 内核映像檔存放於 `/boot`，建議該分割不小於 500M；若要使用多個内核，請按需分配

執行完分割表編輯后，以 60G 的磁碟爲例，所得結果可能是這樣的：

| `/dev/sda` | Size | Type |
|-|-|-|
`/dev/sda1` | 0.5G | EFI System |
`/dev/sda2` | 40G | Linux root (x86-64) |
`/dev/sda3` | 19.5G | Linux home |

3. 對磁碟分割執行初始化

請再次執行 `lsblk` 以作參照

繼續先前例子，可以執行以下命令

```sh
mkfs.fat -F32 /dev/sda1 -n ARCH
mkfs.btrfs /dev/sda2 -f
mkfs.f2fs /dev/sda3 -f -l Home
```

> `mkfs` 是用於在磁碟分割區中建立檔案系統的命令


4. 檢查鏡像列表

執行

```sh
nano /etc/pacman.d/mirrorlist
```

加入欲使用的鏡像伺服器，按下 `Control` 和 `O` 寫入，按下 `Enter` 以使用原本的檔案名，再按下 `Control` 和 `X` 退出

例如於頂部寫入

```
Server = https://mirrors.cernet.edu.cn/archlinux/$repo/os/$arch
```

欲進一步瞭解該鏡像，請前往其 [官方網站](https://mirrors.cernet.edu.cn/about)

**至此，您已準備好執行安裝作業**

## 於 Live ISO 環境執行作業系統的安裝

### 使用 Arch Linux 的 `archinstall` 脚本執行安裝

> 注意：慾進行更多自訂，請查閱下文的手動安裝

爲確保安裝作業順利進行，請使用最新版本的 `archinstall`

執行

```sh
pacman -Sy archinstall
```

> `pacman` 是 Arch Linux 所使用的包管理員，`-Sy` 即要求與遠端（伺服器）進行同步化處理並重新整理本機資料庫，此處可簡單理解爲“重新整理本機資料庫並安裝 `archinstall`”

`archinstall` 是一個全自動的 Arch Linux 安裝脚本，提供了使用者以填寫問卷的形式執行安裝作業。

> 注意：於 `archinstall` 中， *Mirrors* 欄目應選取您所在地區的鏡像伺服器

*Disk configuration* 中應選取 *Manual Partitioning*，按下 `Space` 選取您先前編輯過磁碟分割表的硬碟，再按下 `Enter` 確認

繼續以上述例子進行設定

| Status | Device | Size | FS type | Mountpoint | Mount options | Flags | Btrfs vol |
|-|-|-|-|-|-|-|-|
| existing |`/dev/sda1` | 524 MB | fat32 | `/boot` | | Boot, ESP | |
| modify | `/dev/sda2` | 42 GB | btrfs | | compress=zstd | | 2 subvolume |
| existing | `/dev/sda3` | 20 GB | f2fs | `/home` | | | |

對於 btrfs 子卷的設定，可以參考此表格

| name | mountpoint | compress | nodatacow |
|-|-|-|-|
`@` | `/` | True | False |
`@log` | `/var/log` | False | True |
`@pkg` | `/var/cache/pacman/pkg` | False | True |

其他設定可以依照個人偏好設定

| Options | Description |
|-|-|
| Mirrors | 自訂 |
| Locales | 自訂 |
| Disk configuration | Manual Partitioning |
| Disk encryption | |
| Bootloader | Grub |
| Swap | False |
| Hostname | 自訂 |
| Root password | ****** |
| User account | 1 User(s) |
| Profile | Desktop |
| Audio | Pipewire |
| Kernels | linux |
| Additional packages | 自訂 |
| Network configuration | Use NetworkManager |
| Timezone | 自訂 |
| Automatic time sync (NTP) | True |
| Optional repositories | multilib |

對於 *Additional packages* ，可以加入：

`firefox` `nano` `fish`

對於 *Profile* ，建議新手選取 KDE Plasma 或 Gnome 桌面環境

選取 *Install* 以執行安裝

### 手動安裝 Arch Linux

> `archinstall` 提供了包辦式的安裝，但手動安裝始終能提供更靈活的自訂選項，此處以使用 LVM 磁碟配置以及使用 `systemd-boot` 啟動 Unified Kernel Image 為例

#### 準備 LVM

> LVM (Linux Volume Manager) 利用内核的裝置映射實現對儲存裝置虛擬化，在一個抽象出來的裝置中建立並使用虛擬磁碟分割，可以更加簡便地實現磁碟分割調整，不再需要擔心磁碟的連續空間

回到上文使用 `cfdisk` 進行分割表編輯，我們可以在磁碟上只劃分兩個分割，一個 `/boot`，一個作為 LVM 的物理卷

對於 `/dev/sda1`，請抹除為 FAT32 檔案系統

```sh
mkfs.fat -F32 /dev/sda1
```

對於 `/dev/sda2`，我們需要建立 LVM2 Physical Volume

```sh
pvcreate /dev/sda2
```

> `pvcreate` 意為“建立 Physical Volume”，指定磁碟分割即可

隨後，建立 Volume Group

```sh
vgcreate Arch /dev/sda2
```

> `vgcreate` 意為“建立 Volume Group”，賦予名稱 (此處名為 Arch)，並指定一個或多個已作為 LVM Physical Volume 使用的磁碟分割

最後，建立 Logical Volume

```sh
lvcreate -L 40G Arch -n root
lvcreate -l 100%FREE Arch -n home
```

> `lvcreate` 意為“建立 Logical Volume”，於某個 Volume Group 中分配具體的或按百分比計算的餘下的空間給某個 Logical Volume 並賦予其名稱

#### 於 LVM 中建立檔案系統

> LVM 中的 Logical Volume 是被映射至 Volume Group 裝置中的分割

此處以根目錄使用 btrfs 以及家目錄使用 f2fs 檔案系統爲例：

1. 建立檔案系統

```sh
mkfs.btrfs /dev/Arch/root
mkfs.f2fs /dev/Arch/home
```

2. 挂載並建立 btrfs 子卷

將整個 btrfs 容器掛載至 `/mnt`

```sh
mount /dev/Arch/root /mnt
```

於 btrfs 容器中建立子卷

```sh
cd /mnt
btrfs subvolume create @
btrfs subvolume create @log
btrfs subvolume create @pkg
cd && umount /mnt
```

3. 掛載分割區

我們需要先掛載用作新系統的根目錄的 btrfs 子卷

```sh
mount /dev/Arch/root -o subvol=@,compress=zstd /mnt
```

於新系統的根目錄中建立其他分割的掛載點

```sh
cd /mnt
mkdir boot home
mkdir -p ./var/log
mkdir -p ./var/cache/pacman/pkg
```

掛載所有分割區

```sh
mount /dev/sda1 /mnt/boot
mount /dev/Arch/home /mnt/home
mount /dev/Arch/root -o subvol=@log,nodatacow /mnt/var/log
mount /dev/Arch/root -o subvol=@pkg,nodatacow /mnt/var/cache/pacman/pkg
```

請執行 `lsblk` 再次檢查，確保所有分割區已正確掛載

#### 安裝基礎系統包

> `pacstrap` 是 Arch Linux 所使用的“播種器”，為整個系統建立樹狀檔案結構並安裝一些必需的包，例如包管理員

執行以下命令以安裝：

```sh
pacstrap -K /mnt base base-devel linux-firmware linux linux-headers nano fish networkmanager btrfs-progs f2fs-tools dosfstools lvm2
```

對於 Intel CPU，還需要安裝 `intel-ucode`

對於 AMD CPU，還需要安裝 `amd-ucode`

> 若需要使用 `linux-zen` 內核，請替換 `linux` `linux-headers` 為 `linux-zen` `linux-zen-headers`

#### 建立新系統的檔案系統表格

> Arch Linux Live ISO 中可使用 `genfstab` 自動生成

執行以生成一個檔案系統表：

```sh
cd
genfstab -U /mnt > /mnt/etc/fstab
```

此處我們需要再次檢查 `/mnt/etc/fstab` 的內容

```sh
nano /mnt/etc/fstab
```

- 欲使用 `fstrim.timer` 進行排程 TRIM，請停用檔案系統的 `discard`

  對於 btrfs 與 f2fs，請將 `/mnt/etc/fstab` 中的挂載選項的 `discard` 改為 `nodiscard`

#### 進入 chroot 環境配置

> Arch Linux 使用 `arch-chroot` 自動化程式，此舉是“切換根目錄”，即以 root 身份登入新系統

```sh
arch-chroot /mnt
```

1. 設定當地時間

```sh
ln -sf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
```

> `ln -sf` 是建立一個名爲 `/etc/localtime` 的符號鏈接，此處該鏈接指向 `/usr/share/zoneinfo/Asia/Hong_Kong`

2. 設定主機名

```sh
nano /etc/hostname
```

並於首行輸入主機名，可以按照 `<使用者>-<作業系統>-<裝置型號或牌子>` 的格式命名，例如

```
Jimmy-Arch-SP5
```

3. 設定慾使用的 locale

```sh
nano /etc/locale.gen
```

> 此處必須反注釋 `en_US.UTF-8 UTF-8` ，在此基礎上可以再反注釋其餘語言格式以支援多語言

生成慾使用的 locale

```sh
locale-gen
```

4. 安裝必要程式、驅動程式以及 KDE Plasma

```sh
pacman -Sy openssh htop wget iwd wireless_tools wpa_supplicant smartmontools xdg-utils plasma-meta plasma-workspace ark dolphin konsole egl-wayland pipewire pipewire-alsa pipewire-jack pipewire-pulse gst-plugin-pipewire libpulse wireplumber sddm
```

對於 Intel GPU 及 NVIDIA GPU 的 Hybrid 配置

```sh
pacman -S mesa intel-media-driver libva-intel-driver vulkan-intel xorg-server xorg-xinit nvidia-dkms
```

5. 配置 Unified Kernel Image

> Arch Linux 預設使用 `mkinitcpio` 建立 initramfs 映像檔

建立 Unified Kernel Image 存放資料夾

```sh
mkdir -p /boot/EFI/Linux
```

變更 `mkinitcpio` 設定

```sh
nano /etc/mkinitcpio.conf
```

於已啟用的 HOOKS 預設檔案中加入 `lvm2` 以支援 LVM 邏輯卷根目錄挂載

變更内核包的映像檔建立設定

```sh
nano /etc/mkinitcpio.d/<包名稱>.preset
```

註釋 `default_image` 與 `fallback_image` 項目

反註釋 `default_uki` 與 `fallback_uki` 項目，並將其存放分割區由 `/efi/EFI/Linux/<包名稱>.efi` 改為 `/boot/EFI/Linux/<包名稱>.efi`

最後，設定内核啓動要執行的命令

```sh
mkdir /etc/cmdline.d
```

> `mkinitcpio` 在建立映像檔時會使用 `/etc/cmdline.d/` 中的配置檔案，可以分類管理

指定根目錄的掛載選項

```sh
nano /etc/cmdline.d/root.conf
```

寫入

```
root=/dev/Arch/root rootflags=subvol=@ rootfstype=btrfs rw
```

隨後執行

```sh
mkinitcpio -P
```

以建立所有内核的 Unified Kernel Image

6. 安裝 `systemd-boot` 啟動載入器

```sh
bootctl install
```

執行

```sh
nano /boot/loader/loader.conf
```

反註釋 `timeout 3` 與 `console-mode keep`

加入

```
auto-entries yes
editor yes
```

執行

```sh
bootctl list
```

以檢查啟動項目

7. 安裝程式包

> 若您使用的是 EOS 的 Live ISO，請務必將 `/etc/pacman.conf` 中的 EOS Repository 刪除

執行

```sh
pacman -S fcitx5 fcitx5-chinese-addons kcm-fcitx5 fcitx5-qt fcitx5-gtk ttf-sarasa-gothic snapper snap-pac power-profiles-daemon
```

> 注意：`power-profiles-daemon` 爲 Plasma 桌面的電源計劃管理程式，您也可以使用 `tlp` 對其進行替換，在進入新系統前請啟用服務


8. 使用者管理

執行

```sh
passwd
```

以變更 root 的密碼

執行

```sh
useradd -m -G wheel <使用者名稱>
```

以建立自己的使用者賬戶

> 此處使用 `useradd` 新增使用者賬戶，`-m` 即於 `/home` 建立該使用者的家目錄，`-G` 即加入某個組

9. 允許使用者賬戶使用 `sudo`

> `sudo` 是 GNU/Linux 上最常用的暫時賦予使用者管理員權限的工具

執行

```sh
nano /etc/sudoers
```

於

```
root ALL=(ALL:ALL) ALL
```

下行加入

```
<使用者名稱> ALL=(ALL:ALL) ALL
```

寫入該檔案並按原檔案名儲存

10. 取消磁碟挂載

完成後回到終端，按下 `Control` 和 `D` 或直接執行 `exit` 以回到 Live ISO 環境

執行 `cd` 以切換目錄至當前使用者的家目錄

取消挂載所有磁碟，執行

```sh
umount -Rl /mnt
```

隨後重新開機，並啟動 Arch Linux 檢查各項功能是否正常


**至此，您已完成 Arch Linux 的基本安裝作業**
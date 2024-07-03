# Installation Guide for Arch Linux, An Arch Linux for EVERYONE!

## 1. 歸檔並燒錄映像檔

前往 [官方網站](https://endeavouros.com/#Download) 選擇鏡像並歸檔

使用 [Etcher](https://etcher.balena.io/) 燒錄映像檔至USB Disk

## 2. 變更 UEFI BIOS 設定並載入 ISO

>此處要强調的是 Arch 的 “Live ISO” 環境，使用 Arch Linux Install Medium ISO 并不是强制條件。因爲在此 ISO 中有條件不一定充分，例如高校學生欲使用要求身份認證的校園網執行安裝作業，此時利用 Endeavour OS 的 Live ISO 或許更好。

### ASUS:

顯示 ASUS/ROG 徽標時按下 `Escape` 開啓啓動選單，選擇 `Enter UEFI Setup`。

按下 `F7` 進入進階選單，檢查「安全性」欄，關閉「安全啓動」或將類型變更爲 `Other OS` 或 `Microsoft & 3rd Party`。

套用變更並重設電源，於啓動選單中選擇 `UEFI: <Your Device Name>` 以載入 `grub` 載入程式，選擇載入 `Arch Linux install medium (x86_64, x64 UEFI)`，進入Live ISO環境。

## 3. 執行安裝作業前的準備

### 選取作業系統的鍵盤格式

>注意：此處以 UTC+8 以及 English (US) 鍵盤格式爲例。

欲選擇鍵盤格式，請執行
```sh
localectl list-keymaps
```

>此舉是列出所有可用的鍵盤格式。

>注意：您可以使用 pipe 輸出至 `grep` 以篩選輸出結果，例如執行此命令以篩選美式鍵盤及其變種。

```sh
localectl list-keymaps | grep us
```

預設的鍵盤格式是 English (US)，請按需變更設定。

### 選取 Console 所使用的字體

在 Hi-DPI 熒幕上的 tty 可能顯示得非常小，此時可以變更字體

>注意：終端字體是儲存在 `/usr/share/kbd/consolefonts/` 的。

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
並檢查輸出項目中的 `state`，UP 即已啓用。

欲使用 WLAN 或 WWAN（即無綫介面卡），請透過 `rfkill` 檢查介面卡的封鎖狀態。

> `rfkill` 是 `systemd` init 程式的無綫介面卡控制程式

執行
```sh
rfkill list
```
以檢查所有無綫介面卡

執行
```sh
rfkill enable <介面卡編號>
```

以 `rfkill list` 輸出結果如下爲例：
```sh
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

#### 對於乙太網路，請檢查綫纜是否穩定地插入 LAN 埠。

#### 對於 WLAN，請使用 `iwctl` 或 `nmtui` 進行認證。

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
以獲取 WLAN SSID 列表。

執行
```sh
[iwd] > station <Name> connect <SSID>
```
以選取欲連接的 WLAN。

>`<Name>` 即介面卡的名稱， `<SSID>` 即 WLAN 的名稱。

提示 `Passphrase:` 時輸入密碼即可連接。

使用 `nmtui`，例如：

執行
```sh
nmtui
```

於選單中選取 `Activate a connection`

隨後，於次級選單中選擇一個 WLAN SSID 並輸入密碼進行認證

最後退出

#### 對於流動寬頻調解器，請使用 mmcli。
>注意：鑒於流動寬頻調節器在當今相對罕見，此處便不再贅述。（附上[Wiki連結](https://wiki.archlinux.org/title/Mobile_broadband_modem#ModemManager)）

#### 現在，驗證是否有 Internet 存取。

執行
```sh
ping www.archlinux.org
```
通常，若輸出結果有 `time=<latency> ms` 即可存取至 Arch Linux 倉庫以完成安裝作業。

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
以列出時區，同樣的，使用 pipe 和 `grep` 以篩選輸出結果。

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

**首先，我們需要檢查磁碟裝置**
```sh
lsblk
```
此處注意檢查整個磁碟的大小以區分

>`lsblk` 即“列出本機的塊裝置”

若您使用的是相同大小的甚至是相同型號的磁碟
請執行
```sh
fdisk -l
```
以仔細檢查

>`fdisk` 是一個使用命令行使用者界面的磁碟分割表與分割區管理程式，此處是“列出所有裝置的分割表及其分割區”

**確定目標磁碟后，開始編輯磁碟分割表**

與終端中執行
```sh
cfdisk <目標磁碟>
```

>`cfdisk` 是一個對使用者更友好的 `fdisk`，其使用了 Terminal User Interface（亦稱爲 Curses-based User Interface），這樣更容易讓您清楚您在做什麽

例如
```sh
cfdisk /dev/sda
```
即可進入 `cfdisk` 實用程式以編輯 `/dev/sda` 裝置的磁碟分割表

>注意：由於 Arch Linux 會把 vmlinuz （亦稱 bzImage）内核映像檔存放於 `/boot`，建議該分割不小於 500M，若要使用多個内核，請按需分配

執行完分割表編輯后，以 60G 的磁碟爲例，所得結果可以是這樣的：
| `/dev/sda` | Size | Type |
|-|-|-|
`/dev/sda1` | 0.5G | EFI System |
`/dev/sda2` | 40G | Linux root (x86-64) |
`/dev/sda3` | 19.5G | Linux home |

**對磁碟分割執行初始化**

請再次執行 `lsblk` 以作參照

繼續先前例子，可以執行以下命令

```sh
mkfs.fat -F32 /dev/sda1 -n ARCH
mkfs.btrfs /dev/sda2 -f
mkfs.f2fs /dev/sda3 -f -l Home
```
>`mkfs` 是用於在磁碟分割區中建立檔案系統的命令


**在使用執行安裝作業前，請檢查鏡像列表**

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

#### 至此，您已準備好執行安裝作業。

## 於 Live ISO 環境執行作業系統的安裝

### 使用 Arch Linux 的 `archinstall` 脚本執行安裝

>注意：慾進行更多自訂，請查閱下文的手動安裝

爲確保安裝作業順利進行，請使用最新版本的 `archinstall`

執行
```sh
pacman -Sy archinstall
```
>`pacman` 是 Arch Linux 所使用的包管理員，`-Sy` 即要求與遠端（伺服器）進行同步化處理並重新整理本機資料庫，此處可簡單理解爲“重新整理本機資料庫並安裝 `archinstall`”

`archinstall` 是一個全自動的 Arch Linux 安裝脚本，提供了使用者以填寫問卷的形式執行安裝作業。

**開始安裝**

>注意：於 `archinstall` 中， *Mirrors* 欄目應選取您所在地區的鏡像伺服器

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

對於 *Profile* ，建議選取 KDE Plasma 或 Gnome 桌面環境

選取 *Install* 以執行安裝

### 手動安裝 Arch Linux

>`archinstall` 提供了包辦式的安裝，但手動安裝始終能提供更靈活的自訂選項，此處以使用 LVM 磁碟配置以及使用 systemd-boot 啟動 Unified Kernel Image 為例

#### 準備 LVM

> LVM (Logical Volume Manager) 利用内核的裝置映射實現對儲存裝置虛擬化，在一個抽象出來的裝置中建立並使用虛擬磁碟分割，可以更加簡便地實現磁碟分割調整，例如不再需要擔心磁碟的連續空間

回到上文使用 `cfdisk` 進行分割表編輯，我們可以在磁碟上只劃分兩個分割，一個 `/boot`，一個作為 LVM 的物理卷

對於 `/dev/sda1`，請抹除為 FAT32 檔案系統

```sh
mkfs.fat -F32 /dev/sda1
```

對於 `/dev/sda2`，我們需要建立 LVM2 Physical Volume

```sh
pvcreate /dev/sda2
```

>`pvcreate` 意為“建立 Physical Volume”，並指定一個磁碟分割

隨後，建立 Volume Group

```sh
vgcreate Arch /dev/sda2
```

>`vgcreate` 意為“建立 Volume Group”，賦予名稱（此處名為 Arch），並指定一個或多個已作為 LVM Physical Volume 使用的磁碟分割

最後，建立 Logical Volume

```sh
lvcreate -L 40G Arch -n root
lvcreate -l 100%FREE Arch -n home
```

>`lvcreate` 意為“建立 Logical Volume ”，於某個 Volume Group 中分配具體的或按百分比計算的餘下的空間給某個 Logical Volume 並賦予其名稱

#### 於 LVM 中建立檔案系統

>LVM 中的 Logical Volume 是被映射至 Volume Group 裝置中的分割

此處以根目錄使用 btrfs 以及家目錄使用 f2fs 檔案系統爲例：

```sh
mkfs.btrfs /dev/Arch/root
mkfs.f2fs /dev/Arch/home
```

**建立 btrfs 子卷**

將整個 btrfs 容器掛載至 `/mnt`

```sh
mount /dev/Arch/root /mnt
```

於容器中建立子卷

```sh
cd /mnt
btrfs subvolume create @
btrfs subvolume create @log
btrfs subvolume create @pkg
cd && umount /mnt
```

**掛載分割區**

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

>`pacstrap` 是 Arch Linux 所使用的“播種器”，為整個系統建立樹狀檔案結構並安裝一些必需的包，例如包管理員

```sh
pacstrap -K /mnt base base-devel linux-firmware linux linux-headers nano fish networkmanager btrfs-progs f2fs-tools dosfstools lvm2
```

對於 Intel CPU，還需要安裝 `intel-ucode`
對於 AMD CPU，還需要安裝 `amd-ucode`

>若需要使用 `linux-zen` 內核，請替換 `linux` `linux-headers` 為 `linux-zen` `linux-zen-headers`

#### 建立新系統的檔案系統表格

>Arch Linux 使用 `genfstab`，相較於 Gentoo Linux 的手動編寫，這是相當方便的

```sh
cd
genfstab -U /mnt > /mnt/etc/fstab
```

此處我們需要再次檢查 `/mnt/etc/fstab` 的內容

```sh
nano /mnt/etc/fstab
```

把掛載點為 `/var/log` 和 `/var/cache/pacman/pkg` 的分割區的掛載選項中的 `compress=zstd` 改為 `nodatacow`

**使用 fstrim.timer 進行排程TRIM**

對於 btrfs，請刪除 `/mnt/etc/fstab` 挂載選項中的 `discard` 項目

對於 f2fs，請將 `/mnt/etc/fstab` 中的挂載選項的 `discard` 改為 `nodiscard`

#### 進入 chroot 環境配置

>Arch Linux 使用 `arch-chroot` 自動化程式，此舉是“切換根目錄”，即以 root 身份登入新系統

```sh
arch-chroot /mnt
```

**設定當地時間**

```sh
ln -sf /usr/share/zoneinfo/Asia/Hong_Kong /etc/localtime
```

>`ln -sf` 是建立一個名爲 `/etc/localtime` 的軟鏈接，且該鏈接指向 `/usr/share/zoneinfo/Asia/Hong_Kong`

**設定主機名**

```sh
nano /etc/hostname
```

並於首行輸入主機名，可以按照 `<使用者>-<作業系統>-<裝置型號或牌子>`的格式命名，例如

```
Jimmy-Arch-SP5
```

**設定慾使用的 locale**

```sh
nano /etc/locale.gen
```

>此處建議在反注釋 `en_US.UTF-8 UTF-8` 的基礎上再反注釋 `zh_CN.UTF-8 UTF-8` 以及 `zh_HK.UTF-8 UTF-8`

將慾使用的 locale 反註釋並生成

```sh
locale-gen
```

**安裝必要程式、驅動程式以及 KDE Plasma**

```sh
pacman -Sy openssh htop wget iwd wireless_tools wpa_supplicant smartmontools xdg-utils plasma-meta plasma-workspace ark dolphin konsole egl-wayland pipewire pipewire-alsa pipewire-jack pipewire-pulse gst-plugin-pipewire libpulse wireplumber sddm
```

對於 Intel GPU 及 NVIDIA GPU 的 Hybrid 配置

```sh
pacman -S mesa intel-media-driver libva-intel-driver vulkan-intel xorg-server xorg-xinit nvidia-dkms
```

**配置 Unified Kernel Image**

>Arch Linux 預設使用 `mkinitcpio` 建立 initramfs 映像檔

建立 Unified Kernel Image 存放資料夾

```sh
mkdir -p /boot/EFI/Linux
```

變更 `mkinitcpio` 設定

```sh
nano /etc/mkinitcpio.conf
```

於已啟用的 HOOKS 預設檔案中加入 `lvm2`

變更内核包的映像檔建立設定

```sh
nano /etc/mkinitcpio.d/linux.preset
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

以建立 Unified Kernel Image

**安裝 systemd-boot 啟動載入器**

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

**安裝程式包**

>若您使用的是 EOS 的 Live ISO，請務必把 `/etc/pacman.conf` 中的 EOS Repository 刪除

執行
```sh
pacman -S fcitx5 fcitx5-chinese-addons kcm-fcitx5 fcitx5-qt fcitx5-gtk ttf-sarasa-gothic snapper snap-pac power-profiles-daemon
```
>注意：`power-profiles-daemon` 爲 Plasma 桌面的電源計劃管理程式，您也可以使用 `tlp` 對其進行替換，在進入新系統前請啟用服務

**為 NVIDIA GPU 加入 Kernel Parameters**

- 對於 grub 載入器

執行
```sh
nano /etc/default/grub
```

找到
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
```

把 `quiet` 改爲 `splash` 以便偵錯，並於後方加入 `nvidia.drm_modeset=1`

即
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 splash nvidia_drm.modeset=1"
```

儲存檔案變更

執行
```sh
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
以套用變更

>此處即使得 `grub` 套用變更並將最終的配置檔案輸出至 `/boot/grub/grub.cfg`

- 對於 systemd-boot 啟動的 UKI

```sh
nano /etc/cmdline.d/nvidia.conf
```

輸入

```
nvidia_drm.modeset=1
```

再次執行
```sh
mkinitcpio -P
```
以重新建立 UKI

**建立使用者賬戶**

執行
```sh
passwd
```
以變更 root 的密碼

執行
```sh
useradd -m -s /usr/bin/fish -G wheel 使用者名稱
```
以建立自己的使用者賬戶

> 此處使用 `useradd` 新增使用者賬戶，`-m` 即於 `/home` 建立該使用者的家目錄，`-s` 為選取登入的Shell，建議使用 `fish`，`-G` 即加入某個組

**允許使用者賬戶使用 `sudo`**

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
使用者名稱 ALL=(ALL:ALL) ALL
```

寫入該檔案並按原檔案名儲存

完成後，按下 `Control` 和 `D`，回到 Live ISO 環境

執行

```sh
reboot
```
以重新開機，並於開機中選擇 `UEFI OS (On 磁碟名稱)` 以啟動 Arch Linux

### 安裝後配置

#### 設定與 Windows 相同的時間標準

```sh
sudo timedatectl set-local-rtc 1 --adjust-system-clock
```

>注意：您也可以變更 Windows 的時間標準設定，此處不贅述

#### 啓用 `systemd` 的某些服務

`fstrim.timer` 計時器會在每周激活服務，在所有已掛載的支持 discard 操作的檔案系統上執行 fstrim，此舉是通知磁碟主控使用演算法來將寫入作業平衡到整塊閃存上

執行
```sh
sudo systemctl enable --now fstrim.timer
```

`nvidia-powerd.service` 是動態管理 NVIDIA GPU 電源狀態的服務，啓用以優化 NVIDIA GPU 電源使用狀況

執行
```sh
sudo systemctl enable --now nvidia-powerd.service
```

`nvidia-suspend.service` 是通知 NVIDIA GPU 您執行了 Suspend 的服務，啓用以實現 NVIDIA GNU/Linux Driver 的 Suspend

（可選）`nvidia-resume.service` 是通知 NVIDIA GPU Linux 內核正在恢復先前狀態的服務，若您無法從掛起中喚醒，可以嘗試啓用

>注意：Gnome 必須啓用該服務

執行
```sh
sudo systemctl enable nvidia-suspend.service
```

（可選）執行
```sh
sudo systemctl enable nvidia-resume.service
```

>注意：Gnome 必須啓用該服務

#### 加入 archlinuxcn 倉庫

由於 archlinuxcn 倉庫並不在官方所認可的範圍内，我們需要手動添加 archlinuxcn 倉庫及其位址。

執行
```sh
sudo nano /etc/pacman.conf
```
以使用 `nano` 編輯 `pacman` 的配置檔案

按下 `Control` 和 `End` 將光標移至檔案末尾，添加
```
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

>中國內地的使用者可以使用大陸鏡像源，例如使用先前所提及的校園網聯合鏡像，可改爲
```
[archlinuxcn]
Server = https://mirrors.cernet.edu.cn/archlinuxcn/$arch
#Server = https://repo.archlinuxcn.org/$arch
```

執行
```sh
sudo pacman-key --lsign-key "farseerfc@archlinux.org"
```
以在本機添加對 farseerfc 的密鑰的信任

> `pacman` 不會安裝來自不受信任的倉庫或維護者的包，所以需要於本機添加信任

隨即執行
```sh
sudo pacman -Sy archlinuxcn-keyring
```
以加入 archlinuxcn 倉庫的 PGP 密鑰

（可選）安裝 `archlinuxcn-mirrorlist-git` 以獲得一份 archlinuxcn 鏡像列表，以便于 `/etc/pacman.conf` 中直接引入

#### 啓用 AUR 倉庫 (Arch User Repository)

`paru` 是一個專注於功能的 AUR 包管理員

- 透過 archlinuxcn 倉庫安裝

執行
```sh
sudo pacman -S paru
```

- 同樣，您也可以自行安裝

執行
```sh
sudo pacman -S --needed base-devel git
```
以安裝依賴項目

執行
>建議於 `~` 下執行，若您日後需要歸檔自己或他人的項目，亦可以於您慾進行歸檔管理的目錄下執行
```sh
git clone https://aur.archlinux.org/paru.git
cd paru
```

編譯安裝 `paru`

執行
```sh
makepkg -si
```

> `makepkg` 是 Arch Linux 所使用的建立包的軟體，類似於 Gentoo Linux 的 `portage` 和 FreeBSD 的 `ports`

#### 使用 Snapper 對作業系統進行快照備份

**安裝 Snapper 及其附加程式**

執行
```sh
sudo pacman -S snapper snap-pac btrfs-assistant
```

>`snapper` 是由 openSUSE 的 Arvin Schnell 開發的程式，用於管理 Btrfs 檔案系統子卷與 LVM Thin-provisioned 卷快照。通過建立和比較快照在快照間回滾，且支援自動按時間序列建立快照。

>`snap-pac` 是一個 `pacman` 的挂鈎，用於在 `pacman` 執行前後觸發 `snapper` 建立快照

**配置 Snapper 對 `/` 進行快照備份**

執行 `btrfs-assistant`

> `btrfs-assistant` 是一個帶有圖形界面的 btrfs 管理工具，可以方便地管理 `snapper` 設定檔及 btrfs 子卷

- 於 *Snapper Settings* 欄建立對於 `/` 的設定檔
- 套用 `systemd` 的服務
- 變更 *Timeline* 自動快照的設定

設定 `snap-pac` 的 `pacman` 掛鉤

`snap-pac` 的配置檔案爲 `/etc/snap-pac.ini`

執行
```sh
sudo nano /etc/snap-pac.ini
```

請認真閱讀該檔案中的註釋並按需修改！

#### 啓用防火牆

`firewalld` 是一個動態管理的防火牆，支援使用區域來標識網路連綫與介面卡的可信等級。支援 IPv4、IPv6 防火牆設定、乙太網路橋接和 IP sets。可以使用分離的執行時配置和永久設定，也提供了一個接口用於直接為服務或程式加入防火牆規則。

**安裝 `firewalld`**

執行
```sh
sudo pacman -S firewalld
```

**啓用服務**
```sh
sudo systemctl enable --now firewalld
```

此時，可於 *System Settings > Wi-Fi & Networking > Firewall* 中存取防火牆設定

#### 啓用 `AppArmor`

`apparmor` 是內核的一個安全模塊，實現的功能與 `SELinux` 類似，管理每個程式的存取行爲

**安裝 `apparmor`**

執行
```sh
sudo pacman -S apparmor
```

**啓用服務**

執行
```sh
sudo systemctl enable apparmor
```

**要求內核啓動載入模塊**

- 使用 `grub` 啓動載入器
  
  編輯 `/etc/default/grub`

  於行
  ```
  GRUB_CMDLINE_LINUX_DEFAULT=
  ```

  後方的引號中加入
  ```
  lsm=lockdown,landlock,yama,integrity,apparmor,bpf
  ```

#### 關於 NVIDIA GPU 上的 Xorg 進程

若您需要把 Xorg Server 進程遷移至 Intel GPU，請於 `/etc/X11/xorg.conf.d/` 中建立設定檔案

例如：

執行
```sh
sudo nano /etc/X11/xorg.conf.d/20-force-intel.conf
```

寫入
```
Section "ServerFlags"
  Option "AutoAddGPU" "off"
EndSection
```

套用變更

**本手冊僅供新手參考，欲進行更多自訂設定，請自行探索，當然，別忘了 BTRFS 備份**

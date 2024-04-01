# Installation Guide for Arch Linux, An Arch Linux for EVERYONE!

## 1. 歸檔並燒錄映像檔

前往 [官方網站](https://endeavouros.com/#Download) 選擇鏡像並歸檔

使用 [Etcher](https://etcher.balena.io/) 燒錄映像檔至USB Disk

## 2. 變更UEFI BIOS設定並載入ISO

### ASUS:

顯示ASUS/ROG徽標時按下`Escape`開啓啓動選單，選擇`Enter UEFI Setup`。

按下`F7`進入進階選單，檢查「安全性」欄，關閉「安全啓動」或將類型變更爲`Other OS`或`Microsoft & 3rd Party`。

套用變更並重設電源，於啓動選單中選擇 `UEFI: <Your Device Name>` 以載入 `grub` 載入程式，選擇載入`Arch Linux install medium (x86_64, x64 UEFI)`，進入Live ISO環境。

## 3. 執行安裝作業前的準備

### 選取作業系統的鍵盤格式

>注意：此處以UTC+8以及English (US)鍵盤格式爲例。

欲選擇鍵盤格式，請執行
```sh
localectl list-keymaps
```
>注意：您可以使用pipe輸出至`grep`以篩選輸出結果，例如
```sh
localectl list-keymaps | grep us
```
預設的鍵盤格式是English (US)，請按需變更設定。

### 選取Console所使用的字體

在Hi-DPI熒幕上的tty可能顯示得非常小，此時可以變更字體

>注意：終端字體是儲存在`/usr/share/kbd/consolefonts/`的。

執行
```sh
setfont <欲選取的字體>
```
例如
```sh
setfont ter-132b
```

### 連接至Internet

欲檢查網路介面卡，執行
```sh
ip link
```
並檢查輸出項目中的`state`，UP即已啓用。

欲使用WLAN或WWAN（即無綫介面卡），請透過`rfkill`檢查介面卡的封鎖狀態。

執行
```sh
rfkill list
```
以檢查所有無綫介面卡

執行
```sh
rfkill enable <介面卡編號>
```

以`rfkill list`輸出結果如下爲例：
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
以解除對WLAN介面卡的封鎖。

此時，我們可以嘗試連接至Internet。

#### 對於乙太網路，請檢查綫纜是否插入LAN埠。

#### 對於WLAN，請使用iwctl進行認證。

例如：

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
以獲取WLAN SSID列表。

執行
```sh
[iwd] > station <Name> connect <SSID>
```
以選取欲連接的WLAN。

提示`Passphrase: `時輸入密碼即可連接。

#### 對於流動寬頻調解器，請使用mmcli。
>注意：鑒於流動寬頻調節器在當今相對罕見，此處便不再贅述。（附上[Wiki連結](https://wiki.archlinux.org/title/Mobile_broadband_modem#ModemManager)）

#### 現在，驗證是否有Internet存取。

執行
```sh
ping www.archlinux.org
```
通常，若輸出結果有`time=<latency> ms`即可存取至Arch Linux倉庫以完成安裝作業。

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
以列出時區，同樣的，使用pipe和`grep`以篩選輸出結果。

欲選取作業系統時區，請執行
```sh
timedatectl set-timezone <Timezone>
```
例如
```sh
timedatectl set-timezone Asia/Hong_Kong
```
此時選取的是HKT (UTC+8)

```sh
timedatectl set-timezone Asia/Chongqing
```
此時選取的是CST (UTC+8)

#### 至此，您已準備好執行安裝作業。

## 於Live ISO環境執行作業系統的安裝

>此處要强調的是“Live ISO”環境，使用Arch Linux Live ISO并不是强制條件。因爲在Arch Linux Live ISO中有時候條件不充分，例如高校學生欲使用要求身份認證的校園網執行安裝作業，此時利用Endeavour OS的Live ISO或許更好。

### 使用Arch Linux的`archinstall`脚本執行安裝

爲確保安裝作業順利進行，請使用最新版本的`archinstall`

執行
```sh
pacman -Sy archinstall
```
>`pacman`是Arch Linux所使用的包管理員，`-Sy`即要求與遠端（伺服器）進行同步化處理並重新整理本機資料庫，此處可簡單理解爲“重新整理本機資料庫並安裝`archinstall`”

`archinstall`是一個全自動的Arch Linux安裝脚本，提供了使用者以填寫問卷的形式執行安裝作業。

>注意：鑒於`archinstall`提供的磁碟分割工具較難使用，此處先使用`cfdisk`編輯磁碟分割表，再進入`archinstall`程式執行安裝

#### 準備磁碟

**首先，我們需要檢查磁碟裝置**
```sh
lsblk
```
此處注意檢查整個磁碟的大小以區分

>`lsblk`即“列出本機的塊裝置”

若您使用的是相同大小的甚至是相同型號的磁碟
請執行
```sh
fdisk -l
```
以仔細檢查

>`fdisk`是一個使用命令行使用者界面的磁碟分割表與分割區管理程式，此處是“列出所有裝置的分割表及其分割區”

**確定目標磁碟后，開始編輯磁碟分割表**

與終端中執行
```sh
cfdisk <目標磁碟>
```

>`cfdisk`是一個對使用者更友好的`fdisk`，其使用了Terminal User Interface（亦稱爲Curses-based User Interface），這樣更容易讓您清楚您在做什麽

例如
```sh
cfdisk /dev/sda
```
即可進入`cfdisk`實用程式以編輯`/dev/sda`裝置的磁碟分割表

>注意：由於Arch Linux會把vmlinuz與ucode鏡像檔存放於`/boot`，建議該分割不小於500M，若要使用多個内核，請按需分配

執行完分割表編輯后，以60G的磁碟爲例，所得結果可以是這樣的：
| `/dev/sda` | Size | Type |
|-|-|-|
`/dev/sda1` | 500M | EFI System |
`/dev/sda2` | 40G | Linux root (x86-64) |
`/dev/sda3` | 19.5G | Linux home |

**對磁碟分割執行初始化**

請再次執行`lsblk`以作參照

繼續先前例子，可以執行以下命令

```sh
mkfs.fat -F32 /dev/sda1 -n ARCH
mkfs.btrfs /dev/sda2 -f
mkfs.f2fs /dev/sda3 -f -l Home
```
>`mkfs`是用於在磁碟分割區中建立檔案系統的命令


**在使用`archinstall`執行安裝作業前，請檢查鏡像列表**

執行
```sh
nano /etc/pacman.d/mirrorlist
```

加入欲使用的鏡像伺服器，按下`Control`和`O`寫入，按下`Enter`以使用原本的檔案名，再按下`Control`和`X`退出

例如於頂部寫入
```
Server = https://mirrors.cernet.edu.cn/archlinux/$repo/os/$arch
```

欲進一步瞭解該鏡像，請前往其 [官方網站](https://mirrors.cernet.edu.cn/about)

**開始安裝**

>注意：於`archinstall`中，*Mirrors*欄目應選取您所在地區的鏡像伺服器

*Disk configuration*中應選取*Manual Partitioning*，按下`Space`選取您先前編輯過磁碟分割表的硬碟，再按下`Enter`確認

繼續以上述例子進行設定

| Status | Device | Size | FS type | Mountpoint | Mount options | Flags | Btrfs vol |
|-|-|-|-|-|-|-|-|
| existing |`/dev/sda1` | 524 MB | fat32 | `/boot` | | Boot, ESP | |
| modify | `/dev/sda2` | 42 GB | btrfs | | compress=zstd | | 2 subvolume |
| existing | `/dev/sda3` | 20 GB | f2fs | `/home` | | | |

對於btrfs子卷的設定，可以參考此表格

| name | mountpoint | compress | nodatacow |
|-|-|-|-|
`@` | `/` | True | False |
`@var` | `/var` | False | True |

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

對於*Additional packages*，可以加入：
`firefox` `nano` `fish`

選取*Install*以執行安裝

### 進入chroot環境進行配置

>若您使用的是EOS的Live ISO，請務必首先把`/etc/pacman.conf`中的EOS Repository刪除

**安裝程式包**

執行
```sh
pacman -S fcitx5 fcitx5-chinese-addons kcm-fcitx5 fcitx5-qt fcitx5-gtk ttf-sarasa-gothic snapper snap-pac power-profiles-daemon
```
>注意：`power-profiles-daemon`爲Plasma桌面的電源計劃管理程式，您也可以使用`tlp`對其進行替換

**為NVIDIA GPU加入Kernel Parameters**

執行
```sh
nano /etc/default/grub
```

找到
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
```

把`quiet`改爲`splash`以便偵錯，並於後方加入`nvidia.drm_modeset=1`

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

>此處即使得`grub`套用變更並將最終的配置檔案輸出至`/boot/grub/grub.cfg`

完成後，按下`Control`和`D`，回到Live ISO環境
執行
```sh
reboot
```
以重新開機

### 安裝後配置

#### 設定與Windows相同的時間標準

```sh
sudo timedatectl set-local-rtc 1 --adjust-system-clock
```

>注意：您也可以變更Windows的時間標準設定，此處不贅述

#### 啓用`systemd`的某些服務

`fstrim.timer`計時器會在每周激活服務，在所有已掛載的支持discard操作的檔案系統上執行fstrim，此舉是通知磁碟主控使用演算法來將寫入作業平衡到整塊閃存上

執行
```sh
sudo systemctl enable --now fstrim.timer
```

`nvidia-powerd.service`是動態管理NVIDIA GPU電源狀態的服務，啓用以優化NVIDIA GPU電源使用狀況

執行
```sh
sudo systemctl enable --now nvidia-powerd.service
```

`nvidia-suspend.service`是通知NVIDIA GPU您執行了Suspend To RAM的服務，啓用以實現NVIDIA GNU/Linux Driver的Suspend

（可選）`nvidia-resume.service`是通知NVIDIA GPU內核正在恢復先前狀態的服務，若您無法從掛起中喚醒，可以嘗試啓用

執行
```sh
sudo systemctl enable nvidia-suspend.service
```

（可選）執行
```sh
sudo systemctl enable nvidia-resume.service
```

#### 加入archlinuxcn倉庫

由於archlinuxcn倉庫並不在`pacman`所認可的範圍内，我們需要手動添加archlinuxcn倉庫及其位址。

執行
```sh
sudo nano /etc/pacman.conf
```
以使用`nano`編輯`pacman`的配置檔案

按下`Control`和`End`將光標移至檔案末尾，添加
```
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

>中國大陸使用者可以使用大陸鏡像源，例如使用先前所提及的校園網聯合鏡像，可改爲
```
[archlinuxcn]
Server = https://mirrors.cernet.edu.cn/archlinuxcn/$arch
#Server = https://repo.archlinuxcn.org/$arch
```

執行
```sh
sudo pacman-key --lsign-key "farseerfc@archlinux.org"
```
以在本機添加對farseerfc的密鑰的信任

隨即執行
```sh
sudo pacman -Sy archlinuxcn-keyring
```
以加入archlinuxcn倉庫的PGP密鑰

（可選）安裝`archlinuxcn-mirrorlist-git`以獲得一份archlinuxcn鏡像列表，以便于`/etc/pacman.conf`中直接引入

#### 啓用AUR倉庫

`paru`是一個專注於功能的AUR包管理員

- 透過archlinuxcn倉庫安裝

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
>建議於`~`下執行，若您日後需要歸檔自己或他人的`git`，亦可以於您慾進行歸檔管理的目錄下執行
```sh
git clone https://aur.archlinux.org/paru.git
cd paru
```

編譯安裝`paru`

執行
```sh
makepkg -si
```

#### 使用Snapper對作業系統進行快照備份

**安裝Snapper及其附加程式**

執行
```sh
sudo pacman -S snapper snap-pac btrfs-assistant
```

>`snapper`是由openSUSE的Arvin Schnell開發的程式，用於管理Btrfs檔案系統子卷與LVM Thin-provisioned卷。通過建立和比較快照在快照間回滾，且支援自動按時間序列建立快照。

>`snap-pac`是一個`pacman`的挂鈎，用於在`pacman`執行前後觸發`snapper`建立快照

**配置Snapper對`/`進行快照備份**

執行`btrfs-assistant`

- 於*Snapper Settings*欄建立對於`/`的設定檔
- 套用`systemd`的服務
- 變更*Timeline*自動快照的設定

>顧名思義，`btrfs-assistant`是一個擁有圖形使用者界面的Btrfs檔案系統管理員

設定`snap-pac`的`pacman`掛鉤

`snap-pac`的配置檔案爲`/etc/snap-pac.ini`

執行
```sh
sudo nano /etc/snap-pac.ini
```

請認真閱讀該檔案中的註釋並按需修改！

#### 啓用防火牆

`firewalld`是一個動態管理的防火牆，支援使用區域來標識網路連綫與介面卡的可信等級。支援IPv4、IPv6防火牆設定、乙太網路橋接和IP sets。可以使用分離的執行時配置和永久設定，也提供了一個接口用於直接為服務或程式加入防火牆規則。

**安裝`firewalld`**

執行
```sh
sudo pacman -S firewalld
```

**啓用服務**
```sh
sudo systemctl enable --now firewalld
```

此時，可於 *System Settings > Wi-Fi & Networking > Firewall* 中存取防火牆設定

#### 啓用`AppArmor`

`apparmor`是內核的一個安全模塊，實現的功能與`SELinux`類似，管理每個程式的存取行爲

**安裝`apparmor`**

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

- 使用`grub`啓動載入器
  
  編輯`/etc/default/grub`

  於行
  ```
  GRUB_CMDLINE_LINUX_DEFAULT=
  ```

  後方的引號中加入
  ```
  lsm=landlock,yama,integrity,apparmor,bpf
  ```

- 不啓用內核Lockdown功能是因爲使用了閉源的NVIDIA GNU/Linux Driver

#### 關於NVIDIA GPU上的Xorg進程

若您需要把Xorg Server進程遷移至Intel GPU，請於`/etc/X11/xorg.conf.d/`中建立設定檔案

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

**本手冊僅供新手參考，欲進行更多自訂設定，請自行探索，當然，別忘了BTRFS備份**
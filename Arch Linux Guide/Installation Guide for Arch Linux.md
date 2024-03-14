# Installation Guide for Arch Linux, An Arch Linux for EVERYONE!

# TO BE ACCOMPLISHED

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

`archinstall`是一個全自動的Arch Linux安裝脚本，提供了使用者以填寫問卷的形式執行安裝作業。

>注意：鑒於`archinstall`提供的磁碟分割工具較難上手，此處先使用`cfdisk`編輯磁碟分割表，再進入`archinstall`程式執行安裝

#### 準備磁碟

**首先，我們需要檢查磁碟裝置**
```sh
lsblk
```
此處注意檢查整個磁碟的大小以區分

若您使用的是相同大小的甚至是相同型號的磁碟
請執行
```sh
fdisk -l
```
以仔細檢查

**確定目標磁碟后，開始編輯磁碟分割表**

與終端中執行
```sh
cfdisk <目標磁碟>
```

例如
```sh
cfdisk /dev/sda
```
即可進入`cfdisk`實用程式以編輯`/dev/sda`裝置的磁碟分割表

`cfdisk`是`fdisk`的Terminal UI版本，這樣可以讓您清楚您在做什麽

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

**在使用`archinstall`執行安裝作業前，請檢查鏡像列表**

執行
```sh
nano /etc/pacman.d/mirrorlist
```

加入欲使用的鏡像伺服器，按下`Control`和`O`寫入，按下`Enter`以使用原本的檔案名，再按下`Control`和`X`退出

**開始安裝**

>注意：於`archinstall`中，*Mirrors*欄目應選取您所在地區的鏡像伺服器

*Disk configuration*中應選取*Manual Partitioning*，按下`Space`選取您先前編輯過磁碟分割表的硬碟，再按下`Enter`確認

可以以此為例進行設定

| Status | Device | Size | FS type | Mountpoint | Mount options | Flags | Btrfs vol |
|-|-|-|-|-|-|-|-|
| existing |`/dev/sda1` | 524 MB | fat32 | `/boot` | | Boot, ESP | |
| modify | `/dev/sda2` | 42 GB | btrfs | | compress=zstd | | 2 subvolume |
| existing | `/dev/sda3` | 20 GB | f2fs | `/home` | | | |

對於btrfs子卷的設定，可以參考此表格

| name | mountpoint | compress | nodatacow|
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

>若您使用的是EOS的Live ISO，請把`/etc/pacman.conf`中的EOS Repository刪除

**安裝程式包**

執行
```sh
pacman -S fcitx5 fcitx5-chinese-addons kcm-fcitx5 fcitx5-qt fcitx5-gtk ttf-sarasa-gothic snapper snap-pac
```

**為NVIDIA GPU加入Kernel Parameters**

執行
```sh
nano /etc/default/grub
```

找到
```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"
```

把`quiet`改爲`splash`，並於後方加入`nvidia.drm_modeset=1`

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

完成後，按下`Control`和`D`，回到Live ISO環境
執行
```sh
reboot
```
以重新開機

### 安裝後配置


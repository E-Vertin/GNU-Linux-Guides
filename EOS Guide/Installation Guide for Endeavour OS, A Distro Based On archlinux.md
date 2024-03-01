# Installation Guide for *Endeavour OS*, A Distro Based On `archlinux`

## 1. 歸檔並燒錄映像檔

前往 [官方網站](https://endeavouros.com/#Download) 選擇鏡像並歸檔

使用 [Etcher](https://etcher.balena.io/) 燒錄映像檔至USB Disk

## 2. 變更UEFI BIOS設定並載入ISO

### ASUS:

顯示ASUS/ROG徽標時按下`Escape`開啓啓動選單，選擇`Enter UEFI Setup`。

按下`F7`進入進階選單，檢查「安全性」欄，關閉「安全啓動」或將類型變更爲`Other OS`或`Microsoft & 3rd Party`。

套用變更並重設電源，於啓動選單中選擇 `UEFI: <Your Device Name>` 以載入 `systemd-boot` 載入程式，選擇載入專有驅動程式（NVIDIA Cards），進入Live ISO環境。

## 3. 配置網路界面卡並選擇鏡像

連綫至WLAN或使用乙太網路，確保可存取Internet。

- 自動選擇鏡像，可執行
    ```sh
    sudo reflector --sort rate --threads 100 -c China --save /etc/pacman.d/mirrorlist
    ```
- 手動添加鏡像，可編輯`/etc/pacman.d/mirrorlist`\
（例子）於頂端添加該行：
    ```
    Server = http://mirrors.aliyun.com/archlinux/$repo/os/$arch
    ```
- （可選）變更`/etc/pacman.conf`中的`ParallelDownloads`值為`8`，預設為`5`

## 4. 使用安裝程式安裝作業系統

- 於*eos-welcome*程式中選取*Start the Installer*，選取*Online*，進入安裝程式。
- 根據自身情況選擇語言、時區、鍵盤格式。
- 桌面環境建議選擇*KDE Plasma*或*GNOME* （此處以*Plasma*爲例）
- 預先安裝的程式包套用預設即可，或加入*Printing Support*
- 載入程式建議使用`grub`，否則無法啓動至btrfs快照
- 磁碟分割可以參考附錄表
- 建立使用者賬戶
- 確認安裝作業系統並重新開機

## 5. 登入作業系統並進行配置

### 校準硬體時間

此處使得Linux使用Windows時間寫入標準

```sh
sudo timedatectl set-local-rtc 1 -–adjust-system-clock
```

### 加入archlinuxcn倉庫

**“archlinuxcn倉庫是由Arch Linux中文社區管理的第三者倉庫，其提供了許多額外的程式包，也包含了許多程式的git版本及其變種。一部分來源於AUR，但也有許多與AUR的不同。”**

由於archlinuxcn倉庫並不在`pacman`所認可的範圍内，我們需要手動添加archlinuxcn倉庫及其位址。

執行
```sh
sudo nano /etc/pacman.conf
```
以使用`nano`編輯`pacman`的配置檔案

按下Control和End將光標移至檔案末尾，添加
```sh
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

隨即執行
```sh
sudo pacman -S archlinuxcn-keyring
```
以加入archlinuxcn倉庫的PGP密鑰

再執行
```sh
sudo pacman -Syy
```
以强制重新整理本機數據庫

（可選）安裝`archlinuxcn-mirrorlist-git`以獲得一份archlinuxcn鏡像列表，以便于`/etc/pacman.conf`中直接引入

### 安裝必備程式包（透過`pacman`），執行：

```sh
sudo pacman -S plasma-wayland-session fcitx5 fcitx5-qt fcitx5-gtk fcitx5-chinese-addons kcm-fcitx5 fcitx5-material-color grub-btrfs inotify-tools htop nvtop yay
```

### 配置btrfs文件系統快照備份（使用*Timeshift*）

`yay`可以存取*Arch User Repositories*，提供了更多的軟體

#### 安裝*Timeshift*

```sh
yay -S timeshift timeshift-autosnap
```

#### 配置`grub`啓動項目，可以使用`grub-btrfs`守護程式

- 變更`grub-btrfsd`設定：
  ```sh
  sudo systemctl edit –-full grub-btrfsd
  ```
  找到
  ```sh
  ExecStart = /usr/bin/grub-btrfsd –-syslog /.snapshots
  ```
  改爲
  ```sh
  ExecStart = /usr/bin/grub-btrfsd –-syslog -t
  ```
- 套用變更后，啓用該服務並檢查狀況：
  ```sh
  sudo systemctl enable grub-btrfsd --now
  sudo systemctl status grub-btrfsd
  ```
- 套用`grub`設定以自動重新建立啓動選單：
  ```sh
  sudo grub-mkconfig -o /boot/grub/grub.cfg
  ```

  >注意：由於Arch Linux滾動更新的特性，Linux内核通常是緊隨Linus Torvalds的更新步調，而更新内核后會出現`grub`啓動選單沒有`Endeavour OS Snapshots`的選項，此時再次執行上述套用設定的命令即可。

#### 配置*Timeshift*

執行`Timeshift`，快照類型選取*BTRFS*，位置套用預設即可，自動建立快照可以勾選*Daily*和*Boot*，例子中`/home`已是另一個分割區，因此無需涵蓋`@home`子卷。

#### 使用*Timeshift*定時快照

*Timeshift*定時快照依賴於`cron`，而這并非是預設啓用的。

執行
```sh
sudo systemctl enable cronie --now
```

#### 嘗試建立一個快照並檢查`grub-btrfs` 守護進程的狀況

於*Timeshift*中建立一個快照，注釋為`Setup`，退出*Timeshift*

執行
```sh
sudo systemctl status grub-btrfsd
```

檢查輸出日志，若有`Grub menu recreated`即配置成功，下次重新開機即可於`grub`選單中選擇`Endeavour OS Snapshots`啓動到唯讀快照。

### 重新開機並配置fcitx5

於`sddm`登入畫面左下角選擇*Plasma* (*Wayland*)並登入

根據提示，應於*System Settings > Input Devices > Virtual Keyboard*中選擇*Fcitx 5 Wayland Launcher*

於*System Settings > Regional Settings > Input Method*中加入欲使用的輸入方法

## 附錄：以60 GB的硬碟爲例建立的磁碟分割表

| `/dev/sda` | Size | File System | Mount Point | Flags |
|-|-|-|-|-|
`/dev/sda1` | 300 MB | FAT32(VFAT) | `/boot/efi` | `boot` |
`/dev/sda2` | 40,960 MB | BTRFS | `/` | `root` |
`/dev/sda3` | 20,175 MB | XFS | `/home`	 |  |

> 注意：請勿將 `grub` 載入器安裝於 *Windows Boot Manager* 的 `ESP` 分割，否則更新 *Windows* 重建 *Windows Boot Manager* 時會損毀 `grub` 載入器

**本手冊僅供新手參考，欲進行更多自訂設定，請自行探索，當然，別忘了BTRFS備份。**
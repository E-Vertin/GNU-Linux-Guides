# Arch 使用贴士

## 設定與 Windows 相同的時間標準 (本地時間作爲硬體時間)

```sh
sudo timedatectl set-local-rtc 1 --adjust-system-clock
```

> 注意：您也可以變更 Windows 的時間標準設定，此處不贅述

## 啓用 `systemd` 的某些服務

`fstrim.timer` 計時器會在每周激活服務，在所有已掛載的支持 discard 操作的檔案系統上執行 fstrim，此舉是通知磁碟主控使用演算法來將寫入作業平衡到整塊閃存上

執行

```sh
sudo systemctl enable --now fstrim.timer
```

`nvidia-powerd.service` 是動態管理 NVIDIA GPU 電源狀態的服務，啓用以優化 NVIDIA GPU 電源使用狀況 (僅適用於筆記本電腦)

執行

```sh
sudo systemctl enable --now nvidia-powerd.service
```

## 加入 archlinuxcn 倉庫

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

> 中國內地的使用者可以使用大陸鏡像源，例如使用先前所提及的校園網聯合鏡像，可改爲

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

## 啓用 AUR 倉庫 (Arch Users' Repository)

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

> 建議於 `~` 下執行

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

## 使用 Snapper 對作業系統進行快照備份

**安裝 Snapper 及附加程式**

執行

```sh
sudo pacman -S snapper snap-pac btrfs-assistant
```

>`snapper` 是由 openSUSE 的 Arvin Schnell 開發的程式，用於管理 Btrfs 檔案系統子卷與 LVM Thin-provisioned 卷快照。通過建立和比較快照在快照間回滾，且支援自動按時間序列建立快照。

>`snap-pac` 是一個 `pacman` 的腳本，用於在 `pacman` 執行前後觸發 `snapper` 建立快照

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

## 啓用防火牆

`firewalld` 是一個動態管理的防火牆，支援使用“區域”來標識網路連綫與介面卡的可信等級。支援 IPv4、IPv6 防火牆設定、乙太網路橋接和 IP 組。可以使用分離的執行時配置和永久設定，也提供了一個接口用於直接為服務或程式加入防火牆規則。

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

## 啓用 `AppArmor`

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

- 使用傳統的 initramfs 與內核映像分離配置

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

  - 使用 `systemd-boot` 啓動載入器

    編輯 `/<ESP>/loader/entries/<啓動項>`

    於 `option` 後加入 `lsm=lockdown,landlock,yama,integrity,apparmor,bpf`

- 使用 UKI

  若使用 `mkinitcpio` 建立 EFI 可執行的 UKI

  於 `/etc/cmdline.d` 中加入 `.conf` 純文字檔案並寫入 `lsm=lockdown,landlock,yama,integrity,apparmor,bpf` 即可

## 關於筆電的 NVIDIA GPU 上的 Xorg 進程

若您需要把 Xorg Server 進程遷移至 Intel GPU 以關閉 NVIDIA GPU，請於 `/etc/X11/xorg.conf.d/` 中建立設定檔案

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

**本手冊僅供參考，欲進行更多自訂設定，請自行探索**

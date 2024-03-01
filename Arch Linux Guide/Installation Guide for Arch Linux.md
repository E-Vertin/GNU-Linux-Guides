# Installation Guide for Arch Linux, A Meta Linux Distro

# TO BE ACCOMPLISHED

## 1. 歸檔並燒錄映像檔

前往 [官方網站](https://archlinux.org/download/) 進行歸檔或選擇鏡像

使用 *Etcher* 燒錄映像檔至USB Disk

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

### 使用Arch Linux Live ISO中的`archinstall`脚本執行安裝

`archinstall`是一個全自動的Arch Linux安裝脚本，提供了使用者以填寫問卷的形式執行安裝作業。


# Instructions For Enabling Secure Boot via `Shim` On Gentoo Linux

## 前言

在 Gentoo Linux 上啓用 Secure Boot 與在 Arch Linux 上執行的操作有相似之處

目前有兩類方法在 GNU/Linux 作業系統上啓用 Secure Boot

- 使用自己的簽章給 Bootloader 和內核以及模塊簽名

- 使用預先簽名的 Bootloader 並使用 Machine Owner Key 給內核及模塊簽名

鑑於第一種方式需要讓主機板進入 Secure Boot Setup Mode，相對於第二種而言風險更大也更麻煩，本文使用第二種方法藉助 `sys-boot/shim` 配合 `systemd-boot` 實現 Secure Boot

> 本文需要用到的包：`sys-boot/shim` `sys-boot/mokutil` `app-crypt/sbsigntools` `dev-libs/openssl` `sys-boot/efibootmgr`

## 1. 建立 PEM 私鑰與簽章

執行
```sh
openssl req -newkey rsa:2048 -nodes -keyout priv-key.key -new -x509 -sha256 -days 3650 -subj "/CN=<你的暱稱> Machine Owner Key/" -out cert.crt
```

請妥善保管，此爲用於簽名 Bootloader 以及內核和模塊的私鑰與簽章

## 2. 建立 Machine Owner Key Manager 可以使用的 DER 密鑰

執行
```sh
openssl x509 -outform DER -in MOK.crt -out MOK.cer
```

請將此密鑰妥善儲存至 EFI System Partition 以便 MOK Manager 存取

## 3. 變更 `/etc/portage/make.conf` 設定以全域啓用 Secure Boot 與自動簽名

執行
```sh
nano /etc/portage/make.conf
```
以編輯

加入
```
USE="secureboot modules-sign"

SECUREBOOT_SIGN_KEY=<私鑰的絕對位址>
SECUREBOOT_SIGN_CERT=<簽章的絕對位址>

MODULES_SIGN_KEY=<私鑰的絕對位址>
MODULES_SIGN_CERT=<簽章的絕對位址>
```

保存並退出 `nano`

執行
```sh
emerge -auDN @world
```

以套用變更

> 此舉 `portage` 將會自動爲已安裝的所有 Distribution Kernel（例如 `sys-kernel/gentoo-kernel-bin`）, 3rd-Party Modules（例如 `x11-drivers/nvidia-drivers`） 以及 Bootloader 簽名

## 4. 部署 `shim` 與 `systemd-boot`

### 安裝 `sys-boot/shim`

執行
```sh
emerge -a sys-boot/shim
```
以安裝

### 重新部署 `systemd-boot`

執行
```sh
bootctl install --no-variables
```

在 `systemd-boot` 資料夾中加入 `shim`

執行
```sh
cp /usr/share/shim/BOOTX64.EFI /<你的 ESP 分割區>/EFI/systemd/shimx64.efi
cp /usr/share/shim/mmx64.efi /<你的 ESP 分割區>/EFI/systemd/mmx64.efi
```

### 建立啓動項目

執行
```sh
efibootmgr --disk /dev/<你的 ESP 分割區所在磁碟> --part <第幾個分割> --create --label "Systemd-boot via Shim" --loader '\EFI\systemd\shimx64.efi' --unicode '\EFI\systemd\systemd-bootx64.efi'
```

> 判斷 ESP 分割區所在磁碟及其分割序號，可以執行
> ```sh
> lsblk
> ```
>
> 我的輸出是這樣的
> ```
> nvme1n1     259:7    0   1.9T  0 disk 
>   ├─nvme1n1p1 259:8    0    16M  0 part 
>   ├─nvme1n1p2 259:9    0 953.9G  0 part 
>   ├─nvme1n1p3 259:10   0     1G  0 part /boot
>   ├─nvme1n1p4 259:11   0   100G  0 part /
>   ├─nvme1n1p5 259:12   0    80G  0 part /home
> ```
> 則應該執行
> ```sh
> efibootmgr --disk /dev/nvme1n1 --part 3 --create --label "Systemd-boot via Shim" --loader '\EFI\systemd\shimx64.efi' --unicode '\EFI\systemd\systemd-bootx64.efi'
> ```

## 5. 使用 `dracut` 自動爲內核映像簽名

### 修改 `/etc/dracut.conf`

確保
```
uefi="yes"
```
以使用 Unified Kernel Image

加入
```
uefi_secureboot_cert="<簽章的絕對位址>"
uefi_secureboot_key="<私鑰的絕對位址>"
```

### 重新建立 UKI

對於 Distribution Kernel，有兩種方式重新建立 UKI

- 使用 `portage`
  
  執行
  ```sh
  emerge --config gentoo-kernel-bin
  ```
  > 若使用的是 `sys-kernel/gentoo-kernel` 則相應替換

- 使用 `dracut` 直接建立
  
  執行
  ```sh
  eselect kernel list
  ```
  以檢視當前 `/usr/src/linux` 這個 Symbolic Link 的指向

  使用
  ```sh
  eselect kernel set <序號>
  ```
  以重新建立 Symbolic Link 以選中內核

  執行
  ```sh
  dracut
  ```
  以爲當前所選內核重新建立 UKI

> 注意：若需要手動簽名，可以執行
> ```sh
> sbsign --key <私鑰的絕對位址> --cert <簽章的絕對位址> --output /<ESP 分割區>/EFI/Linux/<UKI 檔案名>.efi /<ESP 分割區>/EFI/Linux/<UKI 檔案名>.efi
> ```

## 6. 於 Machine Owner Key Manager 錄入自行建立的密鑰

> 請確保先前建立的 DER 密鑰已妥善儲存至 EFI System Partition 以便 MOK Manager 存取

- 重啓計算機

- 於 UEFI BIOS 中開啓 Secure Boot

- 於啓動選項中選擇 `Systemd-boot via Shim`

  > 此時應該會進入 MOK Manager

- 選擇 `Enroll key` 並選擇先前建立的 DER 密鑰

- 選擇 `Reboot`

- 於啓動選項中選擇 `Systemd-boot via Shim` 後應該會見到 `systemd-boot` 介面，選擇先前簽名的 UKI 並嘗試啓動

## 7. 檢查 Secure Boot 狀態與第三方內核模塊

- 檢查 Secure Boot 狀態
  
  執行
  ```sh
  dmesg | grep Secure
  ```
  > 我的輸出是這樣的
  > ```
  > [    0.009330] Secure boot enabled
  > ```

- 檢查第三方內核模塊（以 NVIDIA Unix/Linux Driver 爲例）
  
  執行
  ```sh
  nvidia-smi
  ```
  有表格類型輸出則成功載入


**本文中實現的 Secure Boot 的安全性依完全賴於您的 UEFI BIOS Passcode 的強度**

**啓用 Secure Boot 其實並不會爲您的電腦的安全性帶來實質性的提升，僅僅只是爲了不影響 Windows 的功能**
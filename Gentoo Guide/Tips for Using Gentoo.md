# Tips for Using Gentoo Linux

## 前言

Gentoo Linux 的 `portage` 包管理員以其靈活程度與精細化控制著稱，這一功能主要通過 `USE Flags` 實現，但這也帶來了一些“不變”，即並非所有的包都預設啓用其所有功能

因此，本文主要圍繞某些包的 `USE Flags` 並針對一般使用情境下提出建議

> 本文用到的工具：`app-portage/eix`
>
> 本文對於包名的格式：`<類別>/<包名>::<倉庫>`

### 檢視包的 `USE Flags`

執行
```sh
sudo eix -s <包名>
```

以檢視某個包提供的所有 `USE Flags`

### 套用 `USE Flags` 的變更

照常更新整個系統即可，執行

```sh
sudo emerge -auDN @world
```

> 同樣地，可以加入 `-g` 以使用二進制包加快安裝速度


## Microsoft Visual Studio Code

> `app-editors/vscode::gentoo`

Gentoo 倉庫提供的 VS Code 有如下 `USE Flags`：

- `egl`   Use EGL platform, enables smooth rending in high refresh rate monitors on X11/Xwayland.
- `wayland`   Run in wayland mode under wayland sessions, xwayland otherwise. This flag doesn't affect x11 sessions. 
- `kerberos`  Add kerberos support.
 
通常情況下，我們只需要啓用 `egl` 和 `wayland`

建立 `/etc/portage/package.use/vscode` 並寫入

```
app-editors/vscode egl wayland
```

> 若已經於 `/etc/portage/make.conf` 中全域啓用 `wayland` 的 `USE Flags`，則不需要加入該 `USE Flags`

## Libre Office 及其多語言設定

> `app-office/libreoffice-bin::gentoo` 或 `app-office/libreoffice::gentoo`，出於簡便，本文只以前者爲例

Gentoo 倉庫提供的 Libre Office 二進制包有如下 `USE Flags`：

- `gnome`   Add GNOME support.
- `kde` Add support for software made by KDE, a free software community.
- `java`    Add support for Java.
  
通常情況下，Profile 與 `/etc/portage/make.conf` 已全域啓用 `gnome` 或 `kde`

> `app-office/libreoffice-l10n::gentoo`

Gentoo 倉庫提供的多語言包的 `USE Flags` 爲 Libre Office 所有支援的語言，並以 `USE_EXPAND Flags` 呈現：

> “L10N” 爲英文 “Localisation” 的“簡寫”，同理 “I18N” 爲英文 “Internationalisation” 的“簡寫”

通常情況下，使用者可用在 `/etc/portage/make.conf` 中全域啓用自己需要的語言

例如

```
L10N="en-US zh-CN"
```

也可以爲某個包單獨設定

在 `/etc/portage/libreoffice` 中寫入

```
app-office/libreoffice-l10n l10n_en-US l10n_zh-CN
```

## QEMU

> `app-emulation/qemu::gentoo` 

Gentoo 倉庫提供的 QEMU 除了常規的 `USE Flags` 還提供了 `PYTHON_TARGETS`, `QEMU_SOFTMMU_TARGETS`, `QEMU_USER_TARGETS` 的 `USE_EXPAND Flags`

> `PYTHON_TARGETS`   Controls support for various Python implementations (versions) in packages. These essentially control what version of Python the package will reference during and after installation.
> 
> `QEMU_SOFTMMU_TARGETS`     The standard qemu use-case of emulating an entire system (like VirtualBox or VMWare, but with optional support for emulating CPU hardware along with peripherals).
> 
> `QEMU_USER_TARGETS`   Executes user-mode code only; the (somewhat shockingly ambitious) purpose of these targets is to "magically" allow importing user-space linux ELF binaries from a different architecture into the native system (like multilib, without the awkward need for a software stack or CPU capable of running it).

簡單地說，`QEMU_SOFTMMU_TARGETS` 就是虛擬機要模擬的架構，而 `QEMU_USER_TARGETS` 是宿主機的架構，`PYTHON_TARGETS` 則控制了該包取用的 Python 版本

例如宿主機爲 x86_64 的架構，並且想要模擬 ARM 和 x86_64 架構

在 `/etc/portage/package.use/qemu` 中寫入

```
app-emulation/qemu QEMU_SOFTMMU_TARGETS: arm x86_64 QEMU_USER_TARGETS: x86_64
```

即可

## OBS Studio

> `media-video/obs-studio::gentoo`

Gentoo 倉庫中提供的 OBS Studio 並不會預設啓用硬體編碼

對於 NVIDIA GPU 慾使用 NVENC，於 `/etc/portage/package.use/obs` 寫入

```
media-video/obs-studio nvenc
```

即可

(TO BE CONTINUED)
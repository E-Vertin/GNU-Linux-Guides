# Tips for Using Gentoo Linux

## Preface

Gentoo Linux's `portage` package manager is renowned for its flexibility and fine-grained control, primarily achieved through `USE Flags`. However, this also leads to some "inconveniences," as not all packages have all features enabled by default.

Therefore, this article focuses on the `USE Flags` of certain packages and provides recommendations for general usage scenarios.

> The tools used in this article: `app-portage/eix`
>
> The format for package names in this article: `<category>/<package>::<repository>`

### Inspecting Package `USE Flags`

Run the following command to view all `USE Flags` provided by a specific package:

```bash
eix -s <package-name>
```

### Applying Changes to `USE Flags`

To apply changes to `USE Flags`, simply update the entire system by executing:

```bash
emerge -auDN @world
```

> Similarly, you can add `-g` to use binary packages for faster installation.


## Microsoft Visual Studio Code

> `app-editors/vscode::gentoo`

The Gentoo official repository provides the following `USE Flags` for VS Code: 

- `egl`   Use EGL platform, enables smooth rending in high refresh rate monitors on X11/Xwayland.
- `wayland`   Run in wayland mode under wayland sessions, xwayland otherwise. This flag doesn't affect x11 sessions. 
- `kerberos`  Add kerberos support.

Typically, we only need to enable `egl` and `wayland`.

Create the file `/etc/portage/package.use/vscode` and add the following lines:

```
app-editors/vscode egl wayland
```

> If you have already globally enabled the `wayland` `USE Flag` in `/etc/portage/make.conf`, you do not need to add this here again.

## Libre Office and Its Multilingual Settings

> `app-office/libreoffice-bin::gentoo` or `app-office/libreoffice::gentoo`, for simplicity, this article will refer to the former.

The Gentoo official repository provides the following `USE Flags` for the Libre Office binary package:

- `gnome`   Add GNOME support.
- `kde` Add support for software made by KDE, a free software community.
- `java`    Add support for Java.

Typically, our profile and global configuration have already enabled `gnome` or `kde`, so we only need to consider whether to enable `java` or not.

The Gentoo repository provides `USE Flags` for all supported languages of Libre Office, presented as `USE_EXPAND Flags`:

> "L10N" stands for "Localisation," and "I18N" stands for "Internationalisation."

For the most commonly scenario, users can specify the languages they need in `/etc/portage/make.conf`:

```
L10N="en-US zh-CN"
```

Also, you can specify the languages for each package.

For example, create the file `/etc/portage/package.use/libreoffice` and add:

```
app-office/libreoffice-bin L10N: en-US zh-CN
```

## QEMU

> `app-emulation/qemu::gentoo`

The Gentoo repository provides not only `USE Flags`, but `USE_EXPAND Flags` such as `PYTHON_TARGETS`, `QEMU_SOFTMMU_TARGETS` and `QEMU_USER_TARGETS` for QEMU:

> `PYTHON_TARGETS`   Controls support for various Python implementations (versions) in packages. These essentially control what version of Python the package will reference during and after installation.
> 
> `QEMU_SOFTMMU_TARGETS`     The standard qemu use-case of emulating an entire system (like VirtualBox or VMWare, but with optional support for emulating CPU hardware along with peripherals).
> 
> `QEMU_USER_TARGETS`   Executes user-mode code only; the (somewhat shockingly ambitious) purpose of these targets is to "magically" allow importing user-space linux ELF binaries from a different architecture into the native system (like multilib, without the awkward need for a software stack or CPU capable of running it).

In short, `PYTHON_TARGETS` controls the Python version, `QEMU_SOFTMMU_TARGETS` controls the system emulation targets, and `QEMU_USER_TARGETS` controls the user-mode emulation targets.


Take emulating an amd64 and aarch64 system on an amd64 machine as an example, you can set the `USE Flags` in `/etc/portage/package.use/qemu` as follows:

```
app-emulation/qemu QEMU_SOFTMMU_TARGETS: arm x86_64 QEMU_USER_TARGETS: x86_64
```

## OBS Studio

> `media-video/obs-studio::gentoo`


OBS Studio installed from the Gentoo repository does not support hardware encoding by default, which is a common requirement for streamers.


To enable hardware encoding for NVIDIA GPUs, you need to add the `nvenc` `USE Flag` for OBS Studio in `/etc/portage/package.use/obs`.

```
media-video/obs-studio nvenc
```

(TO BE CONTINUED)
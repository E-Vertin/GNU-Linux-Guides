# Instructions For Enabling Secure Boot via `Shim` On Gentoo Linux

## Preface

Enabling Secure Boot on Gentoo Linux is similar to doing so on Arch Linux.

There are two approaches to enabling Secure Boot for GNU/Linux:

- Use your own keys and certificates to sign bootloaders, kernel images, and kernel modules

- Use a pre-signed bootloader and sign the kernel and modules with a Machine Owner Key (MOK)

Given that the former approach is more complex—requiring you to manipulate UEFI BIOS settings to import your own keys and certificates—this guide will focus on the latter method using `Shim` and `systemd-boot`.

> Necessary packages: `sys-boot/shim` `sys-boot/mokutil` `app-crypt/sbsigntools` `dev-libs/openssl` `sys-boot/efibootmgr`

## 1. Create PEM Private Key and Certificate

Execute the following command to generate a private key named `priv-key.key` and a self-signed certificate named `cert.crt`:

```bash
openssl req -newkey rsa:2048 -nodes -keyout priv-key.key -new -x509 -sha256 -days 3650 -subj "/CN=<your-name> Machine Owner Key/" -out cert.crt
```

> Remember to replace `<your-name>` with your desired identifier.

Please keep this key and certificate secure, as they will be used to sign the bootloader, kernel, and modules.

## 2. Create DER Format Key for MOK Manager

Execute the following command to convert the certificate to DER format and name it as `MOK.cer`, which is required for MOK Manager:

```bash
openssl x509 -outform DER -in cert.crt -out MOK.cer
```

Please store this key securely in the EFI System Partition for MOK Manager access.

## 3. Modify `/etc/portage/make.conf` to Globally Enable Secure Boot and Module Signing

Edit the `/etc/portage/make.conf` file:

```bash
nano /etc/portage/make.conf
```

Add the following lines to enable Secure Boot and module signing globally:

```
USE="${USE} secureboot modules-sign"

SECUREBOOT_SIGN_KEY=<path-to-your-private-key>
SECUREBOOT_SIGN_CERT=<path-to-your-certificate>

MODULES_SIGN_KEY=<path-to-your-private-key>
MODULES_SIGN_CERT=<path-to-your-certificate>
```

Save and exit the editor (in `nano`, press `CTRL+X`, then `Y`, and `Enter`).

After modifying the configuration, update your system to apply the changes:

```bash
emerge -auDN @world
```

> This will ensure that all supported packages are rebuilt with Secure Boot and module signing enabled.

**Please note that you should add your own certificates to your custom kernel.**

- For `.config`, you can add the following lines to specify the signing key and certificate:

  ```bash
  CONFIG_SYSTEM_TRUSTED_KEYS="<path-to-your-certificate>"
  ```

- For `make menuconfig`, navigate to the following options:

  ```
  Cryptographic API
    --->  Certificates for signature checking
      --->  () Additional X.509 keys for default system keyring
  ```

    Here, you can specify the path to your certificate.

    `(<path-to-your-certificate>) Additional X.509 keys for default system keyring`

## 4. Deploy `shim` and `systemd-boot`

### Install `sys-boot/shim`

To install `sys-boot/shim`, execute the following command:

```bash
emerge -a sys-boot/shim
```

### Redeploy `systemd-boot`

You can redeploy `systemd-boot` by executing:

```bash
bootctl install --no-variables
```

Now, copy `shim` to the EFI System Partition:

```bash
cp /usr/share/shim/BOOTX64.EFI /<path-to-ESP>/EFI/systemd/shimx64.efi
cp /usr/share/shim/mmx64.efi /<path-to-ESP>/EFI/systemd/mmx64.efi
```

### Add a boot entry for `shim`

To create a boot entry for `shim`, execute the following command:

```bash
efibootmgr --disk /dev/<your-ESP-partition> --part <part-number> --create --label "Systemd-boot via Shim" --loader '\EFI\systemd\shimx64.efi' --unicode '\EFI\systemd\systemd-bootx64.efi'
```

> To find the correct disk and partition, you can use `lsblk` to list your disk partitions.
>
> Here is mine:
>
> ```
> nvme1n1       259:7    0   1.9T  0 disk 
>   ├─nvme1n1p1 259:8    0    16M  0 part 
>   ├─nvme1n1p2 259:9    0 953.9G  0 part 
>   ├─nvme1n1p3 259:10   0     1G  0 part /efi
>   ├─nvme1n1p4 259:11   0   100G  0 part /
>   └─nvme1n1p5 259:12   0    80G  0 part /home
> ```
>
> In this case, you would execute:
>
> ```bash
> efibootmgr --disk /dev/nvme1n1 --part 3 --create --label "Systemd-boot via Shim" --loader '\EFI\systemd\shimx64.efi' --unicode '\EFI\systemd\systemd-bootx64.efi'
> ```

## 5. Use `dracut` to Automatically Sign Kernel Images

### Modify `/etc/dracut.conf`

Make sure that `uefi="yes"` is set in your `/etc/dracut.conf` file, and add the following lines:

```
uefi_secureboot_key="<path-to-your-private-key>"
uefi_secureboot_cert="<path-to-your-certificate>"
```

### Rebuild the UKI

For the Gentoo distro kernel, you can rebuild the Unified Kernel Image (UKI) by:

- `portage`

```bash
emerge --config sys-kernel/gentoo-kernel-bin
```

> If you are using `sys-kernel/gentoo-kernel`, replace it accordingly.

- `dracut`

  Execute the following command to rebuild the UKI:

  ```bash
  dracut --kernel-image=/usr/src/<path-to-your-kernel>/arch/<your-architecture>/boot/bzImage
  ```

> If you want to sign a UEFI executable file manually, you can use the following command:
>
>   ```bash
>   sbsign --key <path-to-private-key> --cert <path-to-certificate> --output /<path-to-ESP>/EFI/Linux/<your-UKI> /<path-to-ESP>/EFI/Linux/<your-UKI>
>   ```

## 6. Import Your Keys into the Machine Owner Key Manager

> Ensure that you have copied the `MOK.cer` file to the EFI System Partition, as it will be used by the MOK Manager to import your keys.

- Reboot your system.

- Enable Secure Boot in your UEFI BIOS settings.

- Choose `Systemd-boot via Shim` from the boot menu.

  > Now you should be able to access the MOK Manager, where you can import your keys.

- Select `Enroll key` and navigate to the `MOK.cer` file you copied earlier.

- Select `Reboot`.

- After rebooting, your system should now be able to boot with Secure Boot enabled using `systemd-boot`.

## 7. Verify Secure Boot and 3rd-Party Kernel Modules Status

- Verify Secure Boot status by executing:

```bash
dmesg | grep -i secure
```

> Here is mine:
>
> ```
> [    0.009330] Secure boot enabled
> ```

- Check the status of 3rd-party kernel modules:

```bash
nvidia-smi
```

If there is a successful output in the table, it indicates that the NVIDIA driver is loaded correctly and Secure Boot is functioning as expected.


**The security of Secure Boot implemented in this guide relies entirely on the strength of your UEFI BIOS passcode and MOK Manager passcode.**

- **A weak UEFI BIOS passcode may allow others to disable Secure Boot and other security settings.**

- **A weak MOK Manager passcode may allow others to enroll their keys, compromising the security of your system.**

**Enabling Secure Boot, in fact, will not substantially enhance the security of your PC; it is mainly to avoid interference with Windows features.**
# General Tips for Using GNU/Linux

> 若您已按照先前指導完成安裝作業且對自己的動手能力有一定的信心，那麽您可以閲讀本篇指導以進行更多進階設定。

**在您執行任何操作之前，切記建立 btrfs 快照備份！**

## `SDDM` 登入器

- `Numlock` 狀態

**若您的鍵盤附有數字鍵盤，可執行以下操作確保於 SDDM 登入畫面數字鍵盤鎖定是開啓的**

SDDM 登入器的配置檔案存放於 `/etc/sddm.conf.d`

執行
```sh
sudo nano /etc/sddm.conf.d/99-numlock.conf
```

寫入
```
[General]
Numlock=on
```

按下 `Control` 和 `X`，輸入 `Y` 以寫入，再按下 `Enter` 以先前指定的檔案名保存。

- 登入時使用 Wayland 會話

執行
```sh
sudo nano /etc/sddm.conf.d/10-wayland.conf
```

寫入
```
[General]
DisplayServer=wayland
GreeterEnvironment=QT_WAYLAND_SHELL_INTEGRATION=layer-shell

[Wayland]
CompositorCommand=kwin_wayland --drm --no-lockscreen --no-global-shortcuts --locale1
```

按下 `Control` 和 `X`，輸入 `Y` 以寫入，再按下 `Enter` 以先前指定的檔案名保存。


## 排程執行 `fstrim`

- `systemd` 使用者請使用 `.timer` 

`fstrim.timer` 計時器會在每周激活服務，在所有已掛載的支援 discard 操作的文件系統上執行 fstrim，此舉是通知磁碟主控使用演算法來將寫入作業平衡到整塊閃存上

執行
```sh
sudo systemctl enable --now fstrim.timer
```

- `openrc` 使用者請使用 `cronie` 等排程守候進程

使用 `cronie` 的 `/etc/crontab` 以全域排程

於 `/etc/crontab` 中寫入
```
# Run file system trim once a week
@weekly root  /sbin/fstrim --all
```

按下 `Control` 和 `X`，輸入 `Y` 以寫入，再按下 `Enter` 以先前指定的檔案名保存。

以 `root` 執行
```sh
crontab /etc/crontab
```

以套用變更

## NVIDIA Laptop GPU 電源管理

- 啓用 `systemd` 的服務

`nvidia-powerd.service` 是動態管理 NVIDIA Laptop GPU 電源狀態的服務，啓用以優化電源使用狀況

執行
```sh
sudo systemctl enable nvidia-powerd.service
```

- 啓用 `openrc` 的服務

`nvidia-powerd` 是 `openrc` 使用的服務

執行
```sh
rc-update add nvidia-powerd default
```

以將服務加入 default runlevel

## NVIDIA GPU 被 Xorg Server 持續佔用即便沒有視訊輸出

加入 X11 配置檔案

執行
```sh
sudo nano /usr/share/X11/xorg.conf.d/20-intel.conf
```

寫入以下内容：
```
Section "ServerFlags"
  Option "AutoAddGPU" "off"
EndSection
```
按下 `Control` 和 `X`，輸入 `Y` 以寫入，再按下 `Enter` 以先前指定的檔案名保存，最後重新開機

登入系統后可以透過 `nvidia-smi` 或 `nvtop` 檢查NVIDIA GPU狀態
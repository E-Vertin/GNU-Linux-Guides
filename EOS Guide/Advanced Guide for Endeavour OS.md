# Advanced Guide for Endeavour OS
>若您已按照先前指導完成EOS的安裝作業且對自己的動手能力有一定的信心，那麽您可以閲讀本篇指導以進行更多進階設定。

**在您執行任何操作之前，切記建立btrfs快照備份！**

## `SDDM`登入器的`Numlock`狀態
**若您的鍵盤附有數字鍵盤，可執行以下操作確保於SDDM登入畫面數字鍵盤鎖定是開啓的**

SDDM登入器的配置檔案存放於`/etc/sddm.conf.d`

執行
```sh
sudo nano /etc/sddm.conf.d/99-numlock.conf
```

於第一行填入
```
Numlock=on
```

按下`Control`和`X`，輸入`Y`以寫入，再按下`Enter`以先前指定的檔案名保存。

## NVIDIA GPU的MUX Switch控制
**若您的筆電支援NVIDIA Optimus，可透過`envycontrol`實用程式控制MUX Switch**

欲獲取`envycontrol`，可前往其位於*GitHub*的[項目頁面](https://github.com/bayasdev/envycontrol)歸檔源碼

解壓縮獲得其源碼資料夾，於其中開啓終端，執行
```sh
sudo python ./envycontrol.py -s hybrid
```
（可選）若您的筆電使用的NVIDIA GPU是Turing架構（RTX 20與GTX 16）或更新的，則可以使用PCI-Express Runtime D3電源管理功能

執行
```sh
sudo python ./envycontrol.py -s hybrid --rtd3
```
按照envycontrol文檔，對於使用Ampere架構（RTX 30）或更新的GPU，應執行
```sh
sudo python ./envycontrol.py -s hybrid --rtd3 3
```

之後，該實用程式將重新建立initramfs，並提示重新開機以套用變更。

## NVIDIA GPU能耗問題

### 使用`envycontrol`關閉NVIDIA GPU
回到envycontrol源碼資料夾，執行
```sh
sudo python ./envycontrol.py -s integrated
```
重新開機以套用變更，關閉NVIDIA GPU，僅使用Intel Integrated GPU。

### 變更X11配置檔案以使得X Server執行於Intel Integrated GPU
>注意：該方法僅適用於不連接外部監視器的使用場景。該過程需要關閉圖形界面（即進入多使用者模式）。

>該方法對於Ada Lovelace架構（RTX 40）的GPU效果並不明顯，因爲透過`envycontrol`設定混合輸出並使用RTD3時已以較低能耗運作。對Ampere架構（RTX 30）的GPU效果較明顯。

設定systemd開機進入多使用者模式

執行
```sh
sudo systemctl set-default multi-user.target
```
並重新開機

開機後會顯示tty1，此時成功進入多使用者模式

添加X11配置檔案
```sh
sudo nano /usr/share/X11/xorg.conf.d/20-intel.conf
```

寫入以下内容：
```
Section "ServerFlags"
  Option "AutoAddGPU" "off"
EndSection
```
按下`Control`和`X`，輸入`Y`以寫入，再按下`Enter`以先前指定的檔案名保存

設定systemd開機進入圖形化模式

執行
```sh
sudo systemctl set-default graphical.target
```
並重新開機

登入系統后可以透過`nvidia-smi`或`nvtop`檢查NVIDIA GPU狀態，可以發現Xorg的進程已不在NVIDIA GPU上執行
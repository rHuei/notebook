[參考連結:http://dic.vbird.tw/network_project/unit02.php](http://dic.vbird.tw/network_project/unit02.php)
# 實體主機母系統 (Host) 的設計
作為虛擬主機的母系統之用，應該盡量減少不必要的服務，同時降低可以連線操作的人數！讓系統的穩定性提高。 此外，由於作為虛擬機器的原生母系統，其效能得要進行一些調整與設計，這樣你的虛擬機器運作會比較順暢些。
## 1. 系統軟體自動更新與維護
- 先檢查時間，網路自動校時
```bash
date
ntpdate ntp.ksu.edu.tw
hwclock -r  # 讀取bios時間
hwclock -w  # 更新bios時間
```
- 修改 yum 來源，直接指向崑山計中
```bash
vim /etc/yum.repos.d/CentOS-Base.repo
----------------------------------------------
[base]
name=CentOS-$releasever - Base
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
baseurl=http://ftp.ksu.edu.tw/FTP/Linux/CentOS/7/os/x86_64/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

#released updates
[updates]
name=CentOS-$releasever - Updates
#mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
baseurl=http://ftp.ksu.edu.tw/FTP/Linux/CentOS/7/updates/x86_64/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
----------------------------------------------
```
- 進行一次全系統更新
```bash
yum clean all
yum update
```
- 觀察一下核心的項目 (rpm -qa | grep kernel | sort) 以及核心位置的目錄資料 (/lib/modules)
```bash
rpm -qa | grep kernel | sort
abrt-addon-kerneloops-2.1.11-52.el7.centos.x86_64
kernel-3.10.0-957.5.1.el7.x86_64
kernel-3.10.0-957.el7.x86_64
kernel-tools-3.10.0-957.5.1.el7.x86_64
kernel-tools-libs-3.10.0-957.5.1.el7.x86_64

ll /lib/modules
總計 8
drwxr-xr-x. 7 root root 4096  2月 26 09:36 3.10.0-957.5.1.el7.x86_64
drwxr-xr-x. 7 root root 4096  2月 19 20:08 3.10.0-957.el7.x86_64

dracut -v test.img kernel-3.10.0-957.el7.x86_64
```
```bash
lsmod
modinfo e1000e  # 內建網卡模組

ip links show # 查看網卡名稱
lspci | grep -i ether
ethtool -i eno1 # 查看driver名稱

dracut --add-drivers "e1000e" -v test.img kernel-3.10.0-957.el7.x86_64 # 將新模組加入
depmod -a # 重新所有模組
modprobe nfs  # 將模組載入到目前系統
```
- 在 /root/bin 底下，建立一隻名為 maintain.sh 的 shell script，且每日 1:20 執行這個指令
```bash
mkdir /root/bin
cd bin
```

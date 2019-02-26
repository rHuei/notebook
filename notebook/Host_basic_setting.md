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
```bash
yum clean all
yum update
```

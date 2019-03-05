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
```bash
vim maintain.sh
----------------------------------------------
#!/bin/bash

#1. check the date and time
/sbin/ntpdate 120.114.100.1
/sbin/hwclock -w

# 2. yum update
/bin/yum -y update
----------------------------------------------

sh -x maintain.sh
chmod a+x maintain.sh

vim /etc/crontab
----------------------------------------------
20   1  *  *  * root /root/bin/maintain.sh
----------------------------------------------

/root/bin/maintain.sh
```

## 2. 讓系統維護狀態可以讓你瞧見
因為我們的環境裡面已經有既有的 mail server，所以無須重新自己設定 mail server，因為目的不同！ 我們只需要讓這部 server 的訊息，可以傳送到外部的 public mail server 即可
- 修改 /etc/postfix/main.cf 的內容，將 relayhost 設定為 [mail.ksu.edu.tw] 的模樣， 重新啟動 postfix 之後，就支援信件轉送到 relayhost 的機器上了！
```bash
dig -x [ip]  # 複製主機名稱

vim /etc/postfix/main.cf  # 修改這幾行`
----------------------------------------------
myhostname = xxx.xxx.xxx.xxx.ksu.edu.tw
myorigin = $myhostname
relayhost = [mail.ksu.edu.tw]
----------------------------------------------

systemctl restart postfix.service
```
- 再加上 /etc/aliases 修改 root 的收件人成為 root: root,yourname@mail.ksu.edu.tw 這樣， 當 root 有信件時，就可以送一份給你在 mail.ksu.edu.tw 上面的帳號！如果沒有，那可能還是收不到...
```bash
vim /etc/aliases  # 修改最底下
----------------------------------------------
root:           root,s106001694@g.ksu.edu.tw
----------------------------------------------

newaliases

echo "hahaha" | mail -s 'test from dic' root  # 測試寄信看看
```

## 3. 系統安全強化
- 未來可能會開發一些奇怪的東西，因此建議 SELinux 設定改為 permissive 即可
```bash
vim /etc/selinux/config
----------------------------------------------
SELINUX=permissive
----------------------------------------------

restorecon -Rv /etc
getenforce
setenforce 0
getenforce
```
- 將不必要的服務關閉，對 Internet 提供服務的，只需要 port 22 即可
```bash
vim /etc/ssh/sshd_config  # 修改底下幾行
----------------------------------------------
PermitRootLogin no
UseDNS no
----------------------------------------------

systemctl restart sshd
```
- 將 firewalld 服務關閉，改成 iptables 的狀態
```bash
systemctl stop firewalld
systemctl disable firewalld
yum install iptables-services
systemctl start iptables
systemctl enable iptables

iptables-save > firewall.sh
vim firewall.sh
----------------------------------------------
#!/bin/bash

# 1. clean rule
iptables -F
iptables -X
iptables -Z

# 2. create policy
iptables -P INPUT DROP
iptables -P OUTPUT ACCEPT
iptables -P FORWARD ACCEPT

# 3. create rules
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -j ACCEPT
iptables -A INPUT -i lo -j ACCEPT
# port 22 only for DIC class room.
iptables -A INPUT -s 120.114.140.0/24 -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -s 120.114.141.0/24 -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -s 120.114.142.0/24 -p tcp -m state --state NEW -m tcp --dport 22 -j ACCEPT

# 4. NAT
iptables -t nat -F
iptables -t nat -X
iptables -t nat -Z
iptables -t nat -A PREROUTING -p tcp --dport 5050 -j REDIRECT --to-port 22

# final. save rule
iptables-save > /etc/sysconfig/iptables
----------------------------------------------

sh firewall.sh
```
- 定期觀察你的磁碟陣列狀態
```bash
cat /proc/mdstat
----------------------------------------------
Personalities : [raid1]
md126 : active raid1 sda[1] sdb[0]
      488383488 blocks super external:/md127/0 [2/2] [UU]

md127 : inactive sda[1](S) sdb[0](S)
      6192 blocks super external:imsm

unused devices: <none>
----------------------------------------------

mdadm --detail /dev/md126
mdadm -A -U resync /dev/md126 /dev/sda /dev/sdb   # 重新同步

smartctl --scan
smartctl --all /dev/sda
smartctl -t short /dev/sda
```
定期寄信檢查
```bash
vim bin/maintain.sh
----------------------------------------------
#!/bin/bash

#1. check the date and time
echo "############################################"
echo "check your server's time"
/sbin/ntpdate 120.114.100.1
/sbin/hwclock -w

# 2. yum update
echo "############################################"
echo "check update package"
/bin/yum -y update

# 3. check your hard drive's health
echo "############################################"
echo "Your raid status"
cat /proc/mdstat
echo "disk's temp"
smartctl --all /dev/sda | grep -i temp | grep -v Min
smartctl --all /dev/sdb | grep -i temp | grep -v Min
echo "check usage"
df -h | grep -v 'tmpfs'

# 4. who login my server use ssh (last 5 person)
echo "############################################"
echo "check ssh login"
last | head -n 5
----------------------------------------------
```
## 4. 系統效能調整
- 先透過 tuned-adm list 來查看目前的系統自動調整效能
```bash
tuned-adm list
----------------------------------------------
Available profiles:
- balanced                    - General non-specialized tuned profile
- desktop                     - Optimize for the desktop use-case
- latency-performance         - Optimize for deterministic performance at the cost of increased power consumption
- network-latency             - Optimize for deterministic performance at the cost of increased power consumption, focused on low latency network performance
- network-throughput          - Optimize for streaming network throughput, generally only necessary on older CPUs or 40G+ networks
- powersave                   - Optimize for low power consumption
- throughput-performance      - Broadly applicable tuning that provides excellent performance across a variety of common server workloads
- virtual-guest               - Optimize for running inside a virtual guest
- virtual-host                - Optimize for running KVM guests
Current active profile: throughput-performance
----------------------------------------------
```
- 可以使用 tuned-adm profile XXX 來設定好使用的 profile
```bash
# 最高效能
tuned-adm profile throughput-performance
# KVM效能
tuned-adm profile virtual-host
```
- 可以針對主機的網路參數進行優化
```bash
net.core.optmem_max     =  262144
net.core.rmem_default   =  262144
net.core.wmem_default   =  262144
net.core.rmem_max       = 8388608
net.core.wmem_max       = 8388608
net.ipv4.tcp_rmem       = 4096 87380 8388608
net.ipv4.tcp_wmem       = 4096 65536 8388608
net.ipv4.tcp_tw_reuse           = 1
net.ipv4.tcp_tw_recycle         = 1
net.ipv4.tcp_window_scaling     = 1
net.ipv4.tcp_sack               = 0
net.ipv4.tcp_timestamps         = 0
net.ipv4.tcp_syncookies         = 0
net.core.netdev_max_backlog     = 10000
net.ipv4.ip_forward           = 1
```
可以將上述的資料寫入 /etc/sysctl.d/somename.conf ，未來會自動生效，或者使用『 sysctl -p filename 』立刻啟動！

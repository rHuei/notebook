# 虛擬機器的服務、安裝與啟用
[參考資料:http://dic.vbird.tw/network_project/unit03.php](http://dic.vbird.tw/network_project/unit03.php)

## 1. 虛擬機器管理員，使用 libvirtd 以及 virsh 指令的管理
- 觀察 libvirtd 是否執行中
```bash
systemctl status libvirtd
```
- 觀察目前的虛擬機器以及虛擬網路環境
```bash
virsh list
virsh net-list
```
- 關閉與取消定義的機器
```bash
virsh [shutdown|destroy|undefine] domain
virsh [net-destroy|net-undefine] netdomain
```
- 你的系統目前會有很奇特的 port，原因之一，就是因為預設的網路系統會啟動一個虛擬橋接器，這個 virbr0 會自動的加入一個 192.168.122.X/24 的網段給你的虛擬機器使用，並且使用的是 NAT 的機制，因此使用這個 virbr0 橋接器連結的系統， 就可以自動的透過你的 host 上網了。你可以先去底下的檔案內去瞧一瞧網路設定值
```bash
vim /etc/libvirt/qemu/networks/default.xml
ip addr show
```
- 你可以查詢、關閉與取消網路橋接器的定義！透過的方法如上所示
```bash
virsh net-list
virsh net-destroy default
virsh net-list --all
virsh net-start default
virsh net-destroy default
# virsn net-undefine default (這個指令暫時不要進行！)
```
你的系統可能由於近期內有升級過，或者是其他函式庫有更新，可能會導致你的 libvirtd 有點奇怪，有時候會出現如下的訊息
```bash
# 1. 指令操作時，出現如下的奇怪錯誤！明明沒啥問題！
virsh net-start default
錯誤：無法開啟網路 default
錯誤：The name org.fedoraproject.FirewallD1 was not provided by any .service files

# 2. 查看 messages 時，出現如下奇怪的錯誤：
Mar 11 20:11:41 120-114-142-27 kernel: virbr0: port 1(virbr0-nic) entered disabled state
Mar 11 20:11:41 120-114-142-27 libvirtd: 2019-03-11 12:11:41.956+0000: 4955: error : virNetDevSendEthtoolIoctl:3072 : ethtool ioctl error: 沒有此一裝置
Mar 11 20:11:41 120-114-142-27 NetworkManager[4186]:   [1552306301.9576] device (virbr0-nic): released from master device virbr0
Mar 11 20:11:41 120-114-142-27 libvirtd: 2019-03-11 12:11:41.958+0000: 4955: error : virNetDevSendEthtoolIoctl:3072 : ethtool ioctl error: 沒有此一裝置
Mar 11 20:11:41 120-114-142-27 libvirtd: 2019-03-11 12:11:41.960+0000: 4955: error : virNetDevSendEthtoolIoctl:3072 : ethtool ioctl error: 沒有此一裝置
```
處理的方法其實很簡單，完整的重新開機是一個方式，另一個則是透過重新啟動 libvirtd 來處理即可
```bash
systemctl restart libvirtd
```
- 完成底下的實做
a. 將 /etc/libvirt/qemu/networks/default.xml 備份到 /root/virtual/ 目錄內
```bash
mkdir ~/virtual
cd /root/virtual
cp /etc/libvirt/qemu/networks/default.xml /root/virtual
```
b. 列出目前所有的虛擬網路橋接器
```bash
virsh net-list --all
```

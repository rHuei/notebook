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

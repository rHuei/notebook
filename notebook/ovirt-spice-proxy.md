# oVirt SPICE PROXY
如果需要再ovirt透過外部網路連線到vm spice console的話，就會需要使用spice proxy
## 安裝Squid
1. 安裝`squid`在防火牆機器
```
yum install squid
```
2. 開啟 `/etc/squid/squid.conf` 並修改
```
# 搜尋
http_access deny CONNECT !SSL_ports
# 更改為
http_access deny CONNECT !Safe_ports
```
新增底下幾行
```
# 後面IP是ovirt node內部IP
acl spice_servers dst 172.26.0.0/16
http_access allow spice_servers
```
3. 啟動/開機啟動Squid
```
systemctl start squid.service
systemctl enable squid.service
```
4. 開啟防火牆
```
# squid預設port為3128
firewall-cmd --add-port=3128/tcp
```
## 開啟oVirt spice proxy
1. 在管理機器，透過`engine-config`設定proxy
```
engine-config -s SpiceProxyDefault=someProxy

# example
engine-config -s SpiceProxyDefault=http://[public ip]:3128
```
2. 重新啟動`ovirt-engine`
```
systemctl restart ovirt-engine
```
這樣步驟就完成了

## 如何取消 Spice Proxy
1. 將設定值刪掉
```
engine-config -s SpiceProxyDefault=""
```
2. 重新啟動
```
systemctl restart ovirt-engine
```


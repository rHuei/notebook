# Docker 基本教學與指令
## 簡介
[Docker](https://www.docker.com/)

![](https://www.docker.com/sites/default/files/social/docker_facebook_share.png)

Docker是一個開放原始碼軟體專案，讓應用程式部署在軟體貨櫃下的工作可以自動化進行，藉此在Linux作業系統上，提供一個額外的軟體抽象層，以及作業系統層虛擬化的自動管理機制。

### 甚麼是Docker
Linux軟體容器（Linux Containers）簡稱LXC，一種作業系統層虛擬化（Operating system–level virtualization）技術。

起源於2013年由dotCloud公司所發展的軟體計畫，後來受到廣泛的關注與討論，dotCloud公司正式改名為Docker Inc。

直到0.9版之後，以Google發展的Go程式語言開發libcontainer函式庫，以自己的方式開始直接使用由Linux核心提供的虛擬化設施。

### Docker容器與虛擬器比較
因為容器並沒有作業系統，所以容易部署且會快速啟動。
![ContainerAndVM](/images/VMandContainer.PNG)
更多資料請參考[https://www.docker.com/resources/what-container](https://www.docker.com/resources/what-container)

Feauture    |    Containers    | Virtual Machines ( 傳統的虛擬化 )
------------|------------------|--------------------------------
啟動        |        秒開       | 最快也要分鐘
容量        |         MB        | GB
效能        |         快        | 慢
支援數量     | 非常多 Containers | 10多個就很了不起了
複製相同環境 |         快        | 超慢
 
### Docker名詞概念
***Image***

映像檔，可以把它想成是以前我們在玩 VM 的 Guest OS（ 安裝在虛擬機上的作業系統 ）。
Image是唯獨(R/O)

***Container***

容器，利用映像檔（ Image ）所創造出來的，一個 Image 可以創造出多個不同的 Container，

Container 也可以被啟動、開始、停止、刪除，並且互相分離。

Container 在啟動的時候會建立一層在最外（上）層並且是讀寫模式（ R/W ）。

***Registry***

可以把它想成類似 GitHub，裡面存放了非常多的 Image ，可在 [Docker Hub](https://hub.docker.com/) 中查看。

## 安裝 Docker 環境
Docker 已加入現有的yum倉庫了，所以直接 yum 安裝就好了!
```bash
yum install docker
```
開機自動開啟/啟動
```bash
systemctl start docker
systemctl enable docker
```

## Docker指令
### docker images
檢視本地目前有哪些映像檔
```docker
docker images
```

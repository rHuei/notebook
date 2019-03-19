# Docker 基本教學與指令
* [簡介](#簡介)
* [Docker指令](#Docker指令)
* [建立apache容器](#建立apache容器)
* [Dockerfile](#Dockerfile)
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
```bash
docker images
```

### docker search [image name]
搜尋Docker Hub上有哪些Image
```bash
docker search centos 

# 搜尋是官方的 Docker image
docker search centos -f is-official=true
```

### docker pull [image name]
下載image到本地
```bash
docker pull centos

# 下載指定版本
docker pull centos:[tags]
```
### docker ps
查看Container
```bash
docker ps

# 查看全部，包含停止狀態的
docker ps -a
```

### docker run
建立Container
```
# -i: 讓標準輸入維持在打開的狀態
# -t: 替Container配置一個虛擬的終端機
docker run -it centos 

# 給予容器名稱，預設為隨機名稱
docker run -it --name jack centos

# 在背景開啟容器
docker run -itd centos

# 可以將主機的Port綁定到Container的Port
docker run -itd -p 8080:80 centos
```
如果不要停止 container 而要退出 docker container 的terminal 需要輸入 `ctrl + p` 之後再輸入 `ctrl + q` 的按鍵，就不會把 container 關閉。

### docker exec 
進入虛擬機
```
docker exec -it jack bash
```

### docker start
啟動容器
```
docker start jack

# -ia: 啟動並進入容器
docker start -ia jack
```

### docker stop
關閉容器
```
docker stop jack
```

### docker commit
提交更新後的副本
```
docker commit jack web
```

### docker rm
刪除容器
```
docker rm <container_id>

# 強制刪除，如果容器執行中
docker rm -f <container_id>
```

### docker rmi
刪除映像檔
```
docker rmi <images_name>
```
## 建立apache容器
1. 建立新的容器
```
docker run -it -p 80:80  --name test centos 
```
2. 在容器安裝apache
```
yum install httpd
```
3. 製作網頁
```
echo "hello word" > /var/www/html/index.html
```
4. 啟動
```
/usr/sbin/apachectl -DFOREGROUND
```
5. 更新容器
先退出容器
```
docker commit test web
```
6. 啟動容器
```
docker run -itd -p 80:80 web /usr/sbin/apachectl -DFOREGROUND
```
以上步驟非常麻煩，可以建立一個類似腳本的方式，透過Dockerfile一建完成所有動作，快速建置映像檔。

## Dockerfile
使用 Dockerfile 讓使用者可以建立自定義的映像檔
### 基本架構
***FROM***

格式為 FROM <image>或FROM <image>:<tag>。
 
第一條指令必須為 FROM 指令。並且，如果在同一個Dockerfile中建立多個映像檔時，可以使用多個 FROM 指令（每個映像檔一次）。

***MAINTAINER***

格式為 MAINTAINER <name>，指定維護者訊息。
 
 ***RUN***
 
格式為 RUN <command> 或 RUN ["executable", "param1", "param2"]。

前者將在 shell 終端中運行命令，即 /bin/sh -c；後者則使用 exec 執行。指定使用其它終端可以透過第二種方式實作，例如 RUN ["/bin/bash", "-c", "echo hello"]。

每條 RUN 指令將在當前映像檔基底上執行指定命令，並產生新的映像檔。當命令較長時可以使用 \ 來換行。

***CMD***

支援三種格式
 - CMD ["executable","param1","param2"] 使用 exec 執行，推薦使用；
 - CMD command param1 param2 在 /bin/sh 中執行，使用在給需要互動的指令；
 - CMD ["param1","param2"] 提供給 ENTRYPOINT 的預設參數；
 
指定啟動容器時執行的命令，每個 Dockerfile 只能有一條 CMD 命令。如果指定了多條命令，只有最後一條會被執行。

如果使用者啟動容器時候指定了運行的命令，則會覆蓋掉 CMD 指定的命令。

***EXPOSE***

格式為 EXPOSE <port> [<port>...]。
 
設定 Docker 伺服器容器對外的埠號，供外界使用。在啟動容器時需要透過 -P，Docker 會自動分配一個埠號轉發到指定的埠號。

### 安裝Apache範例
```
mkdir myweb
cd myweb
vim Dockerfile
------------------------------------------------------------
FROM centos:latest
MAINTAINER CENTOS WEB APP

RUN yum -y update && \
    yum -y install httpd && \
    yum clean all && \
    echo "hello new world" > /var/www/html/index.html

EXPOSE 80

CMD ["/usr/sbin/apachectl","-DFOREGROUND"]
------------------------------------------------------------

docker build -t web:v1 .

docker images

docker run -itd -p 8888:80 web:v1
```



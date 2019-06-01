# 2019 6月份 SA@Tainan 6/1(六) Working with PowerShell
# WSUS
### How to install and configure WSUS on Windows server 2016
[參考資料:https://0857.000webhostapp.com/windows/how-to-install-and-configure-wsus-on-windows-server-2016-part-1/](https://0857.000webhostapp.com/windows/how-to-install-and-configure-wsus-on-windows-server-2016-part-1/)

### Install
```
The point:
  IIS status
  share Folder
  WID V.S. DB
```
---
### Setting
```
WSUS = APIs(.NET) + IIS + DB
```
### Client
```
 1. GPO
 2. Event Log: Detect,Install,Results
 3. Error Messages
```

> 不要在 WSUS Server 進行同步的時候作管理或維護的動作，以免導致WSUS DB索引錯亂!!


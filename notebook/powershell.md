# 2019 6月份 SA@Tainan 6/1(六) Working with PowerShell
# WSUS
Windows Server更新服務（英語：Windows Server Update Services，縮寫WSUS），曾稱為軟體更新服務（Software Update Services，縮寫SUS），是微軟公司開發的一個電腦程式，它允許管理員管理已為微軟產品發布的更新和熱修復修補程式分發到企業環境中的電腦。WSUS從微軟更新網站下載這些更新，然後分發它們到網路上的電腦。WSUS執行在Windows Server上，免費授權給微軟客戶。

# How to install and configure WSUS on Windows server 2016
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

### Server
- Updates
  * Check Related Information
  * Approve or Decline
- Client
  * State: Doenload,Reboot,Install...etc
  
### Maintain
- Microsoft is "SO SWEET",But...
  * Decline Expired Updates
  * Run Cleanup Wizard
  * Re-Index DB


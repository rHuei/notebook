# 2019 6月份 SA@Tainan 6/1(六) Working with PowerShell
# WSUS
Windows Server更新服務（英語：Windows Server Update Services，縮寫WSUS），曾稱為軟體更新服務（Software Update Services，縮寫SUS），是微軟公司開發的一個電腦程式，它允許管理員管理已為微軟產品發布的更新和熱修復修補程式分發到企業環境中的電腦。WSUS從微軟更新網站下載這些更新，然後分發它們到網路上的電腦。WSUS執行在Windows Server上，免費授權給微軟客戶。

# How to install and configure WSUS on Windows server 2016
[參考資料:https://0857.000webhostapp.com/windows/how-to-install-and-configure-wsus-on-windows-server-2016-part-1/](https://0857.000webhostapp.com/windows/how-to-install-and-configure-wsus-on-windows-server-2016-part-1/)
[指令法寶:https://goalkicker.com/](https://goalkicker.com/)
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

# Working with Powershell
### Agenda
- What's the "PowerShell"?
  * How many people aren't using it?
- Let's Dance with "WSUS"!
### Version
- How to check "PowerShell Version"?
  * For OS:Winver
  * For PS:$PSVersionTable
  * For EV:ENV:(Use 'Get-Item')
  ```
  Get-Item Env:
  ```
- Current Version is "PowerShell 6.0~"
### Help
- powershell.exe /?
- Get-Command
- Get-Alias
- Get-Help
### Type
- PowerShell = Array + String
  * Special Cmdlet: |
    % Usage:AD Cmdlet or EMS on Exchange ...etc.
  * Default format:String
### Live
- Running on "Memory"!
  * Watching on "Task Manager"
  * It's a "Process"
  * Ya...."Unlimited"!!
- Default by "32 Thread" as a Process
  * "Session"?! > "RunSpace"
### Work
- How to "Run"?
  * Excuted by "Line"
  * "Function".....?
- What to do about "nin-Ps file"?
  * 'Bat'?
  * 'exe'?
  ```
  # 等待程式跑完
  Start-Sleep-Seconds 10
  $batfile = [diagnostics.process]::Start("D:\Demo\My_Script.bat")
  $batfile.WaitForExit()
  ```
### Null
- What's "Null"?
  * $a = ""
  * $a = ''
  * $a = $Null
  * $a ≠ "" ≠ '' ≠ $Null
- Up to your "Powershell Version".
### Array
- Default by "One-dimensional array"
### Filter
- How to cut the "Messages"?
  * $a "- Replace ("")"
  * $a "- Split ("")"
  * $a -match $Regex
- Use SPecial Characters like "`n `r `t"
### History
- How to Check "Previous Command"?
  * "Get-History"
  * Alias "H"
  * Added "| ? { $_.CommandLine -eq 'H' }"
  * r -ld "Number"
- Use "F7" > "Read-Host"
### Record
- Fefault "Saved" & "Stored"
  * Use "Start-Transcript"
  * Stored Path "$PROFILE"
    % All Users's Profile Path "$PROFILE.AllUsersAllHosts"
  * To end "Stop-Transcript"
- On Remote Hosts "Use PS-Session"
###  BITS
- What's the "BITS"?
  * "Background Intelligent Transfer Service"
  * Start in "Windows 2000"
### PSObject
- [PScustomObject] in PS v3.0
- No Type "PSObject"!
- No "Hashtable"!
- Usually use with "Foreach".
### Reference Document
- Basic Explanation
- Microsoft Website
- Active Directory with PowerShell
- [Learn Windows PowerShell](https://www.books.com.tw/products/0010809471)

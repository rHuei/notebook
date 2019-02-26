# 實體主機母系統 (Host) 的設計
## 系統軟體自動更新與維護
### 先檢查時間，網路自動校時
```bash
date
ntpdate ntp.ksu.edu.tw
hwclock -r  # 讀取bios時間
hwclock -w  # 更新bios時間
```

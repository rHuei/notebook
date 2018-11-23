# 跨網站指令碼
網站指令碼（英語：Cross-site scripting，通常簡稱為：XSS）是一種網站應用程式的安全漏洞攻擊，是程式碼注入的一種。它允許惡意使用者將程式碼注入到網頁上，其他使用者在觀看網頁時就會受到影響。這類攻擊通常包含了HTML以及使用者端指令碼語言

XSS攻擊通常指的是通過利用網頁開發時留下的漏洞，通過巧妙的方法注入惡意指令程式碼到網頁，使用戶載入並執行攻擊者惡意製造的網頁程式。這些惡意網頁程式通常是JavaScript，但實際上也可以包括Java，VBScript，ActiveX，Flash或者甚至是普通的HTML。攻擊成功後，攻擊者可能得到更高的權限（如執行一些操作）、私密網頁內容、對談和cookie等各種內容。

# 檢測方式
通常有一些方式可以測試網站是否有正確處理特殊字元：
```html
><script>alert(document.cookie)</script>
='><script>alert(document.cookie)</script>
"><script>alert(document.cookie)</script>
<script>alert(document.cookie)</script>
<script>alert (vulnerable)</script>
%3Cscript%3Ealert('XSS')%3C/script%3E
<script>alert('XSS')</script>
<img src="javascript:alert('XSS')">
<img src="http://xxx.com/yyy.png" onerror="alert('XSS')">
<div style="height:expression(alert('XSS'),1)"></div>（這個僅於IE7(含)之前有效）
```

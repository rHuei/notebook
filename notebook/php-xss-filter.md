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

#### 漏洞網頁示範
- 假設有一個具有漏洞的簡單網頁是長這樣的畫面:

![xss-web](/images/xss-web.PNG)

- 正常輸入後會回應名子:

![xss-web-name](/images/xss-web-name.PNG)

- 如果在網址後面帶入特殊自元後: 
```http://example.com/php-xss-filter.php?username=阿明<script>alert('XSS')</script>```

![xss-web-attack](/images/xss-web-attack.PNG)
- 駭客就能透過這個漏洞網頁，傳送資訊給別人了。如果是一張誘人的圖片是否又更讓人想點進去看呢?
![xss-web-attack-img](/images/xss-web-attack-img.PNG)

# 漏洞的防禦和利用
#### 過濾特殊字元
避免XSS的方法之一主要是將使用者所提供的內容進行過濾，許多語言都有提供對HTML的過濾：
  - PHP的```htmlentities()```或是```htmlspecialchars()```。
  - Python的```cgi.escape()```。
  - ASP的```Server.HTMLEncode()```。
  - ASP.NET的```Server.HtmlEncode()```或功能更強的Microsoft Anti-Cross Site Scripting Library
  - Java的xssprotect (Open Source Library)。
  - Node.js的node-validator。

如果要PHP網頁對所有進行過濾，可以在的開頭加入漏洞過濾的函數，將特殊字元編碼，如下:
```php
<?php
//php防注入和XSS攻擊通用過濾
$_GET       && SafeFilter($_GET);
$_POST      && SafeFilter($_POST);
$_COOKIE    && SafeFilter($_COOKIE);

function SafeFilter (&$arr)
{
      if (is_array($arr))
     {
          foreach ($arr as $key => $value)
          {
               if (!is_array($value))
               {
                    if (!get_magic_quotes_gpc())    //不對magic_quotes_gpc轉義過的字符使用addslashes(),避免雙重轉義。
                    {
                         $value    = addslashes($value);    //給單引號（'）、雙引號（"）、反斜線（\）與 NUL（NULL 字符）加上反斜線轉義
                    }
                    $arr[$key]    = htmlspecialchars($value,ENT_QUOTES);   //&,",',> ,< 轉為html實體 &amp;,&quot;',&gt;,&lt;
               }
               else
               {
                    SafeFilter($arr[$key]);
               }
          }
     }
}
?>
```
最後在測試經過濾後的結果:

![xss-web-filter](/images/xss-web-filter1.PNG)

如此被植入的惡意程式碼，便不能正常執行了。

> 參考資料: [跨網站指令碼- 維基百科，自由的百科全書 - Wikipedia](https://zh.wikipedia.org/wiki/跨網站指令碼)


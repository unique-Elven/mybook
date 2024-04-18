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

访问首页，右键检查源码
```c
A7F9B77C16A3AA80DAA4E378659226F628326A95
D82D10564866FD9B201941BCC6C94022196F8EE8 
[md5在线解密破解,md5解密加密](https://www.cmd5.com/)
fuck you
| VIRGIN
```
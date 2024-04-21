
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
```

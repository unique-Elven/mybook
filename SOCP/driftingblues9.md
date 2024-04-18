
|   |   |   |
|---|---|---|
|driftingblues9|easy|aPphp GETSHELL、searchsploit使用、凭据收集、gdb使用、<br><br>缓冲区溢出漏洞（难）、pattern_create.rb、pattern_offset.rb 使用|

# 主机发现

```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo netdiscover -i eth0 -r 192.168.44.139/24
```
# 服务探测

```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo nmap -sV -A -T 4 -p- 192.168.44.139  

80/tcp    open  http    Apache httpd 2.4.10 ((Debian))

111/tcp   open  rpcbind 2-4 (RPC #100000)
```
# 目录扫描
```c
http://192.168.44.139/README.txt
```


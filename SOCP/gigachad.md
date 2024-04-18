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
```

开放了21，22，80



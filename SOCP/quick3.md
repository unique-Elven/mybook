kali和quick3都设置桥接模式
# 主机发现
```C
netdiscover -i eth0 -r 192.168.44.193/24
```
# 服务探测
```C
nmap -sV -A -T4 -p- 192.168.44.133
```
得到只开放了80和22端口

访问80web
dirbsa
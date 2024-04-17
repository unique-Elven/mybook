# 主机发现
```C
sudo netdiscover -i eth0 -r 192.168.44.136/24
```
# 服务扫描
```C
sudo nmap -sV -A -T 4 -p- 192.168.44.136 
```

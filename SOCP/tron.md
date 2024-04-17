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
gobuster dir  -u http://192.168.44.135/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```

```cs
http://192.168.44.135/index.html
 kzhh:SbWP9q94ZtE9qD  
 
http://192.168.44.135/MCP/tron.txt
MASTER CONTROL PROGRAM
----------------------
Ram:
Do you believe in the Users?
Crom:
Sure I do! If I didn't have a User, than who wrote me? 
KysrKysrKysrK1s+Kz4rKys+KysrKysrKz4rKysrKysrKysrPDw8PC1dPj4+PisrKysrKysrKysrKy4tLS0tLi0tLS0tLS0tLS0tLisrKysrKysrKysrKysrKysrKysrKysrKy4tLS0tLS0tLS0tLS0tLS0tLS0tLS4rKysrKysrKysrKysrLg==


http://192.168.44.135/MCP/terminalserver/30513.txt
substitute
--------------------------
plaintext
abcdefghijklmnopqrstuvwxyz

ciphertext
zyxwvutsrqponmlkjihgfedcba
--------------------------


```

对kysr.............一长串的进行base64解码。再进行ba
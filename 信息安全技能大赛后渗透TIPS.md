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
samdump2 system.hive sam.hive > hash.txt
```
```bash
john -format=NT hash.txt  #破解,使用john自带的密码字典
john --show -format=NT hash.txt    #查看破解结果
```




  
  
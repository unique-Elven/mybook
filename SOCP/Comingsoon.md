刚开始没有IP
单用户修改密码也需要输入用户密码，不能改
/boot/grub/grub.cfg配置文件下
```c

```
看了wp，才知道root用户密码ToyStory3，进去后也没有dhclient命令获取ip
只能换一种方式，编辑vim /etc/network/interface
[Linux(Debian)网卡设置\_debian auto eth0-CSDN博客](https://blog.csdn.net/willhuo/article/details/79610260)

|   |   |   |
|---|---|---|
|comingsoon|easy|前端分析、文件上传绕过、提示信息利用、shadow 爆破、hydra 使用、|

# 主机发现
```C
sudo netdiscover -i eth0 -r 192.168.44.133/24
┌──(kali㉿kali)-[~]
└─$ fping -aqg 192.168.44.0/24
```
# 服务探测
```C
sudo nmap -sV -A -T 4 -p- 192.168.44.137
```
只有22和80
# 目录爆破

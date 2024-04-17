# 主机发现
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
得到密码。猜测账户就是player
```cs
┌──(kali㉿kali)-[~/桌面/OSCP]
└─$ python translater.py  kzhh:SbWP9q94ZtE9qD  
kzhh:SbWP9q94ZtE9qD
pass:hydk9j94agv9jw
```

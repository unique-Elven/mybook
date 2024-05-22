IDA小技巧
```C
小技巧
0.使用View▶Open Subviews命令恢复你无意中关闭的数据显示窗口。
1.使用Windows▶Reset Desktop命令可迅速将桌面恢复到原始布局。   
2.利用Windows▶Save Desktop命令保存你认为特别有用的当前桌面布局。
3.使用Windows▶Load Desktop命令迅速打开你之前保存的一个桌面布局。   
4.Disassembly窗口（无论是图形视图或列表视图）是唯一一个你可以修改其显示字体的窗口
5.使用Options▶Font命令可以设置字体。
```
# Srand,rand,time()伪随机数
例题：Simple Encryptor[Hack The Box :: Login](https://app.hackthebox.com/challenges/Simple%2520Encryptor)
题解：
[Simple Encryptor reverse engineering challenge Writeup - Blog by Hiroki Fujino](https://hiroki6.dev/posts/simple-encryptor)

[HackTheBox — Simple Encryptor Write Up | by southbre | Medium](https://medium.com/@southbre/hackthebox-simple-encryptor-308949f7023c)

```css
`__ROL4__`是uint32循环左移，`__ROL2__`是uint16循环左移，`__ROL1__`是uint8循环左移

`__ROR4__`是uint32循环右移，`__ROR2__`是uint16循环右移，`__ROR1__`是uint8循环右移

这些名称都是ida的定义，可以在`ida根目录\plugins\defs.h`中详细查看：
```
```css
# [汇编指令MOVSX与MOVZX](https://www.cnblogs.com/Reyzal/p/5142302.html)

MOVSX 操作数A ，操作数B

MOVZX 操作数A ，操作数B

相同点：操作数B 空间必须小于 操作数A

1、格式与MOV基本相同

2、能完成小存储单元向大存储单元的数据传送 比如 movsx eax,bx  movzx ebx,ax     movsx eax,bx

MOVSX，MOVZX 与MOV指令区别：

  1、MOVSX，MOVZX的操作数B所占空间必须小于操作数A. 

  2、MOV指令是原值传送，不会改动。而MOVSX与MOVZX有可能会改动

MOVSX与MOVZX的区别：

1、MOVSX将用操作数B的符号位扩展填充操作数A的余下空间，如果是负数则符号位为1，如果是正数则和MOVZX功能相同

2、MOVZX将用0来扩展填充操作数A的余下空间。
```

srand()每次种子一样，rand每次生成的随机数也一样
验证代码：
```c
#include <stdio.h>
#include <stdlib.h>

int main(void) {
    FILE *f;
    size_t flagSize;
    unsigned int seed;
    f = fopen("flag.enc", "rb");
    // seek until the end of the file to get the size
    fseek(f, 0, SEEK_END);
    flagSize = ftell(f);
    // seek to the beginning
    fseek(f, 0, SEEK_SET);

    fread(&seed, 1, 4, f);
    fclose(f);

    printf("seed: %d\n", seed);
    srand(seed);

    for(int i = 0; i < (long)flagSize; i++) {
        printf("%d\n", rand());
    }

    return 0;
}
```

exploit
```c
#include<stdio.h>
#include<string.h>
#include<stdlib.h>
int main(void) {
    typedef unsigned char byte;
    FILE* f;
    size_t flagSize;
    byte* flag;
    unsigned int seed;
    long i;
    int rnd1, rnd2;

    f = fopen("flag.enc", "rb");
    // seek until the end of the file to get the size
    fseek(f, 0, SEEK_END);
    flagSize = ftell(f);
    // seek to the beginning
    fseek(f, 0, SEEK_SET);
    // allocate memory of the flag
    flag = (byte*)malloc(flagSize);
    fread(flag, 1, flagSize, f);
    fclose(f);

    // take seed from the first 4 bytes
    int flagOffset = 4;
    memcpy(&seed, flag, flagOffset);
    srand(seed);
    printf("%d", seed);
    for (i = flagOffset; i < (long)flagSize; i++) {
        rnd1 = rand();
        rnd2 = rand();
        rnd2 = rnd2 & 7;
        flag[i] =
            flag[i] >> rnd2 |
            flag[i] << (8 - rnd2);
        flag[i] = rnd1 ^ flag[i];
        printf("%c", flag[i]);
    }
}
```

# 字符猜测
例题：Behind the Scenes
题解：[HackTheBox Behind The Scenes 逆向题目分析\_bbe -e 's/\\x0f\\x0b/\\x90\\x90/g' behindthescenes-CSDN博客](https://blog.csdn.net/qq_45894840/article/details/126484378)
>NOP的指令为：0x90  
UD2的指令为：0x0f 0x0b  
我们可以使用bbe工具来替换指令

```
bbe -e 's/\x0f\x0b/\x90\x90/g' behindthescenes > new
```
例题：ctfshow：flag白给
# 静态分析不出来的可结合动态调试
例题：ctfshow：re3
[CTFShow re3-CSDN博客](https://blog.csdn.net/Mintind/article/details/128138222)
[[ctf.show.reverse] re3\_ctfshow re3-CSDN博客](https://blog.csdn.net/weixin_52640415/article/details/124152965)
可以测出i=5时v16为`e560`  
而想让v16=0xffff，只需要ffff-e560就能得到`v17[6] = 1a9f`  
所以flag就是1a9f！

通过分析：`v17[6] = 1a9f`由v5决定，但v5由sscnf的dest以十六进制拷贝到v5，而dest地址和输入的s地址连续，可以覆盖，如下图
![[Pasted image 20240327102009.png]]

![[Pasted image 20240327102427.png]]

例题ctshow：逆向5
题解：
[ctf.show](https://ctf.show/writeups/1492743)
[REVERSE-PRACTICE-CTFSHOW-1\_ctfshow reverse exe dll-CSDN博客](https://blog.csdn.net/weixin_45582916/article/details/118497453)
![[Pasted image 20240327142348.png]]
第二种解法：
```C
#不需要管1.dll到底干了啥，直接load，然后调H函数直接用就行：
#include <stdio.h>
#include <windows.h>
typedef char(*H)(char);
int main() {
	char Str[] = "dba54edb0?d6>7??3ef0f1caf2ad3102";
	HMODULE hModule;
	hModule = LoadLibraryA("1.dll");
	H h = (H)GetProcAddress(hModule, "H");
	for (int i = 0; i < strlen(Str);i++)
	{
		printf("%c", h(Str[i]));
	}
	return 1;
}
```

例题：ctfshow：红包题 武穆遗书
先脱壳UPX，获取flag的逻辑就是，v6与v4相同，但是程序一运行就会退出，因此需要动态调式修改程序执行逻辑，让程序逻辑流走到对v4的处理上，我是跟踪到ExitProcess把它直接retn掉，然后flag直接就出来了：flag{fmf_is_great!}
![[Pasted image 20240327145535.png]]

# 分析自定义算法的逆向

例题：ctfshow：逆向4
题解：[REVERSE-PRACTICE-CTFSHOW-1\_ctfshow reverse exe dll-CSDN博客](https://blog.csdn.net/weixin_45582916/article/details/118497453)
```python
table=")(*&^%489$!057@#><:2163qwe"

ans="/..v4p$$!>Y59-"

ans_s=""

for c in ans:

    ans_s+=chr(ord(c)^7)

print(ans_s)

num=0

for c in ans_s:

    num *= 26

    index=table.find(c)

    num+=index

print(num)

#())q3w##&9^2>*

#2484524302484524302
```

# java程序逆向+DES
例题：ctfshow：红包六
题解：[[ctf.show.reverse] 红包六-CSDN博客](https://blog.csdn.net/weixin_52640415/article/details/124191991)


# 斐波那契数列+多项式求解
例题：ctfshow：数学不及格
解答：[[ctf.show.reverse] 数学不及格\_ctfshow 数学不及格-CSDN博客](https://blog.csdn.net/weixin_52640415/article/details/124177361)
exploit
```C
def fib(n):
    a=1
    b=1
    i=2
    while i<n:
       a,b = b,a+b 
       i+=1
    return b 
 
s = 0x233F0E151C + 0x1B45F81A32 + 0x244C071725 + 0x13A31412F8C
for v4 in range(58,201):
    v9 = fib(v4)
    if v4+3*v9 ==  s:
        print(v4)
        print(bytes.fromhex(hex(v9-0x233F0E151C)[2:])+ bytes.fromhex(hex(v9-0x1B45F81A32)[2:]) +bytes.fromhex(hex(v9-0x244C071725)[2:]) + bytes.fromhex(hex(v4+0x6543)[2:]))
        break
 
#flag{newbee_here}
```

strtol()函数的作用是将字符串类型转换为相应的long类型。[C++中strtol函数的使用方法-CSDN博客](https://blog.csdn.net/hou09tian/article/details/110630832)

python的pyc文件base64
例题： ctfshow签退
题解：[CTFSHOW签退\_ctfshow 签退-CSDN博客](https://blog.csdn.net/XFox_/article/details/121047030)

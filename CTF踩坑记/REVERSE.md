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
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

# 
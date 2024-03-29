[Shellcode奇技淫巧汇总](https://www.xuenixiang.com/forum.php?mod=viewthread&tid=1969)
整理了一些 shellcode 以及一些有漏洞的程序 , 需要学习的小伙伴如果有兴趣可以参考一下  
**1. 清空通用寄存器 eax**  

```
;=========================================
; 方案一
;=========================================
mov eax,0 ; 直接移动
;=========================================
; 方案二
;=========================================
xor eax,eax ; 异或
;=========================================
; 方案三
;=========================================
shr eax,8 ; 右移 8 位
;=========================================
; 方案四
;=========================================
and eax, 0
;=========================================
; 方案五
;=========================================
; 还有一种思路是 : (但是比较占空间)
mov eax, 01010101B
mov ebx, 10101010B
and eax, ebx
```

**2. 获取内存地址 (例如自己写的 "/bin/sh" 的地址)**  

```
;=========================================
; 方案一
;=========================================
global _start:
  _start:
    jmp target
  joker : 
    pop eax
    ; Your code
    ; ...
    ; ...
  target:
    call joker
    db "/bin/sh", 0
;=========================================
; 方案二
;=========================================
xor eax, eax
push eax ; 字符串以 0 结尾
; "/bin/sh" -> 0x2f 0x62 0x69 0x6e 0x2f 0x2f 0x73 0x68
push 68732f2fH
push 6e69622fH
mov ebx, esp
```

**3.消除 0 字节**  

```
利用低位寄存器例如 al , ah , 进行操作
例如 : 
  mov eax, 0AH
可以写成 : 
  ; 首先清空 eax
  ; 可以用到第一节中的清空寄存器的指令中的任意一个
  shr eax, 8
  mov al, 0AH
```

**4. 获取栈顶指针地址**  

```
mov ebx, esp
```

**5. 如何在字符串后添加 \0**  

```
首先利用 push [立即数(非0)]
这样得到的机器码只有两位 , 但是真正执行的时候被 push 到栈上的数据是 4 字节
然后再 push 字符串
```

**6. 将 edx 设置为 0**  

```
cdq 
; 该指令先把edx的每一位置成eax的最高位
; 若eax>=0x80000000, 则edx=0xFFFFFFFF
; 若eax<0x80000000，则edx=0x00000000
; 再把edx扩展为eax的高位。
```

**7. 消除在 push 较小的数的时候 push 指令中的 0 字节**  

```
例如 : 
push 0BH
字节码为 : 68 0B 00 00 00
我们可以使用下面的指令来消除 0 字节 : 
xor eax, eax
mov al, 0BH
push eax
这样得到的字节码为 : 
31 c0
b0 0b
50   
6a 0b
```

**8. 同时清空 RAX, RDX 和 RSI (64位)**  

```
xor esi,esi
mul esi
```
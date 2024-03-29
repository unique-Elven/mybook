# 1. PE文件结构
![[897781438.gif]]

# 02 PE文件的两种状态

![[QQ截图20240305105327.png]]

各个结构体所占字节大小如下图： 
32位的扩展PE头占224（E0）字节， 扩展PE头的大小是由标准PE头的SizeOfOptionalHeader决定的，32位程序这里存的是E0，64位程序这里存的是F0   
![[QQ截图20240305110246.png]]

在第一个结构体中的最后一个成员，指向的是PE文件头标志的位置，所以从IMAGE_DOS_HEADER最后一个成员结束位置到PE文件头标志的中间都是DOS块，这块长度是可变的，取决于e_fanew的值

从最后一个IMAGE_SECTION_HEADE结构体结束位置后面就是编译器放入的其他的数据了，那这后面的数据怎么确定填充的大小呢？怎么做分割呢？在填充范围内我们才能随意写数据

>[!IPS] 在扩展PE头中：
>成员SizeOfHeaders保存了整个PE头（DOS+PE+所有节表）的按照文件对齐以后的大小
>成员FileAlignment的值是节区在磁盘文件中的最小单位（文件对齐）`FileAlignment的值从扩展PE头开始查36字节后就是`
>`SizeOfHeaders的值从FileAlignment的值再往后查20字节后就是`

从`SizeOfHeaders的值`对齐后紧接着就是第一个节数据的开始位置，然后继续按照文件对齐以后的大小分配存储空间，实际没用上的填00，其他节依次类推

![[QQ截图20240305145357.png]]
成员 `SectionAlignment` 是内存对齐的大下小值，不一定能够和`FileAlignment`一样；`SectionAlignment`位置就在`FileAlignment`前面的4字节
# 3. DOS头属性说明

![[QQ截图20240305151352.png]]

# 04 标准PE头属性说明
![[QQ截图20240305151515.png]]



![[QQ截图20240305151904.png]]


以下是定义在winnt.h文件中的Characteristics的值，需要将其16位按bit位展开来对比，下面有为1时候的含义。

![[QQ截图20240305155306.png]]

# 05 扩展PE头属性说明
![[QQ截图20240305155813.png]]


![[QQ截图20240305211907.png]]
![[QQ截图20240305211759.png]]


![[QQ截图20240305211243.png]]

# 07 RVA与FOA(FA,RAW)的转换
[PE格式第三讲扩展,VA,RVA,FA(RAW),模块地址的概念 - iBinary - 博客园](https://www.cnblogs.com/iBinary/p/7653693.html)
RVA = VA - ImageBase
<1 > 判断RVA是否在头部，在的话直接返回
	FOA = RVA

<2 > 判断当前位置在哪一个节
>   节.VirtualAddress >=RVA >= 节.VirtualAddress
>   差值 = RVA - 节.VirtualAddress
>   FOA = 节.PointerToRawData + 差值

(节.VirtualAddress = = 内存中节区起始位置（RVA）)
(节.PointerToRawData = = 磁盘文件中节区起始位置)
ps : 新编译器编译的程序的文件对齐和内存对齐是一样的，所以FOA=RVA

# 08 空白区添加代码
E8 后面的硬编码  =  call跳转的地址 - 当前指令行的地址 - 5
E9 后面的硬编码  =  jmp跳转的地址 - 当前指令行的地址 - 5

<0 > 构造要写入的代码：
6A 00 6A 00 6A 00 6A 00 E8 `00 00 00 00` E9 `00 00 00 00`

`00 00 00 00`待定

<1 > 在PE的空白区构造一段代码
现在函数地址需要计算:

在 Alt+E 的位置找到USER32.dll后点进去，按Ctrl+N 找到MessageBoxA函数地址
（这种方法只能在本地机器上能执行，要在所有机器都能执行才是标准的shellcode）
MessageBoxA函数地址 - (ImagesBase+当前文件地址) - 5 = E8后面要填写的地址
(AddressOfEntryPoint+ ImageBase) -  (ImagesBase+当前文件地址) -5 = E9后面要填写的地址（就是原来的入口地址）

<2 > 修改入口地址为新增的代码位置
<3 > 新增代码执行后，跳回入口地址

# 09 扩大节
1. 为什么要扩大节？
	我们可以在任意的空白区添加自己的代码，如果添加的代码比较多，空白区域不够怎么办？所以扩大节就是为了解决这个问题。

2. 扩大哪一个节呢？
	最后一个节

3. 节表数据结构说明
	![[QQ截图20240306172157.png]]

4. 扩大节的步骤
	<1 > 分配新的一块空间，大小为S
		具体操作例如可以在UE/Winhex/010里最后位置插入
	<2 > 将最后一个节表的SizeOfRawData和VirtualSize改成N
		N = (SizeOfRawData或者VirtualSize内存对齐后的值) + S
		谁大照着谁的改
	<3 > 修改SizeOfImage大小 
	最后如果节没有可执行的属性，还要修改`Characteristics`

# 10 新增节
[PE知识复习之PE新增节 - iBinary - 博客园](https://www.cnblogs.com/iBinary/p/9737719.html)
[C语言实现PE的拉伸压缩和扩大、合并、增加节区\_pe将区段合并-CSDN博客](https://blog.csdn.net/qq_35289660/article/details/106060024)
[2.11 PE结构：添加新的节区-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2327021)
标准PE头中成员`NumberOfSections`描述的是节表的数量

<1>判断表头文件对其内是否有足够的空间，可以添加一个节表
<2>在节表中新增一个成员
<3>修改PE头中节的数量
<4>修改sizeOflmage的大小
<5>再原有数据的最后，新增一个节的数据(内存对齐的整数倍)
<6>修正新增节表的属性.
>![[QQ截图20240306172157.png]]
>假设新增节的VirtualSize=0x1000，按最大的内存对齐，则新增节的SizeOfRawData=0x1000
>新增节.VirtualAddress = 上节.VirtualAddress + 上节.计算对齐后的VirtualSize
>新增节.PointerToRawData = 上节.PointerToRawData + 上节.SizeOfRawData
>sizeOflmage = 原始sizeOflmage对齐后 + 新增节的SizeOfRawData


# 12 导出表
（文件对齐和内存对齐最小单位不一致时候记得先把RVA转化为FOA才能在文件中找到正确的地址）
exe和dll同属于PE文件，都可以有导入表导出表，`但是exe一般不提供函数给别人用，所以通常只用的到导入表(不是绝对的，看用途)`，dll是两个都有

1. 如何定位导出表
前面学习扩展PE头，里面最后一个成员DataDirectory是由16个IMAGE_DATA_DIRECTORY结构体（8字节）组成的数组；结构体形式如下；
数组内第一个元素是IMAGE_DIRECTORY_ENTRY_EXPORT；
保存的是导出表的相对虚拟地址RVA和大小
![[QQ截图20240307134538.png]]

文件中从节表位置往上推8行就找到了

2. 导出表的结构
![[QQ截图20240307133341.png]]
![[Pasted image 20240307132318.png]]
导出函数个数是怎么计算的？
是直接用最后一个序号减去第一个序号，得到导出函数个数。
但是如果序号不连续呢？
那么所有导出函数的个数NumberOfFunctions就会比实际的多。

函数地址表的函数地址顺序按照序号存储（这是根据文件特征码过去汇编跟踪得到的函数功能，推出的结论）
函数名称表的名称顺序按照字母首A-Z存储
序号表的成员有几个看函数名称表有几个，成员个数一样（其实序号表里的成员存储的就是需要用到的函数地址表的索引）

例如API：GetProcAddress();
如下图，以函数名开始查找的时候
![[QQ截图20240307131123.png]]

如下图，当以序号15开始查找的时候，此时导出函数的起始序号就开始起作用了，导致函数地址表索引刷新，对应着序号表里存储的索引应该也会刷新
![[Pasted image 20240307132318.png]]
![[Pasted image 20240307132648.png]]

# 13 导入表_确定依赖模块


1. 如何定位导入表
前面学习扩展PE头，里面最后一个成员DataDirectory是由16个IMAGE_DATA_DIRECTORY结构体（8字节）组成的数组；结构体形式如下；
数组内第二个元素是IMAGE_DIRECTORY_ENTRY_IMPORT；
保存的是导出表的相对虚拟地址RVA和大小

2. 导入表结构
如下图所示，得知，一个导入表占20字节。有多少个导入模块就有多少个导入表结构，往后数，直到20个`00`的位置，代表导入表结束，这是导入表结束的标志。

![[QQ截图20240307153641.png]]

# 14 导入表_确定依赖函数


1. 导入表的结构

![[QQ截图20240307155605.png]]

INT表（Import Name Table）导入函数名表
IAT：Import Address Table 导入（函数）地址表

![[QQ截图20240307155438.png]]


![[QQ截图20240307161133.png]]IMAGE_IMPORT_BY_NAME
	Hint是导出函数地址表的索引
	Name\[1] 设计非常巧妙，就1个字节，往后开始找，遇到00结束，应为函数名的长度不确定。

![[QQ截图20240307161833.png]]

# 15 导入表_确定函数地址

程序只要用到其他dll里面的函数，那么查看汇编便是一定是间接call
间接call指向的是上面提到的导入函数地址表（IAT）

![[QQ截图20240307162440.png]]

![[QQ截图20240307162305.png]]
如上图分析，可以知道，PE文件加载后可以IAT里面直接存的就是函数地址了，但是思考为什么还需要上面那条INT呢？这就是两条腿走路的思路。如果没有了上面那条线，那么IAT被修改了又记不住了整个文件就废了，有了上面的作为参照就还可以修复IAT。

# 16 重定位表
一般没用，但记录了所有需要修正的位置，理解它也很重要！！
比如一个全局变量，查看汇编代码，发现它直接赋值给了内存里的绝对虚拟地址
但是当要加载这个模块的时候，ImageBase已经被别的程序占用了，只能让模块加载到别的ImageBase上，但是该全局变量则么办？需要重定位



3. 重定位表的位置
前面学习扩展PE头，里面最后一个成员DataDirectory是由16个IMAGE_DATA_DIRECTORY结构体（8字节）组成的数组；结构体形式如下；
数组内第6个元素是IMAGE_DIRECTORY_ENTRY_IMPORT；
保存的是导出表的相对虚拟地址RVA和大小
4. 重定位表结构
![[QQ截图20240307165425.png]]

后边跟着多个重定位块，都是这种结构
SizeOfBlock 以字节为的单位，决定了当前重定位块占多少字节，除去前面开头8字节后面的范围内就是待修改的数据的“偏移”
VirtualAddress 相当于地基，地基+“偏移”低12位=待重定位的数据；这样的设计能大大节省内存，而且待修改的越多越节省
直到遇到8个字节连续的`00`，就是重定位表结束了

	细节：
	为什么是“偏移”低12位？
	"偏移"只占2字节(16bit)，应为物理磁盘4k对齐，4K=4096字节，只占12bit就行，但这会导致高4位没用呀。所以决定当高四位=3的时候才是需要相加后重定位的数据。其它的都是内存对齐的垃圾数据！

# 17 注入shellcode
1. 什么是shellcode?
一段可以放在任意机器上执行的机器码，不依赖环境，放到任何地方都可以执行的机器码

2. Shellcode的编写规则
<1> 不能有全局变量
<2> 不能使用常量字符串 (char szMsg\[] = {'s','h','e','l','l','c','o','d','e'0})
<3> 不能使用系统调用 (TEB-PEB-3个链表-遍历链表-找到dll-导出表)
<4> 不能嵌套调用其他函数

2,3都可以解决如上括号

# 18 VirtualTable_Hook
1. 什么是HOOK
- HOOK是用来获取、更改程序执行的某些数据，或是用于更改程序执行流程的一种技术。
- HOOK的两种主要形式：
	1. 修改函数代码：INLINE HOOK(内联HOOK)
	2. 修改函数地址：IAT HOOK、SSDT HOOK、IDT HOOK、EAT HOOK、IRP HOOK

3. Virtual Table是什么？
虚表
虚函数（Virtual Function）是通过一张虚函数表（Virtual Table）来实现的，简称为V-Table。
一个类的虚函数的地址表，这张表解决了继承、覆盖的问题，保证其反应实际的函数。
在有虚函数的类的实例中，这个表被分配在了这个实例的内存里，当用父类的指针来操作一个子类的时候，这张虚函数表就像一个地图，指明了实际所应该调用的函数。

3. Virtual Table HOOK是什么？
修改间接调用地址

```C
#include <iostream>
#include <Windows.h>
class Base
{
public:
	virtual void Print()
	{
		std::cout << "Base..." << std::endl;
	}
};
 
void MyHookProc()
{
	std::cout << "Hook..." << std::endl;
}
 
int main(int argc, char* argv[])
{
	Base* pb = new Base();
 
	DWORD* pVtAddr = (DWORD*)*(DWORD*)pb;
	DWORD dwOldProct = 0;
	VirtualProtect(pVtAddr, 4, PAGE_READWRITE, &dwOldProct);
	*pVtAddr = (DWORD)MyHookProc;
 
	pb->Print();
 
	return 0;
}
```

# 19 IAT HOOK
[IATHOOK.cpp](https://github.com/smallzhong/drip-education-homework/blob/master/2015.6.9-IAT%20HOOK/IATHOOK.cpp)
# 20 INLINE HOOK
[InlineHook.cpp](https://github.com/smallzhong/drip-education-homework/blob/master/2015.6.10-INLINE%20HOOK/InlineHook.cpp)
# 21INLINE HOOK改进版

# 22 HOOK攻防
HOOK攻防的常用手段:
阶段一:
(防）检测JMP(E9)、检测跳转范围
(破）想方设法绕

阶段二:
(防)写个线程全代码校验/CRC校验
(破)修改检测代码、挂起检测函数

阶段三:
(防)先对相关API全代码校验,多个线程互相检测,并检测线程是否在活动中
(破）使用瞬时钩子/硬件钩子


# 23 瞬时HOOK过检测

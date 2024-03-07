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
保存的是导出表的虚拟内存地址RVA
![[QQ截图20240307134538.png]]
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


# 1. PE文件结构
![[897781438.gif]]

# 02 PE文件的两种状态

![[QQ截图20240305105327.png]]

各个结构体所占字节大小如下图： 
![[QQ截图20240305110246.png]]

在第一个结构体中的最后一个成员，指向的是PE文件头标志的位置，所以从IMAGE_DOS_HEADER最后一个成员结束位置到PE文件头标志的中间都是DOS块，这块长度是可变的，取决于e_fanew的值

![[QQ截图20240305145357.png]]

# 3. DOS头属性说明

![[QQ截图20240305151352.png]]

# 04 标准PE头属性说明
![[QQ截图20240305151515.png]]



![[QQ截图20240305151904.png]]




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
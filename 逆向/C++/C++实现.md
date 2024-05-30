# this指针
C++把函数放在结构体里面和外面没区别，唯一的区别是放在结构体里面汇编会多传一个参数，参数就是当前对象的首地址
类和结构体没区别，都是在栈里
```asm
lea ecx,[ebp-4]
```

# 构造-析构函数
### 构造函数
构造函数和类名一样而且不需要定义返回值
C++可以重载构造函数，**重载函数**，函数一样，只不过参数不一样，名字可以一样
>构造函数特：
1、与类同名
2、没有返回值
3、创建对象的时候执行
4、主要用于初始化
5、可以有多个(最好有一个无参的),称为重载其他函数也可以重载6、编译器不要求必须提供斗

### 析构函数
析构函数和构造函数同名只不过前面加了个`~`
析构函数不允许**重载**
>析构函数的特点:
1、只能有一个析构函数，不能重载
2、不能带任何参数
3、不能带返回值
4、主要用于清理工作
5、编译器不要求必须提供

### 继承本质
代码的复制
父类的指针可以指向子类的对象，但是子类的指针
使用`:`符号继承
### 多层继承
```C
struct X
{
int a;
int b;
}
struct Y:X
{
int c;
int d;
}
struct Z:Y
{
int e;
int f;
}
sizeof(Z)==24
```

# 多重继承
不太推荐使用
```C
struct X
{
int a;
int b;
}
struct Y
{
int c;
int d;
}
struct Z:X,Y
{
int e;
int f;
}
```

# 虚函数
代码上只是多了个virtual

当使用对象的方式调用的时候是直接调用，和普通函数没区别


![[Pasted image 20240510162341.png]]
\_\_chkesp是检查堆栈平衡的
当用指针方式调用虚函数的时候采用的是间接调用

类里面的函数不占字节，而且发现虚函数virtual占用了**4字节，而且在类里无论有几个虚函数只占一次，而且在对象的首地址存储，这个值保存的就是函数地址，也就是call edx里面的edx

其实这四个字节指向一个数组，这个数组存储所有的虚函数的地址，这就是设计思路，这就是虚函数表
### 虚函数：间接调用
```C
虚函数表：				
				
				
				
class Base				
{				
public:				
    void Function_1()				
    {				
        printf("Function_1...\n");				
    }				
    virtual void Function_2()				
    {				
        printf("Function_2...\n");				
    }				
};				
				
				
观察反汇编：				
				
void TestMethod()				
{				
	Base base;			
				
	base.Function_1();			
	00401090 8D 4D FC             lea         ecx,[ebp-4]			
	00401093 E8 9F FF FF FF       call        @ILT+50(Base::Function_1) (00401037)			
				
	base.Function_2();			
	00401098 8D 4D FC             lea         ecx,[ebp-4]			
	0040109B E8 65 FF FF FF       call        @ILT+0(Base::Function_2) (00401005)			
				
	Base* pb = &base;			
				
	pb->Function_1();			
	004010A6 8B 4D F8             mov         ecx,dword ptr [ebp-8]			
	004010A9 E8 89 FF FF FF       call        @ILT+50(Base::Function_1) (00401037)			
				
	pb->Function_2();			
	004010AE 8B 4D F8             mov         ecx,dword ptr [ebp-8]			
	004010B1 8B 11                mov         edx,dword ptr [ecx]			
	004010B3 8B F4                mov         esi,esp			
	004010B5 8B 4D F8             mov         ecx,dword ptr [ebp-8]			
	004010B8 FF 12                call        dword ptr [edx]			
}				
				
				
总结：				
				
1、通过对象调用时，virtual函数与普通函数都是E8 Call				
				
2、通过指针调用时，virtual函数是FF Call,也就是间接Call				

```

### 深入虚函数调用
```C
class Base							
{							
public:							
    int x;							
    int y;							
    virtual void Function_1()							
    {							
        printf("Function_1...\n");							
    }							
    virtual void Function_2()							
    {							
        printf("Function_2...\n");							
    }							
};							
							
							
							
1、当类中有虚函数时，观察大小的变化.							
							
Base base;							
							
printf("%x\n",sizeof(base));							
							
思考：多出这4个字节是什么?							
							
							
2、观察虚函数通过指针调用时的反汇编：							
							
							
pb->Function_1();							
0040D9E3 8B 4D F0             mov         ecx,dword ptr [ebp-10h]							
0040D9E6 8B 11                mov         edx,dword ptr [ecx]							
0040D9E8 8B F4                mov         esi,esp							
0040D9EA 8B 4D F0             mov         ecx,dword ptr [ebp-10h]							
0040D9ED FF 12                call        dword ptr [edx]							
							
							
pb->Function_2();							
0040D9F6 8B 45 F0             mov         eax,dword ptr [ebp-10h]							
0040D9F9 8B 10                mov         edx,dword ptr [eax]							
0040D9FB 8B F4                mov         esi,esp							
0040D9FD 8B 4D F0             mov         ecx,dword ptr [ebp-10h]							
0040DA00 FF 52 04             call        dword ptr [edx+4]							
							
							
							
总结：							
							
1、当类中有虚函数时，会多一个属性，4个字节							
							
2、多出的属性是一个地址，指向一张表，里面存储了所有虚函数的地址							

```

### 打印虚函数表
![[Pasted image 20240513150027.png]]


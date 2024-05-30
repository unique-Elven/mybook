# this指针
C++把函数放在结构体里面和外面没区别，唯一的区别是放在结构体里面汇编会多传一个参数，参数就是当前对象的首地址
类和结构体没区别，都是在栈里
```asm
lea ecx,[ebp-4]
```

# 构造-析构函数
[C++ 重载运算符和重载函数 | 菜鸟教程](https://www.runoob.com/cplusplus/cpp-overloading.html)
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
析构函数和构造函数同名只不过前面加了个`~`号
析构函数不允许**重载**
>析构函数的特点:
1、只能有一个析构函数，不能重载
2、不能带任何参数
3、不能带返回值
4、主要用于清理工作
5、编译器不要求必须提供

### 继承本质
代码的复制
父类的指针可以指向子类的对象，但是子类的指针不能指向父类
使用`:`号是符号继承
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
![[Pasted image 20240530122211.png]]
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
[C++虚函数的作用是什么 • Worktile社区](https://worktile.com/kb/ask/46820.html)
代码上只是多了个virtual

当使用对象的方式调用的时候是直接调用，和普通函数没区别


![[Pasted image 20240510162341.png]]
\_\_chkesp是检查堆栈平衡的
当用指针方式调用虚函数的时候采用的是间接调用

类里面的函数不占字节，而且发现虚函数virtual占用了**4字节**，而且在类里无论有几个虚函数只占一次，而且在对象的首地址存储，这个值保存的就是函数地址，也就是call edx里面的edx

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

### 练习：
```C++
#include <stdio.h>


/// <summary>
/// 虚函数表
/// </summary>
struct Base
{
public:
	virtual void fun1()
	{
		printf("虚函数1执行\n");
	}
	virtual void fun2()
	{
		printf("虚函数2执行\n");
	}
	virtual void fun3()
	{
		printf("虚函数3执行\n");
	}
};

void PrintVitualFunTables()
{
	Base base;
	Base* p;
	p = &base;
	//对象的前四个字节就是虚函数表
	printf("对象前四个字节虚函数表地址：0x%x 值是虚函数表首地址：0x%x\n", p, *(int *)p);

	//通过函数指针调用函数
	typedef void(*pfun)(void);
	pfun pFn;
	for (int i=0;i<=2;i++)
	{
		printf("函数地址：0x%x  ", *(((int *)(*(int *)p))+i) );
		int tmp = *(((int*)(*(int*)p))+i);//通过虚函数地址执行虚函数
		pFn = (pfun)tmp;
		pFn();
	}
	printf("******************************AAAAAAA**********************************\n");
}

/// <summary>
/// 单继承无覆盖
/// </summary>

struct Sub1:Base
{
public:
	virtual void fun4()
	{
		printf("虚函数4执行\n");
	}
	virtual void fun5()
	{
		printf("虚函数5执行\n");
	}
	virtual void fun6()
	{
		printf("虚函数6执行\n");
	}
};

void PrintInheritNoRewriteVitualFunTables()
{
	Sub1 base;
	Sub1* p;
	p = &base;
	//对象的前四个字节就是虚函数表
	printf("对象前四个字节虚函数表地址：0x%x 值是虚函数表首地址：0x%x\n", p, *(int*)p);

	//通过函数指针调用函数
	typedef void(*pfun)(void);
	pfun pFn;
	for (int i = 0; i <= 5; i++)
	{
		printf("函数地址：0x%x  ", *(((int*)(*(int*)p)) + i));
		int tmp = *(((int*)(*(int*)p)) + i);
		pFn = (pfun)tmp;
		pFn();
	}
	printf("******************************AAAAAAA**********************************\n");
}

/// <summary>
/// 单继承有覆盖
/// </summary>
struct Sub2 :Base
{
public:
	virtual void fun1()
	{
		printf("虚函数1执行\n");
	}
	virtual void fun2()
	{
		printf("虚函数2执行\n");
	}
	virtual void fun6()
	{
		printf("虚函数6执行\n");
	}
};

void PrintInheritRewriteVitualFunTables()
{
	Sub2 base;
	Sub2* p;
	p = &base;
	//对象的前四个字节就是虚函数表
	printf("对象前四个字节虚函数表地址：0x%x 值是虚函数表首地址：0x%x\n", p, *(int*)p);

	//通过函数指针调用函数
	typedef void(*pfun)(void);
	pfun pFn;
	for (int i = 0; i <= 3; i++)
	{
		printf("函数地址：0x%x  ", *(((int*)(*(int*)p)) + i));
		int tmp = *(((int*)(*(int*)p)) + i);
		pFn = (pfun)tmp;
		pFn();
	}
	printf("******************************AAAAAAA**********************************\n");
}

int main()
{
	PrintVitualFunTables();
	PrintInheritNoRewriteVitualFunTables();
	PrintInheritRewriteVitualFunTables();
	return 0;
}
```

结果打印：
```C++
对象前四个字节虚函数表地址：0x19fe0c 值是虚函数表首地址：0x503fcc
函数地址：0x454aef  虚函数1执行
函数地址：0x454aa4  虚函数2执行
函数地址：0x455f94  虚函数3执行
******************************AAAAAAA**********************************
对象前四个字节虚函数表地址：0x19fe0c 值是虚函数表首地址：0x5040d0
函数地址：0x454aef  虚函数1执行
函数地址：0x454aa4  虚函数2执行
函数地址：0x455f94  虚函数3执行
函数地址：0x4565fc  虚函数4执行
函数地址：0x4565f2  虚函数5执行
函数地址：0x456601  虚函数6执行
******************************AAAAAAA**********************************
对象前四个字节虚函数表地址：0x19fe0c 值是虚函数表首地址：0x504324
函数地址：0x45661a  虚函数1执行
函数地址：0x456615  虚函数2执行
函数地址：0x455f94  虚函数3执行
函数地址：0x45660b  虚函数6执行
******************************AAAAAAA**********************************
```

# 动态绑定-多态
在多继承无函数覆盖的情况下，有几个直接的父类就会有几个虚函数表，子类的虚函数是放在第一个虚表里面的后面，请看示例代码


在多继承有函数覆盖的情况下，覆盖的函数是哪个，就在哪个表里面，请看示例代码

多层继承无函数覆盖，虚表就一个，虚函数就按顺排列

多层继承有函数覆盖（子类覆盖了父类的父类的虚函数），还是虚表就一个，虚函数就按顺排列

多层继承有函数覆盖（子类覆盖了父类和爷爷类），还是虚表就一个，虚函数就按顺排列



绑定就是将函数调用与地址关联起来
总结：			
1、只有virtual的函数是动态绑定.			
2、动态绑定还有一个名字：多态			

```
_父类指针_也_可以_称为基_类指针_，当_父类_(基_类_)_指针指向_派生类(_子类_)_指针_的时候，_可以_触发“多态的效果
```

一帮写代码的时候也把析构函数定义为虚函数，因为以后一般在开发的时候，通用使用父类的指针数组访问子类的对象，如果不是虚函数，那么析构函数释放的还是父类的内存！
# this指针
在 C++ 中，this 指针是一个特殊的指针，它指向当前对象的实例。

在 C++ 中，每一个对象都能通过 this 指针来访问自己的地址。

this是一个隐藏的指针，可以在类的成员函数中使用，它可以用来指向调用对象。

当一个对象的成员函数被调用时，编译器会隐式地传递该对象的地址作为 this 指针。

友元函数没有 this 指针，因为友元不是类的成员，只有成员函数才有 this 指针。

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
[C++ 继承 | 菜鸟教程](https://www.runoob.com/cplusplus/cpp-inheritance.html)
当创建一个类时，您不需要重新编写新的数据成员和成员函数，只需指定新建的类继承了一个已有的类的成员即可。这个已有的类称为**基类**，新建的类称为**派生类**。

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

一个子类可以有多个父类,即多重继承
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
![[Pasted image 20240530123635.png]]

# class与struct的区别
## 将定义与实现分离
```C++
struct Test			
{			
	int x;		
	int y;		
	int z;		
	void Init(int x,int y,int z);			
};			

xxx.cpp			
void Test::Init(int x,int y,int z)			
{			
	this->x = x;		
	this->y = y;		
	this->z = z;		
}			
```

编译器默认class中的成员为private 而struct中的成员为public
![[Pasted image 20240530125409.png]]
总结：			
1、父类中的私有成员是会被继承的			
2、只是编译器不允许直接进行访问，但是可以通过指针访问

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
```C++
struct Base1						
{						
public:						
    virtual void Fn_1()						
    {						
        printf("Base1:Fn_1...\n");						
    }						
    virtual void Fn_2()						
    {						
        printf("Base1:Fn_2...\n");						
    }						
};						
struct Base2						
{						
public:						
    virtual void Fn_3()						
    {						
        printf("Base2:Fn_3...\n");						
    }						
    virtual void Fn_4()						
    {						
        printf("Base2:Fn_4...\n");						
    }						
};						
struct Sub:Base1,Base2						
{						
public:						
    virtual void Fn_5()						
    {						
        printf("Sub:Fn_5...\n");						
    }						
    virtual void Fn_6()						
    {						
        printf("Sub:Fn_6...\n");						
    }						
};						
int main(int argc, char* argv[])						
{						
	//查看 Sub 的虚函数表					
    Sub sub;						

	//通过函数指针调用函数，验证正确性					
    typedef void(*pFunction)(void);						

	//对象的前四个字节是第一个Base1的虚表					
	printf("Sub 的虚函数表地址为：%x\n",*(int*)&sub);					

	pFunction pFn;					

	for(int i=0;i<6;i++)					
	{					
		int temp = *((int*)(*(int*)&sub)+i);				
		if(temp == 0)				
		{				
			break;			
		}				
		pFn = (pFunction)temp;				
		pFn();				
	}					
	//对象的第二个四字节是Base2的虚表					
	printf("Sub 的虚函数表地址为：%x\n",*(int*)((int)&sub+4));					

	pFunction pFn1;					

	for(int k=0;k<2;k++)					
	{					
		int temp = *((int*)(*(int*)((int)&sub+4))+k);				
		pFn1 = (pFunction)temp;				
		pFn1();				
	}					

	return 0;					
}						

```
![[Pasted image 20240531102102.png]]

在多继承有函数覆盖的情况下，覆盖的函数是哪个，就在哪个表里面，请看示例代码
```C++
struct Base1						
{						
public:						
    virtual void Fn_1()						
    {						
        printf("Base1:Fn_1...\n");						
    }						
    virtual void Fn_2()						
    {						
        printf("Base1:Fn_2...\n");						
    }						
};						
struct Base2						
{						
public:						
    virtual void Fn_3()						
    {						
        printf("Base2:Fn_3...\n");						
    }						
    virtual void Fn_4()						
    {						
        printf("Base2:Fn_4...\n");						
    }						
};						
struct Sub:Base1,Base2						
{						
public:						
    virtual void Fn_1()						
    {						
        printf("Sub:Fn_1...\n");						
    }						
    virtual void Fn_3()						
    {						
        printf("Sub:Fn_3...\n");						
    }						
	virtual void Fn_5()					
    {						
        printf("Sub:Fn_5...\n");						
    }						
};						
int main(int argc, char* argv[])						
{						
	//查看 Sub 的虚函数表					
    Sub sub;						
	//通过函数指针调用函数，验证正确性					
    typedef void(*pFunction)(void);						
	//对象的前四个字节是第一个Base1的虚表					
	printf("Sub 的虚函数表地址为：%x\n",*(int*)&sub);					
	pFunction pFn;					
	for(int i=0;i<6;i++)					
	{					
		int temp = *((int*)(*(int*)&sub)+i);				
		if(temp == 0)				
		{				
			break;			
		}				
		pFn = (pFunction)temp;				
		pFn();				
	}					

	//对象的第二个四字节是Base2的虚表					
	printf("Sub 的虚函数表地址为：%x\n",*(int*)((int)&sub+4));
	pFunction pFn1;					
	for(int k=0;k<2;k++)					
	{					
		int temp = *((int*)(*(int*)((int)&sub+4))+k);				
		pFn1 = (pFunction)temp;				
		pFn1();				
	}					
	return 0;					
}						

```
![[Pasted image 20240531102205.png]]


多层继承无函数覆盖，虚表就一个，虚函数就按顺排列,请看下示例代码
```C++
struct Base1					
{					
public:					
    virtual void Fn_1()					
    {					
        printf("Base1:Fn_1...\n");					
    }					
    virtual void Fn_2()					
    {					
        printf("Base1:Fn_2...\n");					
    }					
};					
struct Base2:Base1					
{					
public:					
    virtual void Fn_3()					
    {					
        printf("Base2:Fn_3...\n");					
    }					
    virtual void Fn_4()					
    {					
        printf("Base2:Fn_4...\n");					
    }					
};					
struct Sub:Base2					
{					
public:					
    virtual void Fn_5()					
    {					
        printf("Sub:Fn_5...\n");					
    }					
    virtual void Fn_6()					
    {					
        printf("Sub:Fn_6...\n");					
    }					
};					
int main(int argc, char* argv[])					
{					
	//查看 Sub 的虚函数表				
    Sub sub;					
	//观察大小：虚函数表只有一个				
	printf("%x\n",sizeof(sub));				
	//通过函数指针调用函数，验证正确性				
    typedef void(*pFunction)(void);					
	//对象的前四个字节是就是虚函数表				
	printf("Sub 的虚函数表地址为：%x\n",*(int*)&sub);				
	pFunction pFn;				
	for(int i=0;i<6;i++)				
	{				
		int temp = *((int*)(*(int*)&sub)+i);			
		if(temp == 0)			
		{			
			break;		
		}			
		pFn = (pFunction)temp;			
		pFn();			
	}				
	return 0;				
}					
```
![[Pasted image 20240531102415.png]]

多层继承有函数覆盖（子类覆盖了父类的父类的虚函数），还是虚表就一个，虚函数就按顺排列

多层继承有函数覆盖（子类覆盖了父类和爷爷类），还是虚表就一个，虚函数就按顺排列

绑定：分为前期绑定和后期绑定（动态绑定，运行时绑定）
C++的动态绑定是通过虚表来实现的
```C++
/// <summary>
/// 什么是绑定？
/// </summary>
class MyClass
{
public:
	MyClass();
	void Function_1();
	virtual void Function_2();
	~MyClass();

public:
	int x;

};

MyClass::MyClass()
{
	x = 100;
}

MyClass::~MyClass()
{
	printf("******************************MyClass**********************************\n");
}
void MyClass::Function_1()
{
	printf("MyClass:Function_1...\n");
}
void MyClass::Function_2()
{
	printf("MyClass:Function_2...\n");
}


class MyClass2:public MyClass
{
public:
	MyClass2();
	void Function_1();
	virtual void Function_2();
	~MyClass2();

public:
	int x;

};

MyClass2::MyClass2()
{
	x = 200;
}

MyClass2::~MyClass2()
{
	printf("******************************MyClass2**********************************\n");
}
void MyClass2::Function_1()
{
	printf("MyClass2:Function_1...\n");
}
void MyClass2::Function_2()
{
	printf("MyClass2:Function_2...\n");
}

void TestBound(MyClass* pb)
{
	int n = pb->x;
	printf("%x\n", n);//
	pb->Function_1();//函数调用 前期绑定，编译期绑定
	pb->Function_2();//体现出了不同行为成为多态 运行期绑定 动态绑定 晚绑定
}

void BinDing()
{
	MyClass2 pb;
	TestBound(&pb);
}

```
![[Pasted image 20240530133744.png]]
绑定就是将函数调用与地址关联起来
总结：			
1、只有virtual的函数是动态绑定.			
2、动态绑定还有一个名字：多态			

```
父类指针也可以称为基类指针，当父类(基类)指针指向派生类(子类)指针的时候，可以触发“多态的效果
```

一帮写代码的时候也把析构函数定义为虚函数，因为以后一般在开发的时候，通用使用父类的指针数组访问子类的对象，如果不是虚函数，那么析构函数释放的还是父类的内存！
```C++
/// <summary>
/// 多态
/// </summary>
class Class1
{
public:
	Class1();
	~Class1();
	virtual void print();
public:
	int x;
	int y;
};
Class1::Class1()
{
	x = 1;
	y = 2;
}
Class1::~Class1()
{
}
void Class1::print()
{
	printf("Class1：%x,%x\n", x, y);
}




class Class2:public Class1
{
public:
	Class2();
	~Class2();
	virtual void print();
public:
	int A;
};
Class2::Class2()
{
	x = 4;
	y = 5;
	A = 6;
}
Class2::~Class2()
{
}
void Class2::print()
{
	printf("Class2：%x,%x,%x\n", x, y,A);
}





class Class3 :public Class1
{
public:
	Class3();
	~Class3();
	virtual void print();
public:
	int B;
};
Class3::Class3()
{
	x = 7;
	y = 8;
	B = 9;
}
Class3::~Class3()
{
}
void Class3::print()
{
	printf("Class3：%x,%x,%x\n", x, y, B);
}


void  Polymorphism()
{
	Class1 c1;
	Class2 c2;
	Class3 c3;
	Class1* arr[] = { &c1,&c2,&c3 };
	for (int i = 0; i < 3; i++)
	{
		arr[i]->print();
	}
}
```
# 模板

例如当设计了一个算法后，暂时不知道需要传入运算的数据类型，就可以用模板代替参数类型
模板生成出来汇编代码和源代码一摸一样
ps: 向左移一位就是除以2，右移一位就是乘以2，位移运算更加高效

以下用一个算法为了加强通用性举例
```C++
/// <summary>
/// 模板
/// </summary>
template<class T>
//冒泡排序算法
void Sort(T arr, int length)
{
	for (int i=0;i<length-1;i++)
	{
		for (int j = 0;j<length-1-i;j++)
		{
			if (arr[j]>arr[j+1])
			{
				int temp = arr[j + 1];
				arr[j + 1] = arr[j];
				arr[j] = temp;
			}
		}
	}
}
//二分法查找
template<class T,class E>
int Find(T arr,int nlength,E nElement )
{
	int nBegin = 0, nEnd = nlength - 1, Idenx;
	while (nBegin <= nEnd)
	{
		Idenx = (nBegin+nEnd) / 2;//(nlength) >> 1
		if (nElement > arr[Idenx])
		{
			nBegin = Idenx + 1;
		}
		else if (nElement < arr[Idenx])
		{
			nEnd = Idenx - 1;
		}
		else
		{
			printf("Find Success: [Idenx]=%d,[Value]=%d\n", Idenx, arr[Idenx]);
			return Idenx;
		}
	}
	return -1;
}
void TestAlgorithm()
{
	char arr[] = { 2,6,1,3,8,9,4,12,11,10 };
	int length;
	char nElement = {10};
	length = sizeof(arr) / sizeof(char);
	Sort(arr, length);
	for (int i = 0; i < 10; i++)
	{
		printf("%d\n", arr[i]);
	}
	
	Find(arr, length, nElement);
	printf("******************************模板template<>**********************************\n");
}
```

# 引用类型-友元函数-运算符重载
引用：相当于取别名
指针的参数传递和引用的参数传递汇编代码完全一样，但是引用更安全，不像指针一样可以乱指，因为引用不能重新赋值地址，这是由编译器维护的
而且使用指针类型进行参数存传递和变量是一样的，调用函数的手看出出来

友元：声明关键字friend就是告诉类里面的的方法是类的朋友，使用场景是当一个函数需要访问一个类里面的私有成员的时候；但是友元函数不是成员函数，不能用this指针

运算符重载：

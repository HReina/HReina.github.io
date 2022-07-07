---
title: C++对C语言的拓展
date: 2022-03-07 15:13:25
tags:
    - c++
categories: 
	- C++语法
comments: true
---

> ​                                                                   C++对C语言的拓展

<!-- more -->

#### 引用

##### 变量名

变量名实质上是一段连续存储空间的别名，是一个标号(门牌号)；
通过变量来申请并命名内存空间；
通过变量的名字可以使用存储空间。 

##### 引用的概念

变量名，本身是一段内存的引用，即别名(alias)。引用可以看作一个已定义变量的别名。 
引用的语法：Type& name = var; 

```c++
#include<iostream>
using namespace	std;	
int	main(void)	
{	
	int	a =	10;	    //c编译器分配4个字节内存,	a内存空间的别名
	int	&b = a;		//b就是a的别名
    return 0;	
}	
```

##### 规则

* 引用没有定义，是一种关系型声明。声明它和原有某一变量(实体)的关系。故而类型与原类型保持一致,且不分配内存。与被引用的变量有相同的地址。 
* 声明的时候必须初始化，一经声明,不可变更。
* 可对引用，再次引用。多次引用的结果，是某一变量具有多个别名。 
* &符号前有数据类型时，是引用。其它皆为取地址。 

```c++
int	main(void)	
{	
	int	a,b;	
	int	&r	=	a;	
	int	&r	=	b;	//错误,不可更改原有的引⽤关系
	float &rr =	b;	//错误,引⽤类型不匹配 cout<<&a<<&r<<endl;//变量与引⽤具有相同的地址。
	int	&ra	= r;	//可对引⽤更次引⽤,表⽰a变量有两个别名,分别是r和ra
	
    return 0;	
}
```

##### 引用作为函数参数

普通引用在声明时必须用其它的变量进行初始化，引用作为函数参数声明时不进行初始化。

```c++
#include <iostream>
using namespace	std;	
struct Teacher	
{	
	char name[64];	
	int	age	;	
};
void printfT(Teacher *pT)	
{	
	cout<< pT->age <<endl;	
}	
//pT是t1的别名，相当于修改了t1
void printfT2(Teacher &pT)	
{	
	pT.age = 33;	
	cout<< pT.age <<endl;	
}	
//pT和t1的是两个不同的变量
void printfT3(Teacher pT)	
{	
	cout<< pT.age <<endl;	
	pT.age = 45;	   //只会修改pT变量，不会修改t1变量
}	
int	main(void)	
{	
	Teacher	t1;	
	t1.age = 35;	
	printfT(&t1);	
	printfT2(t1);	                 //pT是t1的别名
	printf("t1.age:%d\n", t1.age);	 //33
	printfT3(t1);                    //pT是形参，t1 copy⼀份数据给pT
	printf("t1.age:%d\n", t1.age);	 //33
	
    return 0;	
}
```

##### 引用的意义

* 引用作为其它变量的别名而存在，因此在一些场合可以代替指针
* 引用相对于指针来说具有更好的可读性和实用性

```c++
void swap(int a, int b);	//⽆法实现两数据的交换
void swap(int *p, int *q);	//开辟了两个指针空间实现交换
void swap(int &a, int &b){	
	 int tmp;	
	 tmp = a;	
     a = b;	
	 b = tmp;
}
```

> C++中引入引用后,可以用引用解决的问题。避免用指针来解决。

##### 引用的本质

* 引用在C++中的内部实现是一个常指针Type& name <===> Type* const name

* C++编译器在编译过程中使用常指针作为引用的内部实现，因此引用所占用的空间大小与指针相同。

* 从使用的角度，引用会让人误会其只是一个别名，没有自己的存储空间。这是C++为了实用性而做出的细节隐藏。

```c++
#include <iostream>
using namespace	std;	
void func(int &a)	
{				
	a =	5;	
}				
void func(int *const a)	
{				
	*a = 5;	
}
int	main()	
{	
	int	x =	10;	
	func(x);	
	return 0;	
}
/*
间接赋值的3个必要条件 
1 定义两个变量 （一个实参一个形参） 
2 建立关联 实参取地址传给形参 
3 *p形参去间接的修改实参的值 

引用在实现上，只不过是把间接赋值成立的三个条件的后两步和二为一。
当实参传给形参引用的时候，只不过是c++编译器帮我们程序员手工取了一个实参地址，
传给了形参引用（常量指针）。
*/
```

##### 引用作为函数的返回值

1、当函数返回值为引用时，若返回栈变量，不能成为其它引用的初始值（不能作为左值使用）

```c++
include <iostream>	
using namespace	std;	
int	getA1()	
{	
	int	a;	
    a =	10;	
	return a;	
}	
int& getA2()	
{	
	int	a;	
	a =	10;	
	return a;	
}	
int	main(void)	
{	
	int	a1 = 0;	
	int	a2 = 0;	
	
    //值拷⻉
	a1 = getA1();	
	
    //将⼀个引⽤赋给⼀个变量，会有拷⻉动作
	//理解：编译器类似做了如下隐藏操作，a2 =	*(getA2())
	a2 = getA2();	
	
    //将⼀个引⽤赋给另⼀个引⽤作为初始值，由于是栈的引⽤，内存⾮法
	int	&a3	= getA2();	
	cout<<"a1 = "<<a1<<endl;	
	cout<<"a2 =	"<<a2<<endl;	
	cout<<"a3 =	"<<a3<<endl;	
	return 0;	
}
```

2、当函数返回值为引用时，若返回静态变量或全局变量可以成为其他引用的初始值（可作为右值使用，也可作为左值使用）

```c++
#include <iostream>
using namespace	std;	
int	getA1()	
{	
	static int a;	
	a =	10;	
	return a;	
}	
int& getA2()	
{	
	static int a;	
	a =	10;	
	return a;	
}	
int	main(void)	
{	
   int a1 =	0;	
   int a2 =	0;	
				
   //值拷⻉
   a1 =	getA1();	
				
   //将⼀个引⽤赋给⼀个变量，会有拷⻉动作
   //理解： 编译器类似做了如下隐藏操作，a2 = *(getA2())
   a2 =	getA2();	
   
   //将⼀个引⽤赋给另⼀个引⽤作为初始值，由于是静态区域，内存合法
   int &a3 = getA2();	
	cout<<"a1 =	"<<a1<<endl;	
	cout<<"a2 =	"<<a2<<endl;	
	cout<<"a3 =	"<<a3<<endl;	
	return 0;	
}
```

> 引用作为函数返回值，如果返回值为引用可以当左值，如果返回值为普通变量不可以当左值。 

##### 指针引用

```c++
#include <iostream>
using namespace	std;	
struct Teacher	
{	
	char name[64];	
	int	age;	
};	
//在被调⽤函数 获取资源
int	getTeacher(Teacher **p)	
{	
	Teacher	*tmp = NULL;	
	if(p ==	NULL){	
	   return -1;	
	}	
	tmp	= (Teacher *)malloc(sizeof(Teacher));	
	if(tmp == NULL){	
      return -2;	
	}	
	tmp->age = 33;	
	// p是实参的地址 *实参的地址 去间接的修改实参的值
	*p = tmp;	
	return 0;	
}	
//指针的引⽤ 做函数参数
int getTeacher2(Teacher* &myp)	
{	
    //给myp赋值 相当于给main函数中的pT1赋值
	myp	= (Teacher *)malloc(sizeof(Teacher));	
	if(myp == NULL){	
	return -1;	
	}	
	myp->age = 36;	
	return 0;	
}	
void FreeTeacher(Teacher *pT1)	
{	
	if(pT1 == NULL){	
	return ;	
    }	
	free(pT1);	
}	
int	main(void)	
{	
	Teacher	*pT1 = NULL;	
	//1	c语⾔中的⼆级指针
	getTeacher(&pT1);	
	cout<<"age:"<<pT1->age<<endl;	
	FreeTeacher(pT1);	
	//2	c++中的引⽤（指针的引⽤）
	//引⽤的本质 间接赋值后2个条件 让c++编译器帮我们程序员做了.
	getTeacher2(pT1);	
	cout<<"age:"<<pT1->age<<endl;	
	FreeTeacher(pT1);	
	return 0;	
}	
```

##### const 引用

const 引用有较多使用。它可以防止对象的值被随意修改。因而具有一些特性。 
1、const 对象的引用必须是 const 的,将普通引用绑定到 const 对象是不合法的。这个原因比较简单。既然对象是 const 的，表示不能被修改，引用当然也不能修改，必须使用 const 引用。实际上,

```c++
const int a=1; 
int &b=a;
```

这种写法是不合法 的,编译不过。 
2、const 引用可使用相关类型的对象（常量，非同类型的变量或表达式）初始化。这个是 const 引用与普通引用最大的区别。

```c++
const int &a=2;  //是合法的。

double x=3.14; 
const int &b=a;  //也是合法的。
```

```c++
#include <iostream>
using namespace	std;	
int	main(void)	
{	
   //普通引⽤
   int a = 10;	
   int &b =	a;	
   cout<<"b = "<< b <<endl;	
   
   //常引⽤
   int x =	20;	
   const int &y	= x;  //常引⽤是限制变量为只读 不能通过y去修改x了
   //y	=	21;      /error
   return 0;	
}
```

##### const 引用的原理

 const 引用的目的是，禁止通过修改引用值来改变被引用的对象。const 引用的初始化特性较为微妙。

```c++
double val = 3.14;	
const int &ref = val;	
double & ref2 =	val;	
cout<<ref<<" "<<ref2<<endl;	 // 3  3.14
val	= 4.14;	
cout<<ref<<" "<<ref2<<endl;  // 3  4.14
```

上述输出结果为 3、3.14 和 3、4.14。因为 ref 是 const 的,在初始化的过程中已经给定值，不允许修改。而被引用的对象是 val，是非 const 的，所以 val 的修改并未影响 ref 的值，而 ref2 的值发生了相应的改变。那么，为什么非 const 的引用不能使用相关类型初始化呢？实际上，const 引用使用相关类型对象初始化时发生了如下过程：

```c++
int	temp = val;	
const int &ref = temp;
```

如果 ref 不是 const 的，那么改变 ref 值，修改的是 temp，而不是 val。期望对 ref 的赋值会修改 val 的程序员会发现 val 实际并未修改。

> 结论： 
> 1）const int & e 相当于 const int * const e 
> 2）普通引用 相当于 int *const e 
> 3）当使用常量（字面量）对const引用进行初始化时，C++编译器会为常量值
>       分配空间，并将引用名作为这段空间的别名 
> 4）使用字面量对const引用初始化后，将生成一个只读变量 

#### inline内联函数

C语言中有宏函数的概念。宏函数的特点是内嵌到调用代码中去，避免了函数调用的开销。但是由于宏函数的处理发生在预处理阶段，缺失了语法检测和有可能带来的语意差错。

##### 内联函数基本概念  

C++提供了 inline 关键字，实现了真正的内嵌。

```c++
#include<iostream>
using namespace std;
inline void func(int a)
{
     a = 20;
     cout << a <<endl; .
}
int main(void)
{
      func(10);
       /*
         //编译器将内联函数的函数体直接展开
      {
           a = 20;
           cout  <<  a  <<endl ;
      }
       */
    
       return 0;
}
```

特点：

1、内联函数声明时inline关键字必须和函数定义结合在一起，否则编译器会直接忽略内联请求。

2、C++编译器直接将函数体插入在函数调用的地方 。

3、内联函数没有普通函数调用时的额外开销(压栈，跳转，返回)。

4、内联函数是一种特殊的函数，具有普通函数的特征（参数检查，返回类型等）。

5、内联函数由编译器处理，直接将编译后的函数体插入调用的地方，宏代码片段由预处理器处理，进行简单的文	  本替换，没有任何编译过程。

6、C++中内联编译的限制： 

* 不能存在任何形式的循环语句
* 不能存在过多的条件判断语句 
* 函数体不能过于庞大 
* 不能对函数进行取址操作 
* 函数内联声明必须在调用语句之前

7、编译器对于内联函数的限制并不是绝对的，内联函数相对于普通函数的优势只是省去了函数调用时压栈，跳转和返回的开销。因此，当函数体的执行开销远大于压栈，跳转和返回所用的开销时，那么内联将无意义。

##### 内联函数 vs 宏函数

```c++
#include <iostream>
#include <string.h>
using namespace std;

#if 0
优点:   内嵌代码，辟免压栈与出栈的开销
缺点:   代码替换，易使生成代码体积变大，易产生逻辑错误。
#endif

#define SQR(x) ((x)*(x))

#if 0
优点:   高度抽象,避免重复开发
缺点:   压栈与出栈,带来开销
#endif

inline int sqr(int x)
{
    return x*x;
}
int main() 
{
     int i=0;
     while(i<5)
     {
         //printf("%d\n" , SQR(i++));
         printf("%d\n" ,sqr(i++));
     }
return 0;
}
```

##### 内联函数总结

* 优点: 避免调用时的额外开销(入栈与出栈操作) 
* 代价: 由于内联函数的函数体在代码段中会出现多个“副本”，因此会增加代码段的空间。 
* 本质: 以牺牲代码段空间为代价，提高程序的运行时间的效率。 
* 适用场景: 函数体很“小”，且被“频繁

#### 默认参数和占位参数

通常情况下，函数在调用时，形参从实参那里取得值。对于多次调用同一函数同一实参时，C++给出了更简单的处理办法。给形参以默认值，这样就不用从实参那里取值了。

```c++
//单个默认参数  若你填写参数，使⽤你填写的，不填写使用默认
void myPrint(int x = 3)	
{	
	cout<<"x："<< x <<endl;	
}

//多个默认参数  在默认参数规则，如果默认参数出现，那么右边的都必须有默认参数
float volume(float length, float weight	= 4, float high	= 5)	
{	
	return length*weight*high;	
}	
int	main()	
{	
	float v	= volume(10);	
	float v1 = volume(10,20);	
	float v2 = volume(10,20,30);	
	cout<<v<<endl;	
	cout<<v1<<endl;	
	cout<<v2<<endl;	
				
    return 0;	
}	
```

##### 默认参数规则

只有参数列表后面部分的参数才可以提供默认，参数值一旦在一个函数调用中开始使用默认参数值，那么这个参数后的所有参数都必须使用默认参数值。

##### 占位参数

```c++
#include <iostream>
/*
   函数占位参数		
   占位参数只有参数类型声明，⽽没有参数名声明
   ⼀般情况下，在函数体内部⽆法使⽤占位参数
*/
int	func(int a, int	b, int)	
{	
	return a + b;	
}	
int	main()	
{	
	func(1,	2);	 //error，必须把最后⼀个占位参数补上。										
    printf("func(1, 2, 3)	=	%d\n", func(1, 2, 3));	
	return 0;	
}
```

#### 函数重载

函数重载(Function Overload)：用同一个函数名定义不同的函数，当函数名和不同的参数搭配时函数的含义不同。

##### 重载规则

1、函数名相同。 
2、参数个数不同，参数的类型不同，参数顺序不同，均可构成重载。 
3、返回值类型不同则不可以构成重载。 

##### 调用规则

1、严格匹配,找到则调用。 
2、通过隐式转换寻求一个匹配、找到则调用。

编译器调用重载函数的准则: 
1.将所有同名函数作为候选者 
2.尝试寻找可行的候选函数 
3.精确匹配实参 
4.通过默认参数能够匹配实参 
5.通过默认类型转换匹配实参 
6.匹配失败 
7.最终寻找到的可行候选函数不唯一，则出现二义性，编译失败。 
8.无法匹配所有候选者，函数未定义，编译失败。 

##### 重载底层实现

C++利用 name mangling(倾轧)技术，来改名函数名，区分参数不同的同名函数。 
实现原理：用 v c i f l d 表示 void char int float long double 及其引用。

##### 函数重载总结

* 重载函数在本质上是相互独立的不同函数 
* 函数的函数类型是不同的
* 函数返回值不能作为函数重载的依据
* 函数重载是由函数名和参数列表决定的
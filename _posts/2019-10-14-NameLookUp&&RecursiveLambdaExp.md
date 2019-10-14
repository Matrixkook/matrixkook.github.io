---
layout:     post
title:      NameLookUp&&RecursiveLambdaFuc
subtitle:   Trick!Trick!
date:       2018-10-14
author:     Matrixtang
header-img: img/2019-10-14.png
catalog: true
tags:
    - c++
    - Trick
---
这两个问题本身没有什么关联,但是都是我在学习c++时遇到的问题.想在这里做个记录.

------

# Two-Phase Name Lookup



>C++ has more than its fair share of dark, dank corners, especially where
>templates are concerned. One of the most vexing is "two-phase name 
>lookup", which involves lookup for any names that occur in the body of a
>template. 

[参考连接 llvm blog](http://blog.llvm.org/2009/12/dreaded-two-phase-name-lookup.html)



## 名称查找

有了解过编译原理的同学,都应该知道符号表存在的意义

```c++
SymbolTable
{
	("=",OPeq),
	("int",ValName),
	(" ",space)
}//伪代码 看懂意思就行

```



语句的解析是由编译器前端也就是解析器完成的. 在过程中由标识符(identifer)来告诉编译器在不同作用域中遇到重名符号时改如何解决.

比如这个例子

```c++
struct A  { int a; };
struct AB { int a, b; };
struct C  { int c; };

template <typename T> foo(T& v0, C& v1)
{
    v0.a = 1;// unknown
    v1.a = 2;// make sense
    v1.c = 3;// wrong
}
```

可以看出  在函数foo中, v1明显有错误,因为它的成员中并没有a这个成员.

而v0被检查出来问题, 只有在foo被实例化时才能判断出有无错误



> **14.6 名称解析（Name resolution）**

> **1)** 模板定义中能够出现以下三类名称：

> - 模板名称、或模板实现中所定义的名称；
> - 和模板参数有关的名称；
> - 模板定义所在的定义域内能看到的名称。

> …

> **9)** … 如果名字查找和模板参数有关，那么查找会延期到模板参数全都确定的时候。 …

> **10)** 如果（模板定义内出现的）名字和模板参数无关，那么在模板定义处，就应该找得到这个名字的声明。…

> **14.6.2 依赖性名称（Dependent names）**

> **1)** …（模板定义中的）表达式和类型可能会依赖于模板参数，并且模板参数会影响到名称查找的作用域 …  如果表达式中有操作数依赖于模板参数，那么整个表达式都依赖于模板参数，名称查找延期到**模板实例化时**进行。并且定义时和实例化时的上下文都会参与名称查找。（依赖性）表达式可以分为类型依赖（类型指模板参数的类型）或值依赖。

> **14.6.2.2 类型依赖的表达式**

> **2)** 如果成员函数所属的类型是和模板参数有关的，那么这个成员函数中的`this`就认为是类型依赖的。

> **14.6.3 非依赖性名称（Non-dependent names）**

> **1)** 非依赖性名称在**模板定义**时使用通常的名称查找规则进行名称查找。

[Working Draft: Standard of Programming Language C++, N3337](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3337.pdf)



标准把所有的名称分为3类

- 模板名称、或模板实现中所定义的名称；
- 和模板参数有关的名称；
- 模板定义所在的定义域内能看到的名称。

```c++
int a;
struct B { int v; }
template <typename T> struct X
{
    B b;                  // B 是非依赖性名称(和模板定义域一致)，b 是在模板中定义的名称
    T t;                  // T 是模板参数有关的名称
    X* anthor;            // X 这里代指 X<T>，是第一类
    typedef int Y;        // int 是第三类
    Y y;                  // Y 是第一类 y是第一类
    
    void foo() 
    {
       b.v += y;          // b 是第一类，非依赖性名称
       b.v *= T::s_mem;   // T::s_mem 是第二类
                          // s_mem的作用域由T决定
                         
    }
};
```

名称查找会在模板定义和实例化时各做一次,分别处理非依赖性名称和依赖性名称的查找.

----



## 两阶段名称查找

考虑这个例子

```c++

template <typename T> struct x
{
 //do something     
};
X<int> xi; 
X<float> xf;

```

此时如果X中有一些与模板参数无关的错误，如果名称查找/语义分析在两个阶段完成，那么这些错误会很早、且唯一的被提示出来；但是如果一切都在实例化时处理，那么可能会导致不同的实例化过程提示同样的错误,如果实例化了大量的类, 可能会出现大量的报错信息.

```c++
template <typename T> struct X {};
template <typename T> struct Y
{
	void foo()
	{
		X<T> instance0;
		typename X<T>::MemberType instance1;//显然错误的
	}
};

void poo() {
	Y<int> y1;
	Y<float> y2;
	y1.foo();
	y2.foo();
}
```

在MSVC下对模板只有以下提示

```c++
MemberType: 不是“X<T>”的成员
error C2039:         with
error C2039:         [
error C2039:             T=float
error C2039:         ]
message :  参见“X<T>”的声明
message :         with
message :         [
message :             T=float
message :         ]
```

只指出了float实例的问题

可能是将不同的错误合并在一起



而gcc并不是这样

```c++
test.cpp: In instantiation of ‘void Y<T>::foo() [with T = int]’:
test.cpp:29:12:   required from here
test.cpp:15:36: error: no type named ‘MemberType’ in ‘struct X<int>’
          typename X<T>::MemberType instance1;
                                    ^~~~~~~~~
test.cpp: In instantiation of ‘void Y<T>::foo() [with T = float]’:
test.cpp:15:36: error: no type named ‘MemberType’ in ‘struct X<float>’
```









-----------



# Recursive Lambda Fuction

​		使用“Lambda表达式来构造一个递归函数”的难点是因为“我们正在构造的东西是没有名字的”，因此“我们无法调用自身”。那么，如果我们换种写法，把我们正在调用的匿名函数作为参数传给自己就可以了.

我们想要使用Lambda表达式编写一个计算递归的fac函数，一开始我们总会设法这样做：

```c++
auto f1 = [](int i)->int {return (i == 0 ? f1(i) * 1 : 1); };
```



但是显然办不到 , 编译告诉我们 使用 auto 类型说明符声明的变量不能出现在其自身的初始值设定项

于是这样呢?

```c++
int (*f1)(int) =[](int i)->int {return (i == 0 ? f1(i) * i : 1); };
```



手动指定f1 可以吗?

很不幸 编译器告诉我们:封闭函数局部变量不能在 lambda 体中引用，除非其位于捕获列表中 ,而捕获列表并不能捕获自身.

于是我们写成

```c++
static int (*f1)(int i) = [](int i)->int { return ( i > 0 ? f1(i - 1) * i : 1 ); };
//Trick here is that lambdas can access static variables
//and you can convert stateless ones to function pointer.
```

为啥要这么写呢?

将f1声明为静态变量是为了让这个函数指针不被捕获到这个lambda表达式

一个没有捕获列表的lambda表达式才能被转换成一个函数指针

[参考资料](https://en.cppreference.com/w/cpp/language/lambda)




---
layout:     post
title:        GSL and C++ Core Guidelines 
subtitle:   Start using them now
date:       2020-06-07
author:     Matrixtang
header-img: img/cpp_core_guidelines_logo_text-1.png
catalog: true
tags:
    - c++ lang
---





# GSL and C++ Core Guidelines 


## 开发准则支持库(GSL)



准则支持库（GSL）包含由标准C ++基金会维护的C ++核心准则建议使用的功能和类型。 此存储库包含Microsoft的GSL实现。

该库包括span <T>，string_span，owner <>等类型。

指针所有权：明确定义谁拥有指针是防止内存泄漏和指针损坏的简单方式。一般来说，定义所有权的最佳方式使用智能指针 unique_ptr{} 和 sharped_ptr{} 。但有时候，某些情况下是用不了的，这些边缘情况就可以考虑用 GSL 来处理。
管理期望：GSL 还可以用来定义函数期待的输入和保证的输出。
指针算术：指针的算术运算是导致很多内存问题和漏洞的重要原因。GSL 限制指针运算（或者至少只用在测试良好的库上）可以避免这些问题。

[Microsoft的实现](https://github.com/microsoft/GSL)



##　如何使用C++ Core Guidelines

由 Bjarne Stroustrup 和其他人创建， C++核心准则是使用新式C++安全有效的指南。 这些指南强调了静态类型安全和资源安全性。 它们确定了消除或最小化语言中最容易出错的部分的方法，并建议如何以可靠的方式使代码更简单、更具性能。 这些准则由标准C++基础维护。 若要了解详细信息，请参阅[GitHub](https://github.com/isocpp/CppCoreGuidelines)上的文档、 [ C++核心准则](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)和访问C++核心准则文档项目文件。

核心准则很长,而且非常复杂.对于开发人员来说,它只是参考,除了几个基本的准则[CppCon 2017: Kate Gregory “10 Core Guidelines You Need to Start Using Now”](https://www.youtube.com/watch?v=XkDEzfpdcSg&pbjreload=101)

我们需要了解之外,其他的可以在编写代码时,参阅指南.C++ Core Guidelines 让你在摇摆不定时 ,给于更好的选择.并给出为什么这样做的原因和好处.



-----



## 不用raw pointer 传递所有权

>### I.11: Never transfer ownership by a raw pointer (`T*`) or reference (`T&`)
>
>##### Reason
>
>If there is any doubt whether the caller or the callee owns an object, leaks or premature destruction will occur.

比如以下代码

```c++
Object* foo(args)
{
    ...
    Object * pO =  new Object(args);
    ...
    return pO;
}
```



如果我们在后面的代码中这样使用

```c++

bar()
{
    auto p1 =  foo(args);
    DoSomethingMayModify(p1); // p1 may be assigned nullptr or other value
    delete p1;
}

```

​	

我们无法确定在用delete语义时,p1的行为时可控的,我们销毁的不一定是我们想要销毁的对象

或者p1 在DoSomethingMayModify函数里已经被销毁






### 可供选择的方案


####　Return by value

显而易见的,直接用value复制一份数据 不会有这个问题.当需要传递的对象很小时,copy的代价并不是那么大



#### 用返回值表示是否被修改

```c++
bar()
{
    auto p1 =  foo(args);
    auto Res = 
        DoSomethingMayModify(p1); // p1 may be assigned nullptr or other value
    if( Res == NO_MODIFY_PTR)
    	delete p1;
    else
        delete FindOri(p1); // find original p1 
}
bool DoSomethingMayMOidify(Some ptr); //如果修改了p1 就应该返回修改过
enum{NO_MODIFY_PTR , ...}
```



####　使用智能指针



```c++

bar()
{
    auto p1 =  std:make_unique<Object>( foo(args) );
    auto p2 = DoSomethingMayModify(std::move(p1)); //返回指针 ,给与所有权
    
    assert(p1 == nullptr);
    delete p1; 
    delete p2;// 直接销毁p2 , p1 是空指针 不会被销毁两次
}
std::unique_ptr<Object> 
DoSomethingMayModify(std::unique_ptr<Object> up);

```






### 推荐的方法 Use owner<> form GSL

```c++
gsl::owner<Object*>
foo(args)
{
    gsl::owner<Object*> ret = new Object(args);
    return ret
}
bar()
{
    auto p1 = foo(args);
    ...
    auto p2 = DoSomethingMayModify(p1);
    delete p1;
    delete p2;
}
gsl::ower<Object*>
DoSomethingMayModify(gsl::owner<Object*> owner); // 告诉其他函数这个地方的所有权
```







## I.12：将不能为空的指针声明为not_null

>   **Reason**
>
>To help avoid dereferencing `nullptr` errors. To improve performance by avoiding redundant checks for `nullptr`.



我们经常会写这样的代码

```c++
if(some_ptr != nullptr)
    do_something();
```

当我们忘记作这个检查的时候,传入的这个null可能导致程序崩溃,要找到这种空指针错误是非常难的

我们得一遍一遍的看看调用信息

如果我们将一个指针直接声明为不是null 是不是要更简单一些?

### 使用GSL::not_null

```c++

void foo()
{
    gsl::not_null<int*> pint = new int;
    pint = nullptr; // you can not assign nullptr to a not_null ptr
    pint = new int; // OK new int is not a nullptr
}
```



如果你将not_null的指针赋值为nullptr, 是无法通过编译器的

这样做的好处:

​					1.避免nullptr错误,不用检查一个不是nullptr的指针是否是nullptr

​					2.更好的可读性



## ES.46:避免溢出(隐式,截断)
>  ES.46: Avoid lossy (narrowing, truncating) arithmetic conversions
>
>  **Reason** A narrowing conversion destroys information, often unexpectedly so. 



###　use gsl::narrow<>()` 和 `gsl::narrow_cast<>()

这两个函数和 `static_cast<>()` 一样，只不过 `gsl::narrow<>()` 会检查溢出。

`narrow<>` 在数据丢失时会throw一个错误

```c++
#define GSL_THROW_ON_CONTRACT_VIOLATION
#include <gsl/gsl>
#include <iostream>

int foo(void)
{
    uint64_t val = 0xFFFFFFFFFFFFFFFF;
    try {
        gsl::narrow<uint32_t>(val);
    } catch(...) {
        std::cout << "narrow failed\n";
    }
}
// narrow failed
```



使用narrow 来判断是否溢出





参考链接:

[C++ Core Guidelines](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)

[在vs2019中开启检查](https://github.com/microsoft/GSL)
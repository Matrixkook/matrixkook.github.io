---
layout:     post
title:      AAA Style Or limit it
subtitle:   auto auto auto 
date:       2020-06-08
author:     Matrixtang
header-img:  img/2020-06-08-AAA.jpeg
catalog: true
tags:
    - c++ lang
---



# Special Edition: AAA Style Or limit it

>Toward correct-by-default, efficient-by-default, and pitfall-free-by-default variable declarations, using “AAA style”… where “triple-A” is both a mnemonic and an evaluation of its value.



自从C++11,引进auto之后.auto便成为被很多人的使用的,能大幅度减少的代码的特性.但是不完整的类型推导,导致许多错误 例如

```c++
auto test_for_proma() ->std::uint16_t
 {
    std::uint16_t b; 
    const auto a = b + 1; // auto => int
    return b + 1;
 }
void test_auto_derivation_error()
{
    auto c = test_for_proma() + static_cast<std::uint16_t>(1);// auto => int
    auto d = test_for_proma();//auto => uint_t16
    auto e = static_cast<std::uint16_t>(1);// auto => uint_t16
}
// on CXX=c++20
```

按照语义应该推到出uint_t16的auto , 却将两个uint_t16的推导为int

Herb Sutter在[AAA](https://herbsutter.com/2013/06/13/gotw-94-special-edition-aaa-style-almost-always-auto/)这篇文章中提议到的,有这样一种编程风格AAA(Almost Always Auto)

```c++
template<class Container>
void some_function( Container& c ) {
   for(auto v : Container)
       do_something();
}
```

简化了 从容器中取值的操作 支持range

```c++
for(auto a : some_container)
    do_something(a);
```



## Use auto

```c++
// Stack Allocation
auto e = employee{empid};

// Heap Allocation
auto w = make_unique<widget>();

// Literal Suffixes (User-defined suffixes coming in C++14)
auto x = 42; // int 
auto x = 42.0f; // float
auto x = 42ul; // unsigned long
auto x = "42"s; // C++14, std::string
auto x = 42ns; // C++14, std::nano_seconds not in c++20

// Function declarations and named lambdas
auto f(double) -> int { ... }
auto f = [=] (double) -> int { ... };

// Alias declarations
using dict = set<string>;

template <class T>
using vec = vector<T, myalloc>;
shifen
// Not applicable for:
auto arr = std::array<int, 1000>{}; // Expensive to move
auto & arr = std::array<int,1000>{}; // auto & is better
auto lock = std::lock_guard<std::mutex>{m} // Not movable


```

在我们明确知道 一个变量的类型的时候,用auto来推导是十分简单和方便的

配合decltype 或者单纯的auto 可以实现

```c++
class A 
{   
    int i;
    public:
         A(int ii ):i(ii){}
        A& operator*(const A& lhs)
        {
            auto res = new A(this->i * lhs.i);
            return *res;
        }
};

template<class T, class U>
auto mul(T x, U y)
{
    return x*y;
}


void test_auto_in_template_function()
{
    auto a = mul<int , int >(1,2);//  auto=> int
    auto b = mul<double, int>(1.0,2);//auto => intA
    auto c = mul<std::uint16_t, int>(1,2);//auto => int
    auto d = mul<A,A>(A(1),A(2));// auto=> A

}

```



或者这样

```c++
template<class T, class U>
auto mul(T x, U y) -> decltype(x * y) 
{
 
   return x*y;
}
//显式将mul的类型定义为 x * y
```



既然auto有这么多好处 , AAA 想必是一个应该贯彻下去的范式



## with limit

-----



### auto会损失可读性  , auto is weakest concept



> **[T.12: Prefer concept names over `auto` for local variables](http://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#t12-prefer-concept-names-over-auto-for-local-variables)**
>
> ##### Reason
>
> `auto` is the weakest concept. Concept names convey more meaning than just `auto`.

(C++ core guidelines 里auto部分居然是todo的 emmmm)

在我们用临时变量的时候最好用完整的概念(concept)而不是auto去命名(在TS中)

```c++
vector<string> v{ "abc", "xyz" };
auto& x = v.front();     // bad
String& s = v.front();   // good (String is a GSL concept)
```





auto也会让代码变得十分不可读

```c++
auto
make_complex_tuple()
{
    auto vec = {1,3,4};
    auto fun1 = [](decltype(vec) vec) ->auto { return decltype(vec)(vec);};
    return std::make_tuple(fun1(vec),"hello","world"); 
}

void test_auto_in_unreadable()
{
    auto tuple = make_complex_tuple();
    for(auto i : std::get<0>(tuple))
        std::cout << i << " ";
    std::cout << std::get<1>(tuple) << " " << std::get<2>(tuple);

}
```



### Dark edge



1. init_list

```c++
auto x = long long{ 42 };            // error
auto x = int64_t{ 42 };              // ok, better 
auto x = 42LL;                       // ok, better 

auto y = class X{1,2,3};             // error
auto y = X{1,2,3};                   // ok

```

在推导初始化列表时,存在这两种情况无法正确推导



2.proxy class

vector<bool>是一个奇葩的存在，它的[]返回的不是bool，是一个表示单独bool引用的proxy class，于是你得到是这个引用（而你平常使用bool没事，是因为完成了一个隐式转换），而你用auto的话，想要得到意想中的类型，需要使用static_cast<bool>，再给auto。而这也不是特例，其适用于含有proxy class的class。

```c++
void test_vector_bool()
{
    std::vector<bool> v;
    v.push_back(true);
    auto var = v[0];
    std::cout << typeid(var).name() << std::endl; //St14_Bit_reference
}
```





## End

>We already ignore explicit and exact types much of the time,  including with temporary objects, virtual functions, templates, and  more. This is a feature, not a bug, because it makes our code less  tightly coupled, and more generic, flexible, reusable, and future-proof.
>
>Declaring variables using auto,  whether or not we want to commit to a type, offers advantages for  correctness, performance, maintainability, and robustness, as well as  typing convenience. Furthermore, it is an example of how the C++ world  is moving to a left-to-right declaration style everywhere, of the form
>
>**category** name = **type** and/or initializer ;
>
>where “category” can be auto or using, and we can get not only correctness and performance but also  consistency benefits by using the style to consistently declare local  variables (including using literals and user-defined literals), function declarations, named lambdas, aliases, template aliases, and more.

我们已经很多时候忽略了显式和精确类型，包括临时对象，虚函数，模板等等。 这是一个功能，而不是错误，因为它使我们的代码紧密耦合，并且更加通用，灵活，可重用且面向未来。

使用auto声明变量，无论我们是否要提交类型，都具有正确性，性能，可维护性和鲁棒性以及键入方便的优点。 此外，它是一个示例，说明C ++世界如何在各处都从左向右声明样式，形式如下：

**category** name = **type** and/or initializer ;

类别名称=类型和/或初始化器；

在这里“类别”可以是自动的或可以使用的，通过使用样式一致地声明局部变量（包括使用文字和用户定义的文字），函数声明命名的lambda，别名，我们不仅可以获得正确性和性能，而且还可以获得代码命名一致性好处,模板别名等。

但是使用过程中, 也要关心auto的一些奇怪的标准



```c++
// Classic C++ workaround            // Modern C++ style

typedef set<string> dict;            using dict = set<string>;

template<class T> struct myvec {     template<class T>
  typedef vector<T,myalloc> type;    using myvec = vector<T,myalloc>;
};
```



参考链接

[solution_AAA](https://herbsutter.com/2013/08/12/gotw-94-solution-aaa-style-almost-always-auto/)

[auto 关键字 -- 知乎](https://www.zhihu.com/question/35517805/answer/63304992)




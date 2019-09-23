# boost.Spirit入门(2)

之前有用boost写了一个简单的四则运算解析器 

现在开始看官方文档

> The magic of expression templates

[主要参考](https://www.boost.org/doc/libs/1_71_0/libs/spirit/doc/html/spirit/introduction.html)

> 头秃 只有英文

----



## 基本介绍

![img](img\spiritstructure.png)

>The [figure](https://www.boost.org/doc/libs/1_71_0/libs/spirit/doc/html/spirit/introduction.html#spirit.spiritstructure) below depicts the
>overall structure of the Boost Spirit library. The library consists of 4 majorparts



将之前的的代码放在了class部分 ,主要分为4个部分



![spiritkarmaflow](img\spiritkarmaflow.png)

主要利用Qi库写解析器



## Include

Spirit 不含任何动态解析库 , 只有头文件

它的库函数位于

```bash
$ BOOST_ROOT/boost/spirit
```

而

```bash
$ BOOST_ROOT/boost/spirit/home
```

才是真正的home目录 

包含了以下

```noew
[classic]   [karma]     [lex]
[phoenix]   [qi]        [support]
```

每个模块都以



```c++
<boost/spirit/home/xxx.hpp>
```



## Abstract

[![Abstracts](SSP\img\Abstracts.png)

### Syntax Diagram

> [Syntax Diagram 原文档](https://www.boost.org/doc/libs/1_71_0/libs/spirit/doc/html/spirit/abstracts/syntax_diagram.html)



(不太好翻译 , 对照看英语原版吧,是对phraser(解析器)的一个介绍

用来描述之后的解析器



### Parsing Expression Grammars

[Parsing Expression Grammars原文档](https://www.boost.org/doc/libs/1_71_0/libs/spirit/doc/html/spirit/abstracts/parsing_expression_grammar.html)

PEG 是 EBNF 的拓展语法 

不同之处在前面有写 现在来补充几个地方



#### LOOPS

PEG用**regular-expression Kleene star**(**重复 操作符**)

和+ 来构成循环



```PEG
a*
a+
```



> Unlike EBNF, PEGs have greedy loops. It will match as much as it can , until its subject fails to match without regard to what follows

最大的不同在于 PEG中的loop是更加贪婪的 它会一直匹配到无法匹配



### Difference

>In some cases, you may want to restrict a certain expression. The difference operator allows you to restrict this set

如果你想约束一个表达式 , 可以用 **-** 

```PEG
a - b
```

应该被理解为匹配a而不是b



## Attribute

### Attribute of Primitive Components

[原文档](https://www.boost.org/doc/libs/1_71_0/libs/spirit/doc/html/spirit/abstracts/attributes/primitive_attributes.html)

parsers 总是返回一个给定字符串成功解析后的**synthesized
attribute**(综合属性)对象

对于数字型解析器例如 int_或者是 double_ 就会返回 int | double

你可以用任何的C++类型变量去接受这个返回值

```c++
#include <iostream>
#include <boost/spirit/include/qi.hpp>

using namespace std;
using namespace boost::spirit::qi;

int main()
{
	int value = 0;
	int char_value = '1';
	std::string str("123");
	
	std::string::iterator strbegin = str.begin();
	parse(strbegin, str.end(), boost::spirit::qi::int_, value);   // value == 123
	cout << str << endl;

	parse(strbegin, str.end(), ascii::char_, char_value); // char_value == 
	cout << char_value << endl;
	getchar();
	return 0;
}


```



### Attributes of Compound Components

```txt
Qi 			a: A, b:B --> (a >> b): tuple<A,B>
```

这个用法和std中的==pair<A,B>类似 但是这里的a,b是两个parser





### Attributes of Rules and Grammers

解析器可以被构建 , 由小的解析可以生成复杂的解析器

在申明rule类时应该指明其继承和返回的**synthesized
attribute**



```c++
qi::rule<Iterator, double> r
```




---
layout:     post
title:      Dragon Slaying Step Zero
subtitle:   如何开始「 屠龙 」?
date:       2021-04-14
author:     Matrixtang
header-img: img/2021-04-14-Dragon_Slaying_Step_Zero.jpg
catalog: true
tags:
    - Compilation principle
    
---


# Intr	
		众所周知，编译技术被称为「 屠龙 」技 。该领域中最著名的书籍也被称为  龙书  。这些技术一直被冠以 黑魔法、黑箱的称号，学习它们似乎是一个艰难困苦的道路。编译器以其精巧，严谨的代码和原理，向我们展示了 自举  的光景。
但是只要你深入阅读代码，它 恐怖  的 dark edges  又会在你耳畔轻声低语:" 放弃吧，这不是凡人应该涉足的领域。“  
> 要知道，你现在是单骑入风尘，飘飘无所依，迎面扑来的是成千上万如蝗虫如夏蚊的程序代码。

实际上，它和普通的程序并无区别，只是解决的问题不同罢了。你可以无视这些技术，把它当作黑箱。抑或深入了解，构建一条编译之路。了解这些技术有助于你理解自己的代码，直至汇编。知道编译器到底为你做了些什么优化，能做什么，不能做什么。
		如果你希望优化代码，了解它，你会获得优化的技术；如果你希望了解代码生成，了解它，你将会拆解这个黑盒；如果你希望让代码更加健壮、安全，了解它，在编译期减少问题。
		但是 「 屠龙技 」并不能为你带来优越感，学习它，不是为了炫耀，请不要自恃甚高。了解的人少，仅仅是因为不需要这么多人而已。
如何开启你的 「 屠龙 」之旅 ？ 


# Start-Book
Essentials of Compilation 
> 评价： 很好的入门书籍，用 scheme 实现简单的语言。

Engineering a Compiler [COMP 412: Introduction to Compiler Construction (rice.edu)](https://www.clear.rice.edu/comp412/)
> 评价：后端优化知识，以及编译器的实现。PPT 讲的十分清楚。

Parsing Techniques
> 评价： 前端知识，如何手写Parser。

Compiling with Continuations （todo）（没看

# Reference-Book
龙书 Principles of Compiler Design
> 评价： 挺难的，建议已经有背景知识之后当作参考书看。


## 垃圾收集 
> 如果你想搞虚拟机；简单的垃圾收集算法
[https://book.douban.com/subject/26821357/](https://book.douban.com/subject/26821357/)
垃圾回收的算法与实现

很好的垃圾收集入门文章。

# Extend-Thing

## Program Analysis

1. [_Lecture Notes on Static Analysis_](https://lara.epfl.ch/w/_media/sav08:schwartzbach.pdf) --Michael I. Schwartzbach
2. [_Static Program Analysis_](https://cs.au.dk/~amoeller/spa/) --[Anders Møller](https://cs.au.dk/~amoeller/), Michael I. Schwartzbach
3. [静态程序分析入门教程](https://github.com/RangerNJU/Static-Program-Analysis-Book)


# Resource
[https://github.com/shining1984/PL-Compiler-Resource](https://github.com/shining1984/PL-Compiler-Resource)
你需要的技术集散地


# Work on Action
[ ] 随便写点解析器

[https://bodil.lol/parser-combinators/](https://bodil.lol/parser-combinators/) 或者 写一个 json 解析库
任意一本前端的书看的时候就可以实践了。我会写一些入门资料，使用 rust。
[ ] 看完了SICP？ 写一个 解释器吧！

 		没啥说的，已经有很多 lisp 解释器了。
[ ] LLVM ？用 编译器框架 干点事情吧！

学习 llvm 写一个玩具语言的编译器。
依照你的方向，干一些你想做的事情。
[ ] 实现你自己的编译器

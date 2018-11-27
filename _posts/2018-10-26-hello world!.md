---
layout:     post
title:      Hello world！
subtitle:   CNSS招新题目
date:       2018-10-26
author:     matrixkook
header-img: img/post-bg-keybord.jpg
catalog: true
tags:
    - c language
---

**众所周知**
```
#include<stdio.h>
int main(void)
{
printf("hello world!\n");
    return 0;
}

```
(当然因为我就知道C语言的)是广大程序员的 ~~第一个~~几乎是第一个程序。
但是我们真的完全了解他了吗。
**-_-|| 并没有**

近期水的一个题目的确让我，emmmm 反思了一下自己的学习成果。
这就是CNSS的~~招新~~(反正我觉得不是的)题目。
 **花式打出hello world!**

-----
> let it begin.

## 不用' ; '打出
这个比较简单，直接用if语句
```
int main()
{
	if (puts("Helloworld!")){}
}
```
就可以无; 实现了
> 效果
![image](/img/HW01.png)

-----

## 不用" "打出
第二个可以利用ASCII码，利用数组的方式输出。
```
#include <stdio.h>
int main()
{
	char Arr[] = {72,101,108,108,111,87,111,114,108,100,33};
	puts(Arr);
}
```
![image](/img/HW02.png)

就可以了 挺简单的对吧|･ω･｀)。
## 不用#输出
这就触及到我的知识盲区了.emmmm意思我要手写printf咯.
好吧,反正我也不会。~~反正都是面向谷歌做题~~

```
extern int printf (const char* format, ...);
int main()
{
  printf("HelloWorld!\n");
  return 0;
}

```
运行结果 
![image](/img/HW04.png)


总算是可以了，不过这里也有一个问题没有解决:那一个完整的printf函数怎么实现呢，(变参的函数)怎么写呢？
水平有限 先挖个坑吧 ~~我不会挖坑不填的~~

----
## 不用任何括号实现
这里完全超过了我的知识水平，先立一个flag吧。在接下来的几个月搞定这个问题

edited by markdown.

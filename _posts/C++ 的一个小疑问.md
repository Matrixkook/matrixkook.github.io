# C++ 的一个小疑问

[来源](https://github.com/Mooophy/Cpp-Primer/issues/390)

出自**c++ primer**  ex7.35

```c++
typedef string Type;
Type initVal(); 
class Exercise {
public:
    typedef double Type;
    Type setVal(Type);
    Type initVal(); 
private:
    int val;
};
Type Exercise::setVal(Type parm) { 
    val = parm + initVal();     
    return val;
}
```

> 解释下面代码的含义，说明其中的 Type 和 initVal 分别使用了哪个定义。如果代码存在错误，尝试修改它。



中文书上说

> 然而在类中，如果成员使用了外层作用域中的某个名字，而该名字代表一种类型，则类不能在之后重新定义该名字。



英文版的书上说

> `Exercise::initVal()` should be defined.

p285上说

> Definitions of type names usually should appear at the beginning of a class.
>
> 类型别名的定义一般出现在类定义的开头

在类中，如果一个成员使用了类外作用域的类型别名，在该成员后，类不能再次定义相同名字的类型别名

但是你可通过将**Exercise**类里定义的type来申明函数

编译器过程

```c++
Type /编译器看到这里，当然觉得Type是string/
Exercise:: /然后当前scope进入了Exercise/
setVal(Type /因此这个就是double/ parm)
/遇到大括号，发现你声明和定义的函数签名都不一样，报错/
{
val = parm + initVal();
return val;
}

所以要写
Exercise::Type Exercies::setVal(Type parm)
    by vczh
```


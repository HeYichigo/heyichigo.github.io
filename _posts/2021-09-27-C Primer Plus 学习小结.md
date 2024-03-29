---
layout: post
title: C Primer Plus 学习小结
subtitle: 看C语言经典
tags: [语言学习]
comments: true
last-updated: 2021-09-29 13:56
---

# C Primer Plus 学习小结

最近买了这本经典，想要复习一下C的内容，顺便看看C加入了新的标准后有何变化，看的过程中发现了一些有意思的事情。

## 关于C的可移植性的讨论

在我的认知里C程序的运行对环境的要求很高，比起 *Java* 的 *write once run anywhere*，C语言基本不存在可移植性一说，然而这是我的误解。

书中将汇编语言和C进行比较，因为机器硬件的不同，以汇编语言编写的程序要想转移到另外一台机器上运行，有极大的可能会重新编码，因为机器之间的汇编指令可能会不同。但是用C语言来编写就不成问题，高级语言通过**编译器**来编译成机器语言，将编译后的程序在机器上运行，能够期望得到一致的结果。显然，C语言的可移植性依赖于编译器的实现，在这里就发生了有趣的事情。

编译器的实现依赖于标准，像 *Java* 一样存在[语言标准](https://docs.oracle.com/javase/8/)。但是C存在一些暧昧的标准，这些标准并未规定编译器的行为，这样编译器很尴尬，这些行为代码中会出现，但是我们不知道要将这种行为导向什么样的后果，编译器只能自己琢磨出一些办法，这也导致了同样的代码在不同的编译器下的行为不一致。

是不是已经感觉到C标准的混乱了呢？但是我们研究的C语言并不是依赖于某个编译器实现下的情况，如果只讨论这个部分（不依赖编译器的部分），那我们可以说***C语言编写的程序是可移植的***。

影响C程序可移植性的不仅仅是编译器，操作系统也是一个方面。比如经常用来举例的*Socket*编程，其在*windows*和*Linux*上的实现是不一致的，其函数声明也是不一致的。如果我们编写的程序不依赖于操作系统的特定实现，那我们也可以说***C语言编写的程序是可移植的***。

## 关于宏

我对于宏的理解一直有问题，一直都是从形式上理解。比如 `#include` ，我之前的理解就是加入了文件的引用，和 `import` 差不多；比如 `#define`，我之前的理解是，相当于定义了一个变量；如果定义的是含参数的宏，那么我就当成是一个函数去理解。

上面的说法我认为完全是错误的，千万不要被我误导，以下是我现在认为正确的理解：

宏，理解成 **查找 / 替换 / 拷贝 / 粘贴** 的功能更好一些。比如 `#include  <stdio>` ，就是将 `stdio.h` 文件中的内容拷贝，然后粘贴到有标记这个宏的文件当中；比如 `#define Length 5`，其实是将代码中的 **Length** 替换成 **5**。含参数的宏，`#define MAX(a,b) a > b ? a : b`，在使用的时候我们会像下面这样：

```c
#define MAX(a,b) a > b ? a : b
int main()
{
    int num1 = 5, num2 = 6;
    MAX(num1,num2);
}
```

很容易理解成一个函数吧，num1和num2是实参，a和b是形参。其实不然，这里`MAX(num1,num2);`做的，其实是将文字a与b和文字num1与num2进行替换，然后把替换后的字符串放到**MAX**标记的位置：

```c
#define MAX(a,b) a > b ? a : b
int main()
{
    int num1 = 5, num2 = 6;
    MAX(num1,num2);//替换后为 num1 > num2 ? num1 : num2;
}
```

## 关于声明

我认为C语言最复杂的就是声明。声明可以帮助程序员来理解标识符的用途，C语言的声明属实是复杂，大家觉得指针难以理解可能也是因为声明的复杂性吧。

### const 声明

```c++
const int a = 1;
int const b = 2;
b = 3; // 错误的操作
```

对于 `const` 我们知道它声明了一个常量，即不会被改变的量，对常量的改变会造成程序的错误。上述代码中的声明都是正确的，对于一个普通的变量来说，`const`的位置对于声明的理解没有任何影响。

我们来看另外的例子：

```c++
#include <iostream>

using namespace std;

int main(int argc, char const *argv[])
{
    int a = 1, b = 2;
    const int *p1 = &a;
    cout << "p1:a  " << *p1 << "|" << p1 << endl;
    p1 = &b;
    cout << "p1:b  " << *p1 << "|" << p1 << endl;
    return 0;
}
/* output:
p1:a  1|0x7fffffffdd58
p1:b  2|0x7fffffffdd5c
*/
```

为什么程序可以正常输出？我们定义的常量究竟是哪个？

```c++
int *p1 = &a;
```

对于上述声明，我们说定义了一个指针变量**p1**，其值为或者说指向**a**所在的地址。我们可以通过**解引用**来修改**a**的内容。

```c++
const int *p1 = &a;
```

对于上述声明，我们说定义了一个指针常量**p1**，对于指针常量的定义，我们可能会理解成**p1**所指向的地址不可改变，但这显然是错误的，因为在上述程序中我们确确实实的修改了**p1**的指向，那我们该如何理解？

我观察到的现象如下：

```c++
const int * p1 = &a; // p1 的解引用不可改变
int * const p1 = &a; // p1 的引用（指向或值）不可改变
const int * const p1 = &a; // p1 的解引用和引用都不可改变
```

仅仅是记住了这样的现象，我还没有办法得出一个统一的方法来判断`const`的限定范围，暂时强行记忆吧。不过网友们倒是进行了总结：

> 如果*const*位于`*`的左侧，则const就是用来修饰指针所指向的变量，即指针指向为常量；
> 如果const位于`*`的右侧，*const*就是修饰指针本身，即指针本身是常量。

### 函数式声明

这里的函数式声明指的是函数指针。没错，C语言中存在指向函数的指针，我当成函数式编程来进行理解。不过这个函数式编程没有`javascript`用得自在，主要体现在函数的声明上。

如果要使用函数指针，应该是这样的声明：*返回值类型*  (**指针名*) (*参数列表*)

```c++
int (*compar)(const void *, const void *)
```

指针名外部的括号必不可少，如果没有括号，该声明的返回值类型就变为了 `int *`，这是`*`的优先级顺序导致的，在书中有这样的介绍：

```c++
char * fump(int);          // 返回字符指针的函数
char (* frump)(int);       // 指向函数的指针，该函数的返回类型为char
char (* flump[3])(int);    // 内含3个指针的数组，每个指针都指向返回类型为char的函数
```

| 符号 |     含义     |
| :--: | :----------: |
| `*`  | 表示一个指针 |
| `()` | 表示一个函数 |
| `[]` | 表示一个数组 |

`[]`和`()`的优先级最高，`*`的优先级要低，理解了这一点，上面的声明就应该能看得懂了，至少函数指针的声明和其它类型的声明是可以区别开了。

我们当然也可以尝试理解更加复杂的声明：

```c++
int board[8][8];       // 声明一个内含int数组的数组
int ** ptr;            // 声明一个指向指针的指针，被指向的指针指向int
int * risks[10];       // 声明一个内含10个元素的数组，每个元素都是一个指向int的指针
int (* rusks)[10];     // 声明一个指向数组的指针，该数组内含10个int类型的值
int * oof[3][4];       // 声明一个3×4 的二维数组，每个元素都是指向int的指针
int (* uuf)[3][4];     // 声明一个指向3×4二维数组的指针，该数组中内含int类型值
int (* uof[3])[4];     // 声明一个内含3个指针元素的数组，其中每个指针都指向一个内含4个int类型元素的数组
```

扯远了，我们的目的是来讲讲指向函数的指针。

`int (*compar)(const void *, const void *)`其实是`qsort()`函数中的第三个参数，两个对象进行比较时会调用这个函数指针指向的函数，这个函数指针在调用的时候和普通的函数一样，可以看作是将同样的函数换了一个名字。

举个书中的例子：

```c++
void ToUpper(char *);    // 把字符串中的字符转换成大写字符
void (*pf)(char *);      // 定义了一个函数指针
pf = ToUpper;            // 将函数指针指向函数（其实函数名就代表函数的地址）
char mis[] = "Nina Metier";
pf(mis);                 // 把ToUpper 作用于mis
```

可以看出函数指针的调用和函数相同，你也可以采用这样的方式进行调用`(*pf)(mis)`。

### extern声明

`extern`表示某个声明或定义来自于外部文件并可以直接使用。

C中的声明往往向两个对象传递信息，一是用户即开发者，二是编译器。比如`extern`，告诉开发者，这个东西已经在其它文件中声明了，直接用就可以。同样也是告诉编译器，这个是外部的量，你要帮我把它链接过来。

## 总结

C语言姑且总结到这里，后续在学习C++以及UNIX的内容时肯定也会带上C的内容，等到再有收获再来记录。

# ch04 function

[toc]

## concept &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;

函数 (***function***) 是执行任务的语句块。

**函数的作用**：

- 提高代码的可重用性并减少冗余
- 代码模块化
- 代码易读性
- 使代码模块化

## 函数的分类

C语言中有两种类型的函数：

- 标准库函数：C中的内置函数，在头文件中定义
    - `#include &lt;stdio.h&gt;`
- 用户自定义函数：用户自定义的函数
    - `#include &#34;stdio.h&#34;`

## 函数三部曲

C语言中函数分为三个方面，声明(***declaration***)，定义(***defining***)，调用(***calling***)

### 声明

声明是让编译器知道函数的名称、参数信息、参数的返回值的类型。

```c
(type) function_name({type args...});
```

**隐式声明**(***implicit***) ：当在main之后定义的函数而未声明，默认编译器会做隐式声明。

&gt; ISO/IEC 9899:1990 中 关于函数声明的部分：
&gt;
&gt; 函数在调用前必须有一个可用的声明，如果没有被声明，则该函数默认被隐式声明，该隐式声明没有参数，返回值为int &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;

### 定义

C中函数定义的语法如下

```c
return_type function_name(arg1, arg2, ... argn)
{
	function body // 函数中要处理任务的逻辑
}
```

- ***return_type***：函数返回值的数据类型
- ***function_name***：函数名
- ***arg1, arg2, ...argn***：参数列表（可选），定义传递给函数的数据类型、顺序和参数的数量。
- ***function body***：调用函数时任务处理和执行的语句

### 调用

调用是指要由编译器执行的函数，可以在任何部分调用

## 虚函数void

如果函数没有返回值，则使用关键字 ***void***，主要用于两个方面：

- 打印具体信息供用户阅读的函数
- 引用参数，函数通常不是用于返回一个内容，而是修改引用参数的，无需返回值

***void*** 关键字使用注意：

- void仅用于限定函数返回值，函数参数，不可以修饰变量，因为无法对无类型的变量分配指针

- void修饰指针时表示泛指针，可以无需强制转换为其他类型的指针

    ```c
    void *ptr=NULL;
    char * a=&#34;1234&#34;;
    printf(&#34;a %s\n&#34;, a);
    ptr = a;
    printf(&#34;%s\n&#34;, (char*)ptr);
    ```

## 宏函数

宏函数是指带有参数的宏(***Macro-Arguments***)，具有类似函数的功能，例如下列时一个获取最小值的宏函数

```c
#define min(X, Y)  ((X) &lt; (Y) ? (X) : (Y))
    char a=&#39;a&#39;;
    char b=&#39;b&#39;;
    char x = min(a, b);         // →  x = ((a) &lt; (b) ? (a) : (b));
    char y = min(1, 2);         // →  y = ((1) &lt; (2) ? (1) : (2));
    printf(&#34;x is %c, y is %c\n&#34;, x, y);
```

### 宏函数危险部分

在上面例子中存在一些不安全部分

- 错误嵌套：

- **括号优先级**：`#define ceil_div(x, y) x &#43; y` 因为宏函数带有的括号是围绕这个宏函数的，会存在运算符优先级问题，如果用上述宏函数进行输出得到的结果为：`ceil_div(2, 3) * 10 = 32` ，因为括号不是表达式的括号
  
    - 解决方法：每一个宏函数的参数需要用括号括起来 `#define ceil_div(x, y) (x &#43; y)`
    
- **吞分号**：`#define NEW_MACRO()  ({ int x = 1; int y = 2; x&#43;y; })` 上述宏函数在GCC预处理步骤替换时，通常调用宏函数的部分会加分号 `NEW_MACRO();` ，例如下列代码中
  
    ```c
    if(1) 
        SKIP_SPACES();
    else 
        ...
    ```
    
    被替换后为
    
    ```c
    if(1) 
    { 
        int x = 1; int y = 2; x&#43;y; 
    }; // 《这里的分号会跳过else
    else 
        ...
    ```
    
    通常情况下使用do...while替换
    
    ```c
    #define NEW_MACRO() do { int x = 1; int y = 2; x&#43;y; } while (0)
    
    if(1) 
    	do{
            int x = 1; int y = 2; x&#43;y;
        }
    	while(0);
    else 
        ...
    ```
    
- **重复替换的副作用**：`#define min(X, Y)  ((X) &lt; (Y) ? (X) : (Y))` 这个宏函数在gcc预处理中如果调用时是 `next = min (x &#43; y, foo (z));` 将被重复替换，如下列代码所示：
	
	```c
	next = ((x &#43; y) &lt; (foo (z)) ? (x &#43; y) : (foo (z)));  // 参数Y被替换为两个foo
	```
	
	这代表foo被执行两次，这种显然不安全，推荐使用typeof
	
	```c
	#define min(X, Y)                \
	({ typeof (X) x_ = (X);          \
	   typeof (Y) y_ = (Y);          \
	   (x_ &lt; y_) ? x_ : y_; })
	```
	
- **直接自引用**：是指定义的宏引用自己，例如 `#define foo (4 &#43; foo)`，为了方式无限扩展为 `(4&#43;(4&#43;foo))`, `(4&#43;(4&#43;(4&#43;foo)))` ... 直到内存耗尽，这种情况编译器将不允许 `each undeclared identifier is reported only once for each function it appears in`

- **间接自引用**：指a引用b，b引用a，例如下列代码，是不被允许的

  ```c
  #define x (4 &#43; y) 
  #define y (2 * x)
  ```
  
- 参数的换行符不被允许


## 函数的退出

- exit() 是一个终止当前进程的系统调用（无论在代码哪里调用）；非C语言内置功能
- return：向调用函数提供退出状态并将控制权返回给调用函数，C语言内置功能

## 多文件编程 &lt;sup&gt;&lt;a href=&#34;#3&#34;&gt;[3]&lt;/a&gt;&lt;/sup&gt;

多文件程序(***multi-file***) 是指多个含有不同功能的代码文件（ .c 文件模块），编译到一起，生成一个二进制文件。

通常包含三部分：

- 编译：通过编译器编译多个文件程序
- 函数原型（声明）：告知编译器如何使用，表现为：
    - 函数在一个文件中定义，在另一个文件中调用
    - 想对文件中的函数重新排序
    - 函数相互调用，递归

- 头文件：使多个文件中的函数可以访问定义和声明，通常情况下包含：
    - 全局变量和全局常量
    - 类，结构体，联合体，枚举等
    - 创建类型名称的 typedef 语句
    - 函数声明
    - 包含其他文件的语句，如math.h


**防止头文件重复包含**

windows

```c
#pragma once
```

linux

```c
#ifndef __HEAD_H__
#define __HEAD_H__

.... head file body

#endif
```



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [c function](https://www.programiz.com/c-programming/c-functions)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [Are prototypes required for all functions in C89, C90 or C99?](https://stackoverflow.com/questions/434763/are-prototypes-required-for-all-functions-in-c89-c90-or-c99)
&gt;
&gt; &lt;sup id=&#34;3&#34;&gt;[3]&lt;/sup&gt; [multi-file](https://www.cs.hmc.edu/~geoff/classes/hmc.cs070.200401/notes/multi-file.html)
&gt;
&gt; &lt;sup id=&#34;4&#34;&gt;[4]&lt;/sup&gt; [Macros](https://gcc.gnu.org/onlinedocs/cpp/Macros.html#Macros)

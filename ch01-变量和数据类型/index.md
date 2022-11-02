# ch01 变量和数据类型


[toc]

## C语言关键字 &lt;sup id=&#34;keywords&#34;&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;

==C语言有32个关键字==

- ***auto***：定义自动变量，主要是声明变量的生存周期
- ***break***, ***continue*** : break 语句在遇到最内层循环时立即终止。还用于终止 switch 语句。
- ***case***, ***switch***, ***default***：使用 switch 和 case 语句声明一个switch分支
- ***char***：用于声明character 类型的变量
- ***const***：声明常量
- ***do...while***：
- ***double***： double-precision 浮点数变量类型
- ***float***：single-precision 浮点数的变量类型
- ***if***, ***else***：声明if/else 条件判断
- ***enum***：用于声明枚举类型
- ***extern***：关键字声明变量或函数在其声明的文件之外具有外部链接。
- ***for***：C 语言的三种循环之一，for循环
- ***goto***： 用于将程序的控制权转移到指定的标签
- ***int***：声明 integer 类型的变量
- ***short***, ***long***, ***signed***, ***unsigned***：是类型修饰符，它们改变基本数据类型的含义以产生新类型。
  - ***short int***： -32768 to 32767
  - ***long int***：  -2147483648 to 214743648
  - ***signed int***： -32768 to 32767
  - ***unsigned int***： 0 to 65535
- ***return***： 终止函数并返回值
- ***sizeof***：评估变量或常量的大小
- ***register***：创建比普通变量快得多的寄存器变量。
- ***static***：创建一个静态变量。静态变量的值持续到程序结束。
- ***struct***：用于声明结构体。结构体可以包含不同类型的变量。
- ***typedef***：用于将类型与标识符显式关联。
- ***union***：用于将不同类型的变量分组在一个名称下。
- ***void***：没有任何意义，函数修饰为没有返回值，参数修饰为没有参数
- ***volatile***：提醒编译器它后面所定义的变量随时都有可能改变

## C语言控制语句

==C语言有9种控制语句== (***control statements***)

- If..else
- for
- while
- do..while
- continue
- break
- switch
- goto
- return

## C语言运算符 &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;

==C语言有45种运算符== (***operator***)

- 算数运算符 (***Arithmetic Operators***) ：`&#43;`, `-`，`*`，`/`，`%`
- 赋值运算符 (***Assignment Operators***) ：`=`，`&#43;=`，`-=`，`*=`，`/=`，`%=`
- 关系运算符 (***Relational Operators***)：`==`，`&gt;` ，`&lt;`，`!=` ，`&gt;=`，`&lt;=`
- 逻辑运算符 (***Logical Operators***)：`&amp;&amp;`，`||`，`!`
- 位运算符 (***Bitwise Operators***)：`&amp;`，`|`，`^`，`~`，`&lt;&lt;`，`&gt;&gt;`
- 逗号运算符 (***Comma Operator***)：链接相关表达式 ，`int a, c = 5, d;`
- sizeof运算符(***sizeof operator***)：一元运算符，它返回数据的大小（常量、变量、数组、结构）
- 杂项运算符（）：`&amp;` 取址，`*` 取指针，`?:` 二元条件表达式

## GCC编译四部曲 &lt;sup&gt;&lt;a href=&#34;#3&#34;&gt;[3]&lt;/a&gt;&lt;/sup&gt;

- 预处理 (***Preprocessing***)：在预处理步骤，将生成一个扩展名为 `.i` 的文件；使用命令 `gcc -E file.c` 操作
    - 头文件展开，不检查语法错误，将展开所有头（include）文件（任意）
    - 宏定义替换
    - 删除注释
    - 展开条件编译，根据条件来展开指令
- 编译 (***Compilation***) ：会生成一个扩展名为 `.s` 的文件，命令是：`gcc -S file.c`
    - 检查语法错误
    - 将文件翻译成汇编语言
- 汇编 (***Assembler***)：将汇编代码转换为纯二进制代码或机器代码（零和一）。此代码也称为目标代码；将生成一个带有 `.o` 扩展名的文件：`gcc -c file.c`
- 链接 (***Linker***)：链接是编译的最后一步。链接器将来自多个模块的所有目标代码合并为一个，如果使用了库也会引用。这个步骤也是包含前三个步骤的。`gcc file.o -o hello.exe`
    - 接收由汇编步骤生成的 .o 扩展名文件
    - 数据地址回填
    - 数据段合并
    - 库引入

## 变量

变量 (***variables***) 是用于存储数据的内存位置名称，可以改变的内容

### 变量的命名规则

- 不能以数字开头
- 由数字、字母，甚至是下划线 (`_`) 等特殊符号组成
- 变量名不能是任何&lt;a href=&#34;#keywords&#34;&gt;关键字 &lt;/a&gt;
- 变量名中不能有空格或空白
- 变量名是==区分大小写的==

### 变量的数据类型

C 语言中数据类型主要包含以下类型

| **变量类型** | 实际代表名称    | **描述**                               | **用途**                                                     |
| ------------ | --------------- | -------------------------------------- | ------------------------------------------------------------ |
| char         | Character       | 代表1bytes(8bit)，是以单引号引起的字符 | 通常以单个字母的形式使用X、r等，或 ASCII 字符集。            |
| int          | Integer         | 自然整数                               | 用来存储整数，如 4, 300, 8000 ...                            |
| float        | Floating- Point | 单精度浮点数                           | 表示实数值或小数值（7位小数），例如 20.8, 18.56 ...          |
| double       | Double          | 双精度浮点值                           | 比float类型要大 4bytes,允许15位小数                          |
| void         | Void            | 表示没有类型。                         | 这种数据类型是为了用于修饰没有意义函数或变量，如函数用其修饰标识没有返回值，参数用其修饰表示没有参数。 |

### 变量声明与定义

- **变量的定义**  (*Declaration*)：告诉编译器应为变量创建多少存储空间或者在哪里创建存储空间（借助于数据类型）
- **变量声明 **(*Definition*)：只声明不赋值的变量叫做变量定义， `int a`

定义与声明的区别：

- 变量定义会开辟内存空间。变量声明不会开辟内存空间
- 变量要想使用必须有定义
- 声明指示编译器存在变量，而定义表示编译器为变量创建的存储位置和存储量

### 变量的分类

- 全局变量 (***global***) ：在块或函数之外声明的变量称为全局变量
- 局部变量 (***Local***)：在块或函数中声明的一种变量
- 静态变量 (***static***)：使用 *static* 关键字声明的变量。该变量在各种函数调用之间保留给定值
- 自动变量 (***auto***)：变量具有自动存储期，程序在进入该变量声明所在的块时变量存在，程序在退出该块时变量消失
- 外部变量 (***extren***)：能够在多个源文件中共享一个变量 ` extern int a=10;`

### 变量的数据大小 &lt;sup id=&#34;datatype&#34;&gt;&lt;a href=&#34;#6&#34;&gt;[6]&lt;/a&gt;&lt;/sup&gt;

C 编程语言有两种基本数据类型：基本与衍生

| **类型**                               | **范围**                        | **大小（以字节为单位）** | 格式化符号 |
| -------------------------------------- | ------------------------------- | ------------------------ | ---------- |
| unsigned char                          | 0 ~ 255                         | 1                        | %c         |
| signed char/char                       | -128 ~ &#43;127                     | 1                        | %c         |
| unsigned int                           | 0 ~ 65535                       | 2                        | %u         |
| signed int or int                      | -32,768 ~ &#43;32767                | 2                        | %d         |
| unsigned short int                     | 0~ 65535                        | 2                        | %hu        |
| signed short int/short int             | -32,768 ~ &#43;32767                | 2                        | %hd        |
| unsigned long int                      | 0 ~ &#43;4,294,967,295              | 4                        | %lu        |
| signed long int/long int               | -2,147,483,648 ~ 2,147,483,647  | 4                        | %ld        |
| long long int                          | -(2^63) to (2^63)-1             | 8                        | %lld       |
| unsigned long long int                 | 0 to 18,446,744,073,709,551,615 | 8                        | %llu       |
| float  &lt;sup&gt;&lt;a href=&#34;#5&#34;&gt;[5]&lt;/a&gt;&lt;/sup&gt; | 7位精度                         | 4                        | %f         |
| double &lt;sup&gt;&lt;a href=&#34;#5&#34;&gt;[5]&lt;/a&gt;&lt;/sup&gt; | 15位精度                        | 8                        | %lf        |

### 变量类型

C语言中根据变量的声明周期和范围可以被分为两种类型 局部变量和全局变量与静态变量

#### 局部变量

局部变量 (***local variables***) 被声明在函数内部，只要函数存在，它们就只存在于内存中，直到函数结束，局部变量就会消失！

例如创建一个局部变量 a，a在函数运行时被创建在stack段中，当函数foo() 结束，被释放，故下列代码编译错误。

```c
#include &lt;stdio.h&gt;

void	foo(void)
{
	int	a;

	a = 10;
	printf(&#34;Foo function: Variable a = %d\n&#34;, a);
} // the variable &#39;a&#39; ceases to exist in RAM here.

int	main(void)
{
	foo();
	printf(&#34;Main: Variable a = %d\n&#34;, a);
	// ERROR : main does not know any variable named &#39;a&#39;!
	return (0);
}
```

&gt; Notes：函数的参数也是局部变量，如果需要外部更改，则通过指针方式传递进去

#### 全局变量

全局变量  (***global variables***)  是指在函数外部声明的变量；全局变量随函数生命周期结束时消失，因为全局变量被存储在内存结构的data分段中，是属于二进制文件本身的。另外==默认情况下，未被赋值的全局变量会被初始化为 0==。

```c
#include &lt;stdio.h&gt;

int	a; // Global variable initialized to 0 by default

void	foo(void)
{
	a = 42; // Global variable accessible without
		// having been declared in the function
	printf(&#34;Foo: a = %d\n&#34;, a); // a == 42
}

int	main(void)
{
	printf(&#34;Main: a = %d\n&#34;, a); // a == 0
	foo();
	printf(&#34;Main: a = %d\n&#34;, a); // a == 42
	a = 200;
	printf(&#34;Main: a = %d\n&#34;, a); // a == 200
	return (0);
}
```

&gt; Notes：局部变量的作用域高于全局变量，如果同名会被覆盖

**全局变量的作用域**

如果想在一个文件中使用另一个文件中定义的全局变量，需要使用关键字 ”***extern***“ 再次声明。这代表告诉编译器正在声明我们在程序文件的其他地方定义的变量。

例如下面代码中，`main.c` 中，使用 `extern` 关键字声明全局变量，表示我们在其他地方定义了这个变量。并做了 `foo()` 函数原型的声明。并在 `foo.c` 文件中，定义了全局变量 `a` 及 `foo()` 函数

main.c

```c
#include &lt;stdio.h&gt;

extern int	a; // 在其他文件内定义的全局变量

void foo(void);	// 定义在其他方面的函数，这种写法等同于extern void foo(void);

int	main(void)
{
	printf(&#34;Main: a = %d\n&#34;, a); // a == 100
	foo();
	printf(&#34;Main: a = %d\n&#34;, a); // a == 42
	a = 200;
	printf(&#34;Main: a = %d\n&#34;, a); // a == 200
	return (0);
}
```

void.c

```c
#include &lt;stdio.h&gt;

int	a = 100; // 全局变量的定义

void foo(void)
{
	a = 42;
	printf(&#34;Foo: a = %d\n&#34;, a); // a == 42
}
```

输出结果为

```c
Main: a = 100
Foo: a = 42
Main: a = 42
Main: a = 200
```

也可以使用头文件来定义，这种方式比上面的更好，示例只是说明全局变量

#### 静态变量

静态变量是指使用关键字 “***static***&#34; 修饰的变量，静态变量可以分为 **静态全局变量** 与 **静态局部变量** ，静态变量默认是全局的，因为他存储的地方是data区而不是堆，栈中。

静态变量有两点区分与全局变量：

- 在函数内部定义的静态变量是这个函数的全局变量（第一个结束的括号）
- 在函数外声明的静态变量仅在这个声明他的文件内有效。

下面代码会编译异常，因为b生命周期存在与for循环中

```c
#include &lt;stdio.h&gt;
#include &lt;string.h&gt;
void test()
{
    static int a = 10;
    for(int i=0; i&lt;10; i&#43;&#43;)
    {
        static int b = 10;
        a&#43;&#43;;
        b&#43;&#43;;
    }

    printf(&#34;value a is %d\n&#34;,a);
    printf(&#34;value b is %d\n&#34;,b);
}

void main()
{
    test();
}
```

##### 局部静态变量

局部静态变量不能说是真正的局部变量，因为其存储内存位置与局部变量不同，局部变量存储在堆，栈中，而静态变量存储在data中只是说会被限制在对应的作用域中。

下面代码说明了普通局部变量和静态局部变量的区别，由于存储位置不同，静态局部变量只是被访问限制在作用域中，而不会随函数结束释放掉。

```c
#include &lt;stdio.h&gt;

void foo(void)
{
	int	a = 100;
	static int	b = 100;

	printf(&#34;a = %d, b = %d\n&#34;, a, b);
	a&#43;&#43;;
	b&#43;&#43;;
}

int	main(void)
{
	foo();
	foo();
	foo();
	foo();
	foo();
	return (0);
}
```

输出结果

```c
a = 100, b = 100
a = 100, b = 101
a = 100, b = 102
a = 100, b = 103
a = 100, b = 104
```

##### 全局静态变量

全局静态变量是声明在函数外面用static修饰的变量，与全局变量不同的是，静态全局变量访问域被限制在声明它们的文件中，无法从程序的另一个文件中访问它。

在全局变量部分，可以通过关键字 ”***extern***“ 来访问全局变量，如果此时声明一个静态变量 a ，那么通过跨文件的方式这时编译器会提示 ”undefined reference to ‘a&#39;“。通常情况下使用这种场景被用于加速编译。

#### global VS local VS static

- 作用域方面不同：局部变量作用域仅在 同一个 `{}`，而静态变量和全局变量在为整个进程
- 访问作用域不同：全局变量为进程共享，局部变量为函数运行时，静态全局变量为定义它的文件，静态全局变量为 同一个 `{}`
- 存储位置不同，局部变量被存储与堆，栈中，而静态变量和全局变量被存储在data中

## 类型转换

### 隐式类型转换

隐式类型 (***Implicit***) 转换也称自动类型转换，这种类型的转换包含如下特点：

- 编译器自动完成，无需用户干预触发
- 当表达式中存在多种类型时触发，这是为了保证数据不被丢失
- 所有的数据类型都将升级为该类型最大值
- 转换的顺序为：bool -&gt; char -&gt; short int -&gt; int -&gt; unsigned int -&gt; long -&gt; unsigned -&gt; long long -&gt; float -&gt; double -&gt; long double
- 该转换类型会存在一些问题，如符号消失，数据丢失等。

```c
#include&lt;stdio.h&gt;
int main()
{
    int x = 10;    // integer x
    char y = &#39;a&#39;;  // character c
  
    // y 被隐式转换为 char类型，a=97
    x = x &#43; y;
     
    // 计算中，存在浮点数值，结果将被转换为float
    float z = x &#43; 1.0;

    
    // 自动转换为long long
    int h = 2147483648;
    // int 到 short int值溢出将为23352减去int大小65536
    short int g = 88888 &#43; x;
    printf(&#34;x = %d, z = %f\n&#34;, x, z);
    printf(&#34;g = %li, h = %lli\n&#34;, g, h);
    return 0;
}
```

### 显式类型转换

用户定义类型转换的过程称为显式类型转换 (***Explicit***)

```c
(type) expression
```

显示转换示例

```c
#include&lt;stdio.h&gt;
  
int main()
{
    float x = 1.2;
    // 显示转换一个float为int
    int sum = (int) x &#43; 1;
  
    printf(&#34;sum = %d&#34;, sum);
  
    return 0;
}
```

## 进制转换

### 十进制

十进制转二进制： 除2反向取余法

十进制转八进制：除8反向取余法

十进制转十六进制：除16反向取余法

例如：16进制转10进制

- 将十进制数除以 16。将除法视为整数除法
- 写下余数（十六进制）
- 将结果再次除以 16。将除法视为整数除法
- 重复步骤 2 和 3，直到结果为 0
- 求出的十六进制值是从最后到第一个的余数的数字序列

427的16进制

- 将数字除以 16，余数（小数部分乘16为余数），最终为1AB

    ![Decimal To Hexadecimal Conversion](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/decimal-to-hexadecimal.png)

### 八进制

8进制转10进制：从后向前，8的0次方，8的1次方，8的2次方...按照该顺序乘8的

- 8进制75转10进制为：$56&#43;5=61$
- 8进制77655转10进制为：$7(8^4)&#43;7(8^3)&#43;6(8^2)&#43;5(8^1)&#43;5(8^0)=28672&#43;3584&#43;384&#43;40&#43;5=32685$

2进制转8进制：自右向左，每3位一组，按421码转换。高位不足三位补0

- 1 010 111 010 110 二进制转八进制如下表，最后算出结果为12726

- | 4    | 2    | 1    |
    | ---- | ---- | ---- |
    | 0    | 0    | 1    |
    | 0    | 1    | 0    |
    | 1    | 1    | 1    |
    | 0    | 1    | 0    |
    | 1    | 1    | 0    |

### 十六进制

**16进制转10进制**：从后向前依次展开，16的0次方，16的1次方，16的2次方...，每位相加，例如：

- 0x1A = $16 &#43; 10 = 26$
- 15DE = $1(16^3)&#43;5(16^2)&#43;13(16^1)&#43;14= 4096&#43;1280&#43;208&#43;14=5598$

**16进制转二进制**：4位一组一次填充。例如 0X1A的二进制，即00011010如下

| 8    | 4    | 2    | 1    |
| ---- | ---- | ---- | ---- |
| 1    | 0    | 1    | 0    |
| 0    | 0    | 0    | 1    |

**二进制转16进制**：自右向左，每4位一组，按8421码转换。高位不足三位补0

例如 0001 0011 1111的16进制为，如下表 1 3 F(15)

| 8    | 4    | 2    | 1    |
| ---- | ---- | ---- | ---- |
| 0    | 0    | 0    | 1    |
| 0    | 0    | 1    | 1    |
| 1    | 1    | 1    | 1    |

## 源码反码补码

源码 (*** true form****),  反码 (***1‘s complement***) &lt;sup&gt;&lt;a href=&#34;#7&#34;&gt;[7]&lt;/a&gt;&lt;/sup&gt;, 补码 (***2‘s complement***) &lt;sup&gt;&lt;a href=&#34;#8&#34;&gt;[8] &lt;/a&gt;&lt;/sup&gt;是操作系统中存储和计算数据的一种方式

任何数据都以二进制机器码存储与计算机中。对于的机器码，第一位是用来表示正负值的：0是正数，1是负数。故要表示 ***-2***，对应的机器码是 ***10000010***。

机器码不可以直接通过权重展开计算，例如 ***10000010*** 为 $1(2^7) &#43; 1(2^1) = 130$ 。因为第一位是1，所以是负数，接下来计算后一位的权重展开为 $-2$

- 原码：机器码表示的值成为源码：如 43  = 00101011，-43 = 10101011

- 反码：符号位不变，其余位取反：如 43  = 00101011，-43 = 11010100

- 补码：符号位不变，counter code then LSB (***least significant bit***) &#43; 1：如 43  = 00101011，-43 = 11010101

    - | 128  | 64   | 32   | 16   | 8    | 4    | 2    | 1                         |
        | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ------------------------- |
        | 0    | 0    | 1    | 0    | 1    | 0    | 1    | 1                         |
        | 1    | 1    | 0    | 1    | 0    | 1    | 0    | 0&#43;1(如果需要进位则进一位) |
        | 1    | 1    | 0    | 1    | 0    | 1    | 0    | 1                         |

&gt; Note：二进制 10000000 为 -128

反码, 补码 是为了计算和存储正负数诞生的，正如 &lt;a href=&#34;#datatype&#34;&gt;C语言的数据结构&lt;/a&gt; 中 有符号和没符号表示的数值位置不一样。



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [keywords c language](https://www.programiz.com/c-programming/list-all-keywords-c-language)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [Arithmetic Operators](https://www.tutorialspoint.com/cprogramming/c_operators.htm)
&gt;
&gt; &lt;sup id=&#34;3&#34;&gt;[3]&lt;/sup&gt; [four stages of compilation c](https://medium.com/@danielavasquez_77768/the-four-stages-of-compilation-c-programming-and-gcc-compiler-9faadf2e273a)
&gt;
&gt; &lt;sup id=&#34;4&#34;&gt;[4]&lt;/sup&gt; [offline installation visual studio](https://docs.microsoft.com/en-us/visualstudio/install/create-an-offline-installation-of-visual-studio?view=vs-2022)
&gt;
&gt; &lt;sup id=&#34;5&#34;&gt;[5]&lt;/sup&gt; [difference float double](https://www.geeksforgeeks.org/difference-float-double-c-cpp/)
&gt;
&gt; &lt;sup id=&#34;6&#34;&gt;[6]&lt;/sup&gt; [data type in c](https://www.geeksforgeeks.org/data-types-in-c)
&gt;
&gt; &lt;sup id=&#34;7&#34;&gt;[7]&lt;/sup&gt; [1‘s complement](https://www.tutorialspoint.com/one-s-complement)
&gt;
&gt; &lt;sup id=&#34;8&#34;&gt;[8]&lt;/sup&gt; [2‘s complement](https://www.tutorialspoint.com/two-s-complement)







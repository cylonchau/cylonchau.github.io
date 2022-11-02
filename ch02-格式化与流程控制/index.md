# ch02 格式化与流程控制

[toc]

## 格式化 

### printf

 printf() 用于打印消息以及变量的值。

```c
#include&lt;stdio.h&gt;
int main() {
   int a = 24;
   printf(&#34;Welcome! \n&#34;);
   printf(&#34;The value of a : %d&#34;,a);
   getchar();
   return 0;
}
```

### sprintf

sprintf() 不打印字符串，是将字符值和格式化结构一并存储在一个数组中。

```c
int main()
{
    char buffer[50];
    int a = 10, b = 20, c;
    c = a &#43; b;
    sprintf(buffer, &#34;Sum of %d and %d is %d&#34;, a, b, c);
 
    // The string &#34;sum of 10 and 20 is 30&#34; is stored
    // into buffer instead of printing on stdout
    printf(&#34;%s&#34;, buffer);
 
    return 0;
}
```

### scanf

从标准输入读取用户输入的

| type      |                    Argument &amp; Description                    |
| --------- | :----------------------------------------------------------: |
| *****     |     读取标准输入用户输入的值，但不存储在对应接受的变量中     |
| width     |                   这个操作中读取的最大字符                   |
| type      |           指定要读取的数据类型以及预期如何读取数据           |

修饰符类型

| 类型                     | 标识符 |
| :----------------------- | :----- |
| `int`                    | `%d`   |
| `char`                   | `%c`   |
| `float`                  | `%f`   |
| `double`                 | `%lf`  |
| `short int`              | `%hd`  |
| `unsigned int`           | `%u`   |
| `long int`               | `%li`  |
| `long long int`          | `%lli` |
| `unsigned long int`      | `%lu`  |
| `unsigned long long int` | `%llu` |
| `signed char`            | `%c`   |
| `unsigned char`          | `%c`   |
| `long double`            | `%Lf`  |

格式化

|       Description        |            Code            |       Result        |
| :----------------------: | :------------------------: | :-----------------: |
| 接受字符类型保存在数组中 |     scanf(&#34;%19c&#34;, &amp;a);     | &#39;1234567890abcfefg&#39; |
|         整型类型         | scanf(&#34;%d&#34;, &amp;testInteger); |        &#39;10&#39;         |
|        多个接收值        |   scanf(&#34;%d%f&#34;, &amp;a, &amp;b);   |                     |

scanf的缺点

- 如果存储空间不足，数据能存储到内存中，但不被保护。
-  scanf 函数接收字符串时， 碰到 空格 和 换行 会自动终止。不能使用 scanf 的 %s 接收带有空格的字符串。

### 格式化标记符 &lt;sup &gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;

#### 标记符

| 标记符  |                  |
| ------- | ---------------- |
| %i / %d | int              |
| %c      | char             |
| %f      | float            |
| %s      | string           |
| %u      | unsigned decimal |
| %o      | octal            |
| %x      | hexadecimal      |

#### 对字符串填充

在 % 符号后添加一个零 (0)，可以对 printf 整数输出进行零填充

| Code                        | Result     |
| --------------------------- | ---------- |
| printf(&#34;%03d&#34;, 0);          | 000        |
| printf(&#34;%03d&#34;, 1);          | 001        |
| printf(&#34;%03d&#34;, 123456789);  | 123456789  |
| printf(&#34;%03d&#34;, -10);        | -10        |
| printf(&#34;%03d&#34;, -123456789); | -123456789 |

对于此类格式化方式总结有如下几种模式

|              Description              |          Code          |   Result    |
| :-----------------------------------: | :--------------------: | :---------: |
| 填充5位（默认以空白填充，左对齐填充） |  printf(&#34;&#39;%5d&#39;&#34;, 10);  |  &#39;     10&#39;  |
|         填充5位（右对齐填充）         | printf(&#34;&#39;%-5d&#39;&#34;, 10);  |  &#39;10     &#39;  |
|     填充5位“0”（默认左对齐填充）      | printf(&#34;&#39;%05d&#39;&#34;, 10);  |   &#39;00010&#39;   |
| 有符号的表示的数字（默认左对齐填充）  | printf(&#34;&#39;%&#43;5d&#39;&#34;, 10);  | &#39;      &#43;10&#39; |
|  有符号的表示的数字，右对齐填充空白   | printf(&#34;&#39;%-&#43;5d&#39;&#34;, 10); |  &#39;&#43;10    &#39;  |

#### 浮点数格式化

|                         Description                          |                Code                 |     Result     |
| :----------------------------------------------------------: | :---------------------------------: | :------------: |
|                         保留1位小数                          |     printf(&#34;&#39;%.1f&#39;&#34;, 10.3456);      |     &#39;10.3&#39;     |
|                         保留2位小数                          |     printf(&#34;&#39;%.2f&#39;&#34;, 10.3456);      |    &#39;10.35&#39;     |
|                 整数位最少8位宽度，小数位2位                 |     printf(&#34;&#39;%8.2f&#39;&#34;, 10.3456);     |   &#39;  10.35&#39;    |
|                 整数位最少8位宽度，小数位4位                 |     printf(&#34;&#39;%8.4f&#39;&#34;, 10.3456);     |   &#39; 10.3456&#39;   |
| 整数位最少8位，小数位2位，不足8位将用0填充（默认左对齐填充） |    printf(&#34;&#39;%08.2f&#39;&#34;, 10.3456);     |   &#39;00010.35&#39;   |
|     整数位最少8位，小数位2位，不足8位将用空白右对齐填充      |    printf(&#34;&#39;%-8.2f&#39;&#34;, 10.3456);     |   &#39;10.35  &#39;    |
|                 打印更大的浮点数，小数位2位                  | printf(&#34;&#39;%-8.2f&#39;&#34;, 101234567.3456); | &#39;101234567.35&#39; |

#### 字符串格式化

|                      Description                       |            Code             |   Result   |
| :----------------------------------------------------: | :-------------------------: | :--------: |
|                       字符串输出                       |  printf(&#34;&#39;%s&#39;&#34;, &#34;Hello&#34;);   |  &#39;Hello&#39;   |
| 保证输出结果是10位，不足位用空白填充（默认左对齐填充） | printf(&#34;&#39;%10s&#39;&#34;, &#34;Hello&#34;);  | &#39;   Hello&#39; |
|       保证输出结果是10位，不足位用空白右对齐填充       | printf(&#34;&#39;%-10s&#39;&#34;, &#34;Hello&#34;); | &#39;Hello   &#39; |

### 特殊字符

|      |      |
| ---- | ---- |
| \a   | audible alert        |
| \b   | backspace（退格）        |
| \f   | form feed （换页）       |
| \n   | newline（换行） |
| \r   | carriage return（回车）  |
| \t   | tab                  |
| \v   | vertical tab（垂直制表符） |
| \\   | backslash （反斜杠）      |

## 运算符

C语言中运算符优先级为下表所示

| 优先级 | 运算符                      | 说明                                | 关联性   |
| :----: | :-------------------------- | :---------------------------------- | :------- |
|   1    | `&#43;&#43;` `--`                   | 前缀/后缀 自增/减                   | 从左向右 |
|        | `()`                        | 函数调用                            |          |
|        | `[]`                        | 数组下标 (subscripting)             |          |
|        | `.`                         | 结构体成员访问                      |          |
|        | `-&gt;`                        | 指针结构体成员访问                  |          |
|   2    | `&#43;&#43;` `--`                   | 前缀/后缀 自增/减                   | 从右向左 |
|        | `&#43;` `-`                     | (Unary) 一元 &#43;/-（正负号）          |          |
|        | `!` `~`                     | 逻辑非与按位非                      |          |
|        | `(type)`                    | 转换                                |          |
|        | `*`                         | 取消引用                            |          |
|        | `&amp;`                         | 地址符                              |          |
|        | `sizeof`                    | Size-of                             |          |
|   3    | `*` `/` `%`                 | Multiplication, division, remainder | 从左向右 |
|   4    | `&#43;` `-`                     | Addition and subtraction            |          |
|   5    | `&lt;&lt;` `&gt;&gt;`                   | Bitwise left shift and right shift  |          |
|   6    | `&lt;` `&lt;=` `&gt;` `&gt;=` `==` `!=` | 关系运算符 &lt; , ≤ , &gt; , ≥ ,= ,  ≠    |          |
|   7    | `&amp;`                         | 按位与                              |          |
|   8    | `^`                         | 按位异或                            |          |
|   9    | `|`                         | 按位异或                            |          |
|   10   | `&amp;&amp;`                        | 逻辑与                              |          |
|   11   | `||`                        | 逻辑或                              |          |
|   12   | `?:`                        | 三元运算(Ternary conditional)       | 从右向左 |
|   13   | `=`                         | 赋值                                |          |
|        | `&#43;=` `-=`                   | 按和差赋值                          |          |
|        | `*=` `/=` `%=`              | 按乘积，商，余赋值                  |          |
|        | `&lt;&lt;=` `&gt;&gt;=`                 | 按左，右位移赋值                    |          |
|        | `&amp;=` `^=` `|=`              | 按位 与或非赋值                     |          |
|   14   | `,`                         | 逗号                                | 从左向右 |

## 流程控制 &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;

C语言中提供了两种流程控制(***flow control***)

- Branching
- Looping

### Branching

分支 (***Branching***) 将决定采取什么动作，循环将决定采取某种行动的次数。

#### if

形态1：

```c
if (expression)
  statement;

if (expression)
{
    Block of statements;
}
```

形态2:

```c
if (expression)
{
    Block of statements;
}
else
{
    Block of statements;
}
```

形态3：

```c
if (expression)
{
    Block of statements;
}
else if(expression)
{
    Block of statements;
}
else
{
    Block of statements;
}
```

### 三元运算

`&lt;value1&gt; ? &lt;value2&gt; : &lt;value3&gt;` 是三元运算符，因为它需要三个值，这是 C 中唯一的三元运算符。语法

```c
if condition is true ? then X return value : otherwise Y value;
```

### switch

```c
switch( expression )
{
    case constant-expression1:	statements1;
    [case constant-expression2:	statements2;]    
    [case constant-expression3:	statements3;]
    [default : statements4;]
}
```

***break*** 关键字用作退出 switch 语句。在 switch case 中满足条件，则执行继续到下一个 case 子句，如果没有明确指定执行应该退出 switch 语句。

***default*** 关键字用于在所有case中都不满足条件，则执行default

case穿透：case分支中如果,没有 ***break***；那么它会向下继续执行下一个case分支.

### if VS switch

- **检查表达式**：if-else 可以基于值或条件检查表达式，而 switch 语句仅基于字符表达式或整数类型检查表达式。
- **运行速度**：在大量条件检查中进行选择，switch 语句的运行速度将比使用 if-else 的逻辑快得多。
- **适合条件不同**：if-else 适合导致布尔值的可变条件，而 switch 适合固定值。
- **可读性**：if-else较switch-case语句可读性较差

### Looping

循环 (***Looping***) 提供了一种重复命令和控制重复次数的方法。

### while

while 是 c 语言中最基础的循环，while将检查expression，直到expression为false将推出循环

```c
while ( expression )
{
   Single statement 
   or
   Block of statements;
}
```

### for

for是类似与while的循环，只是语法上不同，for提供了三个表达式

```c
for( expression1; expression2; expression3)
{
   Single statement
   or
   Block of statements;
}
```

- expression1 - 通常用于初始化变量（在此初始化的变量作用域仅为该循环中）
- expression2 - 条件表达式，只要该表达式为true则循环将一直被执行
- expression3 -  修饰符，通常用于变量的自增自减操作
- 三个表达式都可以为空，这种场景下循环将一直进行

### do...while

类似与while ，只不过do..while循环，在循环结束开始检查测试条件。这意味着循环的内容将==至少执行一次==。

```c
do
{
   Single statement
   or
   Block of statements;
}
while(expression);
```

### break VS continue

C语言提供了两个命令来控制循环：

- break，退出循环或switch
- continue，跳过当前迭代 (***iteration***)，继续循环

```c
#include 
main()
{
    int i;
    int j = 10;

    for( i = 0; i &lt;= j; i &#43;&#43; )
    {
       if( i == 5 )
       {
          continue;
       }
       printf(&#34;Hello %d\n&#34;, i );
    }
}
```

输出结果将没有第五次迭代

```bash
Hello 0
Hello 1
Hello 2
Hello 3
Hello 4
Hello 6
Hello 7
Hello 8
Hello 9
Hello 10
```

## goto

goto 声明在C语言中提供了了一个无条件跳转到goto label出的

```c
goto label;
..
.
label: statement;
```

下面例子中，将从10开始执行，跳过15继续从16开始到20结束。

```c
#include &lt;stdio.h&gt;
 
int main () {

   /* 局部变量定义 */
   int a = 10;

   /* do循环体 */
   LOOP:do {
   
      if( a == 15) {
         /* 跳出迭代 */
         a = a &#43; 1;
         goto LOOP;
      }
		
      printf(&#34;value of a: %d\n&#34;, a);
      a&#43;&#43;;

   }while( a &lt; 20 );
   return 0;
}
```




&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [printf format](https://alvinalexander.com/programming/printf-format-cheat-sheet/)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [control_statements](https://www.tutorialspoint.com/ansi_c/c_control_statements.htm)





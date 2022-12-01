# ch05 pointer

[toc]

## 指针 

### 指针声明 <sup><a href="#1">[1]</a></sup>

指针/指针变量 (***pointer***) 是用于存储地址的变量

使用 `&` 运算符 来访问变量的地址。例如

```c
#include <stdio.h>

void main()
{
   int a = 100;
   printf("%x", &a);
}
```

输出结果为 16进制的内存地址

```c
61fe1c
```

使用地址运算符 `*` 可以从变量地址中获取变量的值，这个行为被称为间接引用/解引用(***indirection/dereferencing***)。例如：

```c
#include <stdio.h>

void main()
{
   int a = 100;
   printf("%d", *(&a)); 
   // 也可以写为，因为*与&优先级相同，从右到左的顺序，所以有没有()意思是相同的
   printf("%d", *&a);
}
```

输出结果为 100

### 指针变量

指针变量是指存储一个变量的地址的变量，可以使用符号 `*` 来修饰变量，定义语法为：

```c
dataType *pointerVariableName = &variableName;
```

例如，下面的两个输出结果是相同的地址

```c
#include <stdio.h>

void main()
{
   int a = 100;
   int *pointer;
   pointer = &a;
   printf("address of a is: %x\n",&a);
   printf("address of pointer is: %x\n",pointer);
}

address of a is: 61fe14
address of pointer is: 61fe14
```

> Notes：指针可以通过变量修改也可以直接通过地址进行修改，指针变量就是通过地址进行修改

#### 修饰符说明

|修饰符| 说明 |
| ---- | --------------------------- |
| *    | 两个用途：<br>指针变量的声明<br>返回被引用变量的值 |
| &    | 返回变量地址 |

#### 使用const修饰指针 <sup><a href="#3">[3]</a></sup>

使用关键字 `const` 修饰的指针变量是不能改变指针变量所指向的地址的变量，通俗来讲即不能被改变值的指针变量

声明语句 

```c
<type of pointer> *const <name of pointer>;  
int *const ptr;  
```

**const位置**：const修饰的部分（const所在位置）不可改变

- 例如 `const int *p;` 与 `int const *p;` 这里修饰的都是 `*p` 故

    -  `*p` （变量地址）不能被改变

    - `p` （变量值）可以被改变

    - ```c
        #include<stdio.h>
        #include<stdlib.h>
        void main()
        {
          int a = 10;
          int b = 20;
          int const *p;
          *p = &b; // result assignment of read-only location '*p'
          p = 30; // result 30
        
          printf("%d",p);
        }
        ```

- `int * const p; ` const在\*后p前，这里修饰的都是 `p` 故
    - p不可以被修改
    - \*p 可以被修改

- `const int *const p;` 这里 const 修饰 \* 和 p，所以两个都不可修改

> Notes：通常情况下常用只有第一种情况

**使用场景**：最为参数形参修饰该参数为只读参数，例如 printf 

```c
int printf (const char *__format, ...)
```

### 指针的类型 <sup><a href="#2">[2]</a></sup>

C语言中包含多种指针类型：

- 空指针(***Null Pointer***)
- 野指针(***Wild Pointer***)
- 悬空指针(***Dangling pointer***)
- 泛型指针(***void Pointer***)
- 一些早期Dos中的概念
    - 近指针(***Near***)：不能存储大小大于 16 位的地址
    - 远指针(***Far***)：32 位大小的指针
    - 大指针(***Huge***)：类似于远指针。

#### 空指针

在声明期间将 NULL 分配给指针的指针称为 空 (***NULL***) 指针，例如

```c
#include <stdio.h>

void main()
{
   int *var = NULL;
   printf("address of var is: %p\n",var);
}
```

- 空指针不能解引用：NULL指针因引用是一个非法的操作，在解引用之前，必须确保它不是一个NULL指针
- 空指针不能拷贝内容：`strcpy(*p,"1111);`

#### 泛型指针

使用 `void` 关键字声明指针变量，可以接受任意一种类型的变量地址，如果需要使用泛型指针，需要强转为对应类型才可以使用。如下示例：

```c
#include <stdio.h>

void main()
{
   int a = 6666;
   void *p = &a;
   printf("address of p is: %p\n", (int*) p);
   printf("value of p is: %d\n", *(int*) p);
}

address of p is: 000000000061FE14
value of p is: 6666
```

#### 野指针

野指针是指，没有有效地址的空间的指针，例如声明了指针变量没有对其赋值，这种情况下会出现 `Segmentation fault` 异常。

```c
#include <stdio.h>

void main()
{
   int *p;
   printf("address of p is: %p\n", *p); 
}
```

再例如一个无效的地址空间也会发生  `Segmentation fault` 异常；例如下列赋值中，指针p赋值被视为一个内存地址，而不是变量的值，这个地址无效。

```c
#include <stdio.h>

void main()
{
   int *p;
   *p = 10000;
   printf("address of p is: %p\n", *p);
}
```

野指针出现场景：

- 指针变量声明但未初始化
- 指针释放后未置空
- 指针操作超出变量作用域

避免野指针的出现：

- 初始化置 NULL
- 释放后置 NULL

#### 悬空指针

悬空指针是指”已经被释放的内存“的指针变量，此时这个地址空间是无效的。如下所示

```c
#include<stdio.h>
#include<stdlib.h>
void main()
{

  int *P=(int *)malloc(sizeof(int));
  int a=5;
  P=&a;
  free(P);
  printf("After deallocating its memory *p=%d", *P);
}
```

### 指针的偏移量

指针步长是指指针在运算是偏移多少字节

- 指针+1之后跳跃的字节数取决于指针的类型，int 4, char 1, struct struct长度

- 指针解引用时需要转换成对应的数据类型，从而判断被解引用后的大小，`* (int *) p` 指针类型变量转换为指针int类型变量

- 对于结构体指针来说，offsetof函数定位属性对应的偏移量

    ```c
    #include<stddef.h>
    offsetof(<struct struct_name>, <obj_name>)
    ```

## 指针数组

在C语言中 数组是由两部分组成，数组名与数组本身。

```c
#include<stdio.h>
void main()
{
	int a[10] = {0};
	a[1] = 20;
	for (int i=0; i< sizeof(a) / sizeof(a[0]); i++){
		printf("%d\n",a[i]);
	}
	printf("arr %p = &arr %p", a, &a[0]);
   
}
```

上面例子中，变量a是一个数组，而变量a代表的是一个指向该数组第一个元素的地址，`a=&a[0]` ，这是一个const修饰的指针是不可以被改变的。

可以看到变量a和 `&a[0]` 值是相同的，而一个const修饰的指针变量是不可改变的，故a不能被赋值，下列代码是不合法的。总结为：**不允许将任何地址分配给数组变量**

```c
#include<stdio.h>
void main()
{
	int a[10] = {0};
	int b[1] = {0};
	int c = 10;
	a = &b;
	a = &c;
	for (int i=0; i< sizeof(a) / sizeof(a[0]); i++){
		printf("%d\n",a[i]);
	}
	printf("arr %p = &arr %p", a, &a[0]);
   
}
```

### 使用指针访问数组

上面知道了，数组变量指向的是数组的起始元素（第一个元素）的地址指针，那么通过指针可以对数组进行访问。

因为 `a = &a[0]` 那么 `a[0] = *a`，由此可推导出下列公式：

- `&a[1] = a+1` （取数组元素的地址）   那么 `a[1] = *(a+1)` （取数组元素的值） 
- `&a[2] = a+2`   那么 `a[2] = *(a+2)`

这种情况下数组的访问就有四种方法

### Array VS Pointer <sup><a href="#4">[4]</a></sup>

- 数组名是常量，指针是变量
- sizeof(array) 得到的是数组实际占用内存空间的字节数，sizeof(pointer) 是4/8 取决于操作系统

### 指针运算

#### 左值和右值 <sup><a href="#5">[5]</a></sup>

了解对于指针运算前，需要对左值(***lvalue***)和右值(***rvalue***)进行了解

- 左值：通常来说是在占有内存地址（即具有地址）的对象
    - 具有存储数据的内力，例如变量
    - 不能是函数，表达式，或常量
    - 综合来说，左值可以是以下几种：
        - 任何类型的变量：int, float, pointer, struct等
        - 数组的下标表达式，如a[1]
        - 括号内的表达式（指针）
        - 指针的间接引用
        - 常量（不可改变的左值）
        - 通过指针访问对象属性或成员 (-> or .)
- 右值：在内存中没有占有内存地址的对象
    - 返回不可改变的表达式或值，例如a+b是一个常量，函数运行结果是一个右值

**左值的示例**

```c
#include<stdio.h>
void main()
{	
	// 声明变量a为int类型
	int a;
	// a是一个左值，引用对象为int
	a = 1;
	// 左值a出现在右边的场景
	int b = a; 
	
	// 非法，a是左值
	9 = a;

	// 左值，*p是指针，p就是值，*p+4是后面一个int类型的地址，是左值
	int *p;
	int a = 10;
	p = &a;
	// 这里实际上是指针运算，p+0为自己，p存储指针 *p为值，那么a=10000,p=10000
	*(p+0) = 10000;   
	printf("%p\n",a);
	printf("%p\n",*p);
	printf("%d\n",a);
	printf("%d\n",*p);
}
```

**右值的示例**

```c
// 声明变量a b
int a = 1, b;

// 非法，a+1为常量，不是左值
a + 1 = b; 
 
// 声明指针变量 p q
int *p, *q; // *p, *q 为左值
 
*p = 1; // 合法，左值可以赋值
 
// 非法 - "p + 2" 是右值
p + 2 = 18;
 
q = p + 5; // "p + 5" 是右值，合法
 
// 解引用表达式是左值
*(p + 2) = 18;
 
p = &b;
 
int arr[20]; // 数组元素访问arr[12] = *(arr+12) 所以是左值有效
 
struct S { int m; };
 
struct S obj; // obj and obj.m are lvalues
 
// ptr-> 等于  (*ptr).m 是左值有效
```

一元表达式需要有左值，当a是左值&a才生效，12本身是一个右值，不能&12

```c
int a, *p; // a 和 *p都是左值
p = &a; // 合法，&a是常量为右值，p是左值
&a = p; // 缺少左值，非法
```

三元表达式是一个右值（C++是左值）

```c
(  x < y ? y : x) = 10; // c无效，c++有效
```

**左值 VS 右值**

- 左值为内存中可识别的对象，右值为一个常量（广义上，不是const）
- 左值可以在左边和右边，右值必须在右边
- 右值必须有左值才生效
- 指针运算可左可右，变量运算是右值

#### arithmetic <sup><a href="#6">[6]</a></sup>

C语言中，指针支持四种算术运算符，吧地址当作数值进行算数运算

| 运算符                            | 说明                                        |
| --------------------------------- | ------------------------------------------- |
| =                                 | 可以将值赋值给指针                          |
| +                                 | 从指针加整数值以指向不同的内存位置。        |
| -                                 | 从指针中减去整数值以指向不同的内存位置      |
| 比较运算（==, !=, <, >, <= , >=） | 仅比较两个指针地址，例如<br>pointer == NULL |
| ++                                | 指针使用递增运算符将向前位移一位            |
| --                                | 指针使用递减运算符将向后位移一位            |

- 当对指针变量进行递增和递减操作时，会改变指针变量本身所在地址空间
- 当对指针变量进行+-运算时，不会改变指针变量本身

### 指针数组

指针数组是指数组存储的内容是指针，即数组内所有的元素都是指针

```c
#include<stdio.h>
void main()
{	
	int a = 10;
	int b = 20;
	int c = 30;
	int *arr[] = {&a, &b, &c};
}
```

指针数组本质也是一个多级指针，例如一个2D数组每行(***rows***) 存储的值是一列(***colums***)的地址

```c
#include<stdio.h>
void main()
{	
	int a[] = { 10 };
	int b[] = { 20 };
	int c[] = { 30 };
	int *arr[] = {a, b, c};
}
```

### Pointer VS Array

- sizeof
    - `sizeof(array)` 返回数组中所有元素占用内存的大小
    - `sizeof(pointer)` 只返回指针变量本身用内存的大小
- &运算符
    - 数组名是 `&array[0]` 的别名，返回数组中第一个元素的地址
    - `&pointer` 返回指针的地址
- 指针变量可以赋值，而数组变量不可以
- 数组是收集了相同类型元素的集合，而指针是一个存储地址的变量

## 多级指针 <sup><a href="#7">[7]</a></sup>

一个指针用于存储变量的地址，而另一个指针用于存储第一个指针的地址，这种指针被称为多级指针 (***Multi-Pointer or Pointer to Pointer***)

声明多级指针必须在指针变量名称前多家一个 ”*“

```c
int **p;
```

通过示例更好的了解多级指针

```c
#include <stdio.h>
  
// C program to demonstrate pointer to pointer
int main()
{
    int var = 123;
  
    // pointer for var
    int *ptr1;
  
    // double pointer for ptr2
    int **ptr2;

	// third pointer for ptr2
    int ***ptr3;
  
    // storing address of var in ptr1
    ptr1 = &var;
    // Storing address of ptr2 in ptr1
    ptr2 = &ptr1;
	// Storing address of ptr3 in ptr2
	ptr3 = &ptr2;
      
    // Displaying value of var using
    // both single and double pointers
    printf("Value of var = %d\n", var );
    printf("Value of var using single pointer = %d\n", *ptr1 );
    printf("Value of var using double pointer = %d\n", **ptr2);
	printf("Value of var using third pointer = %d\n",  ***ptr3);
    
  	return 0;
}
```

输出结构

```c
Value of var = 123
Value of var using single pointer = 123
Value of var using double pointer = 123
Value of var using third pointer = 123
```

Note：

- 多级指针，不能跨越定义，即二级指针必须拥有一级指针才可以
- 此时的 `*ptr`  可以是左值或右值
    - 作为左值时，存储数据到该变量存储的地址空间内
    - 作为右值时，取出该空间内的内容

```c
#include <stdio.h>
  
// C program to demonstrate pointer to pointer
int main()
{
    int var = 123;
  
    // pointer for var
    int *ptr1;
  
    // double pointer for ptr2
    int **ptr2;

	// third pointer for ptr2
    int ***ptr3;
  
    // storing address of var in ptr1
    ptr1 = &var;
    // Storing address of ptr2 in ptr1
    ptr2 = &ptr1;
	// Storing address of ptr3 in ptr2
	ptr3 = &ptr2;
      
    // Displaying value of var using
    // both single and double pointers
	***ptr3 = 100;
	printf("l-value test, the result of value = %d\n", var );
    printf("l-value test, the result of ptr1 = %d\n",  *ptr1);
	printf("l-value test, the result of ptr2 = %d\n",  **ptr2);
    printf("l-value test, the result of ptr3 = %d\n",  ***ptr3);

	var = (***ptr3+1);
	printf("r-value test, the result of value = %d\n", var );
	return 0;
}
```

输出结果为

```bash
l-value test, the result of value = 100
l-value test, the result of ptr1 = 100
l-value test, the result of ptr2 = 100
l-value test, the result of ptr3 = 100
r-value test, the result of value = 101
```

### 指针和函数

### 指向普通数据类型的指针

指针可以被当作函数参数传递，会改变原有的变量值

```c
#include <stdio.h>
void swap(int *n1, int *n2);

int main()
{
    int num1 = 5, num2 = 10;

    // address of num1 and num2 is passed
    swap( &num1, &num2);

    printf("num1 = %d\n", num1);
    printf("num2 = %d", num2);
    return 0;
}

void swap(int* n1, int* n2)
{
    int temp;
    temp = *n1;
    *n1 = *n2;
    *n2 = temp;
}
```

### 指向函数的指针  <sup><a href="#8">[8]</a></sup>

C语言中，指针也可以被指向一个函数，下面代码是一个指向函数的指针

```c
#include <stdio.h>
// 定义一个无返回值的常规函数
void fun(int a)
{
    printf("Value of a is %d\n", a);
}
  
int main()
{
    // fun_ptr是一个指针类型，他指向函数fun的地址
    void (*fun_ptr)(int) = &fun;
  
    /* 也可以写为如下代码
       void (*fun_ptr)(int);
       fun_ptr = &fun; 
    */
  
    // 调用指向函数的指针
    (*fun_ptr)(10);
  
    return 0;
}
```

声明指针函数函数的说明，通常情况下声明函数语法为 `int foo(int);`  则代表声明了一个foo函数，具有int类型参数和int类型的返回值，而在中间加一个 ”*" 则可以表示一个指针函数的定义 `int * foo(int);` 这种类型是错误的。

因为在c语言中，`*` 的优先级要高于 `()`， 上面说到的情况则表示了声明一个foo函数，int类型的参数和 *int 类型的返回值。所以必须使用 () 改变其优先级 

```c
int (*foo)(int);
```

### 函数指针数组

函数指针数组是指元素为函数指针的数组，有些特殊的地方是，定义时不能定义为数组指针，需要定义为函数指针，函数指针数组也可以替代switch

```c
#include <stdio.h>
void add(int a, int b)
{
    printf("Addition is %d\n", a+b);
}
void subtract(int a, int b)
{
    printf("Subtraction is %d\n", a-b);
}
void multiply(int a, int b)
{
    printf("Multiplication is %d\n", a*b);
}
  
int main()
{
    // fun_ptr_arr is an array of function pointers
    void (*fun_ptr_arr[])(int, int) = {add, subtract, multiply};
    unsigned int ch, a = 15, b = 10;
  
    printf("Enter Choice: 0 for add, 1 for subtract and 2 "
            "for multiply\n");
    scanf("%d", &ch);
  
    if (ch > 2) return 0;
  
    (*fun_ptr_arr[ch])(a, b);
  
    return 0;
}
```

关于函数指针的说明：

- 普通指针指向的数据，函数指针指向的是代码
- 函数名代表函数的地址
- 函数去指不用加取址符 “&”，函数指针变量调用不用加“ \* ”
- 函数指针可以作为参数，也可以作为返回值

### 数组与函数

- **数组作为参数时**：数组作为函数参数传入时，传递不再是整个数组，而是数组的第一元素的地址，也就是指针，此时不能用size()获取数组的元素，获取到的时指针类型的大小。
- **数组作为返回值时**：不允许返回数组，返回的是数组第一个元素指针，sizeof() 查看的大小也是指针类型的大小

## main函数的参数

在C语言中main()函数之前提供了一个函数 `_start()`，但通常情况下 main() 是作为程序执行的第一个函数。main() 函数提供了两个参数，***argc*** 和 ***argv*** 。

- ***argc*** 命令行传入的参数数量，int类型
- ***argv*** 命令行传入的实际参数，参数索引从1开始，0为程序本身名称
    - 正常情况下声明main函数：` main(int argc, char *argv[])`
    - `**argv` 是 `*argv[]` 的另一种表现方式 `main(int argc, char **argv)`

> Notes：这里有一个比较难理解的地方就是二级指针作为形参来替换指针数组。
>
> 由于二级指针变量存放为一个一级指针的地址，而数组名本身是数组首元素的地址，其后的每一个元素都是指针就是将首元素指针传入。由于上面讲到数组作为参数传入时传入的是指针而不是数组本身。所以 *argv[] 与 **argv 是等价的。


> **Reference**
>
> <sup id="1">[1]</sup> [c pointer](https://www.geeksforgeeks.org/pointers-in-c-and-c-set-1-introduction-arithmetic-and-array/)
>
> <sup id="2">[2]</sup> [pointer type](https://www.guru99.com/c-pointers.html)
>
> <sup id="3">[3]</sup> [const pointer](https://www.javatpoint.com/const-pointer-in-c)
>
> <sup id="4">[4]</sup> [pointer VS array](https://www.freecodecamp.org/news/pointers-in-c-are-not-as-difficult-as-you-think/#1-what-exactly-are-pointers)
>
> <sup id="5">[5]</sup> [lvalue VS rvlaue](https://www.geeksforgeeks.org/lvalue-and-rvalue-in-c-language/)
>
> <sup id="6">[6]</sup> [pointer arithmetic](https://aticleworld.com/pointer-arithmetic/)
>
> <sup id="7">[7]</sup> [pointer to pointer](https://www.geeksforgeeks.org/double-pointer-pointer-pointer-c/)
>
> <sup id="8">[8]</sup> [function pointer in c](https://www.geeksforgeeks.org/function-pointer-in-c/)







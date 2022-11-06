# ch07 composite type

[toc]

## Overview

C语言中复合类型 (***composite type***) 是指用户自定义类型，通常由多种元素组成的类型，其元素被紧密存储在内存中。C语言常见的复合类型有：

- 数组
- 字符串
- 结构体
- 联合类型

## 结构体 &lt;sup&gt;&lt;a href=&#34;#1&#34;&gt;[1]&lt;/a&gt;&lt;/sup&gt;

结构体 (***structure***) 是指用户定义的数据类型，允许将不同类型的多个元素组合在一起，来创建出更复杂的数据类型，类似于数组，但又区别于数组，数组只能保存同类型的元素，而结构体可以保存不同类型的元素。

### 定义

声明结构体的语法如下

```c
 struct structureName
 {
     dataType memberVariable1;
     datatype memberVariable2;
     ...
 } variable01, variable02...;
```

这里需要注意的一些地方：

- struct是关键字，structureName定义的新数据类型，variable{}是作为使用 ***structureName*** 声明的新变量名
- 每个成员方法结尾都是 “;&#34; 而不是逗号 ”,&#34;
- 结构体不能递归
- 变量可以有多个

例如声明一个学生的结构体，而student是作为一个新的数据类型存在

```c
struct student
{
    char name[20];
    int roll;
    char gender;
};
```

&gt; Notes：在定义（创建）结构体变量前，结构体成员不会占用内存

### 声明

使用结构体声明变量

也可以一次性定义结构体和声明变量

```c
struct student
{
    char name[20];
    int roll;
    char gender;
} stu1,stu2;
// 结构体名称可以省略
struct
{
    char name[20];
    int roll;
    char gender;
} stu1,stu2;
```

### 赋值

在声明结构体后，student结构体只是自定义数据结构，要使用还需要进行初始化，或者赋值

```c
stu1 = {&#34;zhangsan&#34;, 20, 0};
```

或者

```
stu1.name=&#34;zhangsan&#34;;
```

或者

```c
struct Student
{
    char name[25];
    int age;
    char branch[10];
    char gender;
}stu1 = {&#34;zhangsan&#34;, 20, 0};
```

或者使用不同顺序进行初始化

```c
stu1 = {.age=20, .gender=0, .name=&#34;zhangsan&#34;};
```

也可以仅初始化部分成员，未初始化的成员应该按顺序在后位

```c
stu1 = {&#34;zhangsan&#34;};
```

### 访问

访问结构体可以使用符号 ”.“ 来访问，成员名称==.==成员属性

```c#include&lt;stdio.h&gt;
#include&lt;string.h&gt;

struct Student
{
    char name[25];
    int age;
    char branch[10];
    char gender;
};

int main()
{
    struct Student s1;

    s1.age = 18;
    strcpy(s1.name, &#34;Viraaj&#34;);
    printf(&#34;Name of Student 1: %s\n&#34;, s1.name);
    printf(&#34;Age of Student 1: %d\n&#34;, s1.age);
    return 0;
}
```

也可以使用scanf() 赋值

### 结构体运算

结构体不能够执行算术运算符 ***&#43;, -, x, ÷*** ，关系运算符 ***&lt; &gt; &lt;= &gt;=***, 等式运算符，但是可以在两个相同结构体变量的场景下进行赋值运算。

```c
 /* 无效的操作 */
st1 &#43; st2
st1 - st2
st1 == st2
st1 != st2 etc.

 /* 在相同类型下的结构体，操作是有效的 */
st1 = st2
```

因为C语言没有提供比较运算，所以没法进行结构体比较，需要自行比较结构体成员来比较结构体是否一样

```c
#include &lt;stdio.h&gt;
#include &lt;string.h&gt;

 struct student
    {
        char name[20];
        double roll;
        char gender;
        int marks[5];
    }st1,st2;


void main()
{
    struct student st1= { &#34;Alex&#34;, 43, &#39;M&#39;, {76, 78, 56, 98, 92}};
    struct student st2 = { &#34;Max&#34;, 33, &#39;M&#39;, {87, 84, 82, 96, 78}};

    if( strcmp(st1.name,st2.name) == 0 &amp;&amp; st1.roll == st2.roll)
        printf(&#34;Both are the records of the same student.\n&#34;);
    else printf(&#34;Different records, different students.\n&#34;);

     /* Copiying the structure variable */
    st2 = st1;

    if( strcmp(st1.name,st2.name) == 0 &amp;&amp; st1.roll == st2.roll)
        printf(&#34;\nBoth are the records of the same student.\n&#34;);
    else printf(&#34;\nDifferent records, different students.\n&#34;);
}
```

输出结果

```c
Different records, different students.

Both are the records of the same student.
```

### 结构体数组

结构体数组是指数组元素是结构体，例如下面声明一个类型为student的数组

```c
struct student
{
    char name[20];
    double roll;
    char gender;
    int marks[5];
};

struct student stu[4];
```

初始化和访问可以通过循环进行

```c
for (int i = 0; i &lt; 4; i&#43;&#43;)
{
    printf(&#34;Enter name:\n&#34;);
    scanf(&#34;%s&#34;,&amp;stu[i].name);
    printf(&#34;Enter roll:\n&#34;);
    scanf(&#34;%d&#34;,&amp;stu[i].roll);
    printf(&#34;Enter gender:\n&#34;);
    scanf(&#34; %c&#34;,&amp;stu[i].gender);

    for( int j = 0; j &lt; 5; j&#43;&#43;)
    {
        printf(&#34;Enter marks of %dth subject:\n&#34;,j);
        scanf(&#34;%d&#34;,&amp;stu[i].marks[j]);
    }

    printf(&#34;\n-------------------\n\n&#34;);
}

/* Finding the average marks and printing it */

for(int i = 0; i &lt; 4; i&#43;&#43;)
{
    float sum = 0;
    for( int j = 0; j &lt; 5; j&#43;&#43;)
    {
        sum &#43;= stu[i].marks[j];
    }
    printf(&#34;Name: %s\nAverage Marks = %.2f\n\n&#34;, stu[i].name, sum / (sizeof(stu[i].marks) / sizeof(stu[i].marks[0])));
}
```

将代码整合为一起

```c
#include &lt;stdio.h&gt;
struct student
{
    char name[20];
    double roll;
    char gender;
    int marks[5];
};

struct student stu[4];

void main()
{
    for (int i = 0; i &lt; 4; i&#43;&#43;)
    {
        printf(&#34;Enter name:\n&#34;);
        scanf(&#34;%s&#34;,&amp;stu[i].name);
        printf(&#34;Enter roll:\n&#34;);
        scanf(&#34;%d&#34;,&amp;stu[i].roll);
        printf(&#34;Enter gender:\n&#34;);
        scanf(&#34; %c&#34;,&amp;stu[i].gender);

        for( int j = 0; j &lt; 5; j&#43;&#43;)
        {
            printf(&#34;Enter marks of %dth subject:\n&#34;,j);
            scanf(&#34;%d&#34;,&amp;stu[i].marks[j]);
        }

        printf(&#34;\n-------------------\n\n&#34;);
    }

    /* Finding the average marks and printing it */

	for(int i = 0; i &lt; 4; i&#43;&#43;)
    {
        float sum = 0;
        for( int j = 0; j &lt; 5; j&#43;&#43;)
        {
            sum &#43;= stu[i].marks[j];
        }
        printf(&#34;Name: %s\nAverage Marks = %.2f\n\n&#34;, stu[i].name, sum / (sizeof(stu[i].marks) / sizeof(stu[i].marks[0])));
    }
}
```

### 结构体嵌套结构体

嵌套结构体表示，结构体的成员是另外一个结构体

```c
struct date
{
    int date;
    int month;
    int year;
};

struct student
{
    char name[20];
    int roll;
    char gender;
    int marks[5];
    struct date birthday;
};
```

其定义语法为：`struct &lt;other struct&gt; &lt;member_name&gt;;` 这里 `birthday` 是名为data类型结构体

&gt; Notes：结构体内部不能嵌套自己

访问嵌套结构体和正常结构体访问一样使用符号，成员名称==.==成员属性==.==成员属性

```c
stu1.birthday.date
stu1.birthday.month
stu1.birthday.year
stu1.name
```

### 结构体内存分配

结构体声明后是不占用内存，只有被初始化后才占用内存，结构体内每个成员会被分配到连续的内存内，sizeof()的大小是每个元素所占用的大小。

示例代码为一个student的结构体，有四个成员，name为20 bytes的字符串，roll是4字节的int类型，gender是1字节的char，marks为5个元素的数组，那么这个结构体的总大小应该为 $20&#43;4&#43;1&#43;5\times4$

```c
struct student
{
    char name[20];
    int roll;
    char gender;
    int marks[5];
} stu1;
```

将上述代码整合为

```c
void main()
{
    printf(&#34;Sum of the size of members = %I64d bytes\n&#34;, sizeof(stu1.name) &#43; sizeof(stu1.roll) &#43; sizeof(stu1.gender) &#43; sizeof(stu1.marks));
    printf(&#34;Using sizeof() operator = %I64d bytes\n&#34;,sizeof(stu1));
}

// 输出结果为
Sum of the size of members = 45 bytes
Using sizeof() operator = 48 bytes
```

可以看到两个结果并不相等，可以看出实际被多分配了3个字节，需要知道为什么被多分配需要先打印他们的地址

```c
#include &lt;stdio.h&gt;
struct student
{
    char name[20];
    int roll;
    char gender;
    int marks[5];
} stu1;

void main()
{
    printf(&#34;Address of member name = %d\n&#34;, &amp;stu1.name);
    printf(&#34;Address of member roll = %d\n&#34;, &amp;stu1.roll);
    printf(&#34;Address of member gender = %d\n&#34;, &amp;stu1.gender);
    printf(&#34;Address of member marks = %d\n&#34;, &amp;stu1.marks);
}
```

输出结果为

```bash
Address of member name = 4225408
Address of member roll = 4225428
Address of member gender = 4225432
Address of member marks = 4225436
```

可以看到char类型占用一个字节，而接下来的成员 marks 却是从4225436开始的，而不是4225433。这就需要引入下面的概念数据对齐 (***Data alignment***)

#### 数据对齐

数据对齐是指处理器在数据对齐时访问效率最高，这将代表了数据存储在内存中的大小的倍数。而现代计算机字长通常为4 字节（32 位操作胸痛）或 8 字节（64 位操作系统）的字长。

对于一个int类型的变量，占用的资产时4字节，此时符合处理器读取机制，因为符合计算机字长长度。而作为char类型，占用一个字节。如果不做数据对齐操作，就会出现如下图出现的问题，数据在存储时读取的字长永远是多一个步骤的。

下图是一个错位的数据，粉红代表char类型，蓝色代表short类型，绿色代表int类型，如果不进行对齐，再继续存储int时，在读取数据时一个字长位移都将不足以读取一个int类型，这就需要进行两次数据访问才能读取一个int类型，也就是花费了两倍的时间

![image-20220930234237287](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220930234237287.png)

&lt;center&gt;图：Misaligned memory&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://hps.vi4io.org/_media/teaching/wintersemester_2013_2014/epc-14-haase-svenhendrik-alignmentinc-paper.pdf&lt;/center&gt;&lt;br&gt;

出于上述原因才有了数据对齐的概念，下图所示的对齐模式被称为自然对齐 (***naturally aligned***)

![image-20220930234625752](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/image-20220930234625752.png)

&lt;center&gt;图：Properly aligned memory using padding&lt;/center&gt;
&lt;center&gt;&lt;em&gt;Source：&lt;/em&gt;https://hps.vi4io.org/_media/teaching/wintersemester_2013_2014/epc-14-haase-svenhendrik-alignmentinc-paper.pdf&lt;/center&gt;&lt;br&gt;

#### 内容填充

在对齐时所插入的额外字节数的部分被称为填充 (***padding***)，在上图中，黑色部分为填充的部分，而在上述代码示例中所填充的部分为3字节，而4225433位的内存地址存储int类型（marks[0]的地址）不是4的倍数。

下表说明了需要对其的数据类型规则

| 数据类型                 | 占字节数大小 | 地址倍数 |
| :----------------------- | :----------- | :------- |
| char                     | 1            | 1的倍数  |
| short                    | 2            | 2的倍数  |
| int, float               | 4            | 4的倍数  |
| double, long, *(pointer) | 8            | 8的倍数  |
| long double              | 16           | 16的倍数 |

另外一个示例，应该是多少？

```c
#include &lt;stdio.h&gt;
struct student
{
    int i1;
    double d1;
    char c1;
} stu1;

void main()
{   
    // long long int a, b, c;
    // a = 1, b = 30000000000009, c = 5; 
    // %I64d是微软风格的%lld，为了避免大于4字节的类型被省略，而输出异常，兼容%d
    // printf(&#34;%I64d %I64d %I64d\n&#34;, a, b, c);
    // rintf(&#34;%d %d %d\n&#34;, a, b, c);
    printf(&#34;size = %I64d bytes\n&#34;,sizeof(stu1));
}
```

由于 `i1` 为int类型4字节，`d1` 为 double类型8字节，`c1` 为char类型1字节 ，那么 $4&#43;8&#43;1=13$ 被填充后应该是16字节，那么看下输出结果

```
size = 24 bytes
```

实际上在C语言中结构体的数据类型对齐不是这么计算的，实际上结构体数据对齐条件是根据结构体内最大的元素进行调整 &lt;sup&gt;&lt;a href=&#34;#2&#34;&gt;[2]&lt;/a&gt;&lt;/sup&gt;，例如这里最大元素为8，那么对齐标准就是补足8字节  `i1` 需要补4，`c1` 需要补7

通过调整结构体顺序可以减少填充的大小，例如下列代码，其实际大小为1 byte &#43; 8 bytes &#43; 1 bytes = 10 bytes，而实际大小为24bytes，因为double将影响填充的大小

```
struct Foo {
    char x; // 1 byte
    double y // 8 bytes
    char z; // 1 bytes
};
```

而通过按照类型的由小到大的顺序进行定义成员，可以减少填充的次数与大小，这样1&#43;1&#43;(6)&#43;8=16

```
struct Foo {
    char x; // 1 byte
    char z; // 1 bytes
    double y // 8 bytes
};
```

为此得出的结论为，对结构体成员重新排序可以提高内存效率

#### 数据打包

数据打包 (***Packing***) 是指强制编译器不进行数据填充，与数据填充是相反的作用

在windows上使用宏定义 `#pragma pack(1)` 来指定对齐方式，也可以使用 `__attribute__((packed)) ` 指定一个结构体补进行填充。

```c
#include &lt;stdio.h&gt;
// #pragma pack(1)
struct student
{
    int i1;
    double d1;
    char c1;
} stu1;

struct Foo {
    char x; // 1 byte
    char z; // 1 bytes
    double y // 8 bytes
} __attribute__((packed)) f1;

void main()
{   
    printf(&#34;size = %I64d bytes\n&#34;,sizeof(stu1));
    printf(&#34;size = %I64d bytes\n&#34;,sizeof(f1));
}
```

输出结果为 

```bash
size = 24 bytes
size = 10 bytes
```

还可以指定特定的大小进行填充，例如

```c
#include &lt;stdio.h&gt;
// #pragma pack(1)
struct student
{
    int i1;
    double d1;
    char c1;
} stu1;

struct Foo {
    char x; // 1 byte
    char z; // 1 bytes
    double y // 8 bytes
} __attribute__((packed, aligned(4))) f1;

void main()
{   
    printf(&#34;size = %I64d bytes\n&#34;,sizeof(stu1));
    printf(&#34;size = %I64d bytes\n&#34;,sizeof(f1));
}
```

输出结果为

```c
size = 24 bytes
size = 12 bytes
```

### 指针结构体

这里包含指针作为结构体成员和指针指向结构体

```c
#include &lt;stdio.h&gt;
struct student
{
    char *name;
    int *roll;
    char gender;
    int marks[5];
};

void main()
{  
    int alexRoll = 44;
    struct student stu1 = { &#34;Alex&#34;, &amp;alexRoll, &#39;M&#39;, { 76, 78, 56, 98, 92 }};
    struct student *stu2 = &amp;stu1;
    printf(&#34;stu1 Name is %s\n&#34;, stu1.name);
    
    // 无效的访问
    // printf(&#34;stu1 roll is %s\n&#34;, stu1.(*roll)); 
    
    // 错误的访问，输出的地址
    printf(&#34;stu1 roll is %d\n&#34;, stu1.roll);
    
    // 正确的访问方式
    printf(&#34;stu1 roll is %d\n&#34;, *(stu1.roll));
    
    // 访问指针结构体成员的方法
    printf(&#34;stu2 Name is %s\n&#34;, stu2-&gt;name);
    printf(&#34;stu2 Name is %s\n&#34;, (*stu2).name);

}
```

总结：

- `.` 运算符优先于 `*` 运算符，需要加括号改变优先级

- 如果成员属性是指针类型，访问其内容应先解引用成员 `*(stu1.roll)` 
- 如果指针是结构体需要解引用结构体 `(*stu2).name`
- 指针类型访问成员的特殊方法为 `-&gt;`

### 结构体数组

结构体也可以作为数组的形式，每个数组元素为一个结构体。作为数组结构体时，指针类型需要解引用或者使用 `-&gt;` 来访问。

```c
#include &lt;stdio.h&gt;
struct student
{
    char *name;
    int *roll;
    char gender;
    int marks[5];
};

void main()
{  
    int alexRoll = 44;
    struct student stu1 = { &#34;Alex&#34;, &amp;alexRoll, &#39;M&#39;, { 76, 78, 56, 98, 92 }};
    
    struct student stu[10];
    struct student *stuPtr = stu;
    struct student (*stuPtr)[10] = &amp;stu;
    
    printf(&#34;name %s\n&#34;, stuPtr[10]-&gt;name);
    printf(&#34;name %s\n&#34;, (*stuPtr)[10].name);
}   
```

### 结构体函数

在C语言中，函数不能作为结构体成员，但是函数指针可以，使用 `.` 可以调用指针函数成员

```c
#include &lt;stdio.h&gt;
struct example
{
    int i;
    void (*ptrMessage)(int i);
};

void message(int);

void message(int i)
{
    printf(&#34;Hello, I&#39;m a member of a structure. This structure also has an integer with value %d&#34;, i);
}

void main()
{
    struct example eg1 = {6, message};
    eg1.ptrMessage(eg1.i);
}
```

### 结构体作为函数参数

当函数参数过多时，传递大量参数效率很低，可以将结构体作为参数传递给函数

```c
#include &lt;stdio.h&gt;
struct student {
    char name[20];
    int roll;
    char gender;
    int marks[5];
};
void display(struct student a)
{
    printf(&#34;Name: %s\n&#34;, a.name);
    printf(&#34;Roll: %d\n&#34;, a.roll);
    printf(&#34;Gender: %c\n&#34;, a.gender);

    for(int i = 0; i &lt; 5; i&#43;&#43;)
        printf(&#34;Marks in %dth subject: %d\n&#34;,i,a.marks[i]);
}
void main()
{
    struct student stu1 = {&#34;Alex&#34;, 43, &#39;M&#39;, {76, 98, 68, 87, 93}};
    display(stu1);
}
```

如果结构体比较复杂，传递副本参数效率不高，也可以传递指针结构体作为函数参数

```c
#include &lt;stdio.h&gt;
struct student {
    char name[20];
    int roll;
    char gender;
    int marks[5];
};
void display(struct student *a)
{
    printf(&#34;Name: %s\n&#34;, a-&gt;name);
    printf(&#34;Roll: %d\n&#34;, a-&gt;roll);
    printf(&#34;Gender: %c\n&#34;, a-&gt;gender);

    for(int i = 0; i &lt; 5; i&#43;&#43;)
        printf(&#34;Marks in %dth subject: %d\n&#34;,i,a-&gt;marks[i]);
}
void main()
{
    struct student stu1 = {&#34;Alex&#34;, 43, &#39;M&#39;, {76, 98, 68, 87, 93}};
    struct student *stuPtr = &amp;stu1;
    display(stuPtr);
}
```

### 结构体作为函数返回值

结构体可以作为函数的返回值

```c
#include &lt;stdio.h&gt;
struct student {
    char name[20];
    int roll;
    char gender;
    int marks[5];
};

struct student increaseBy5(struct student p)
{
    for( int i =0; i &lt; 5; i&#43;&#43;)
        if(p.marks[i] &#43; 5 &lt;= 100)
        {
            p.marks[i]&#43;=5;
        }
    return p;
}

void main()
{
    struct student stu1 = {&#34;Alex&#34;, 43, &#39;M&#39;, {76, 98, 68, 87, 93}};
    stu1 = increaseBy5(stu1);
    
    printf(&#34;Name: %s\n&#34;, stu1.name);
    printf(&#34;Roll: %d\n&#34;, stu1.roll);
    printf(&#34;Gender: %c\n&#34;, stu1.gender);

    for(int i = 0; i &lt; 5; i&#43;&#43;)
        printf(&#34;Marks in %dth subject: %d\n&#34;,i,stu1.marks[i]);
}
```

当然如果结构体交复杂，也可以用结构体指针作为返回值

```c
#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
struct student {
    char name[20];
    int roll;
    char gender;
    int marks[5];
};

struct student* increaseBy5(struct student *p)
{   
    for( int i =0; i &lt; 5; i&#43;&#43;)
        if(p-&gt;marks[i] &#43; 5 &lt;= 100)
        {
            p-&gt;marks[i]&#43;=5;
        }
    return p;
}

void main()
{	
    struct student stu1 = {&#34;Alex&#34;, 43, &#39;M&#39;, {76, 98, 68, 87, 93}};
    struct student *stuptr = (struct student *) malloc(sizeof(struct student));
    stuptr = &amp;stu1;
    stuptr = increaseBy5(stuptr);
    
    printf(&#34;Name: %s\n&#34;, stuptr-&gt;name);
    printf(&#34;Roll: %d\n&#34;, stuptr-&gt;roll);
    printf(&#34;Gender: %c\n&#34;, stuptr-&gt;gender);

    for(int i = 0; i &lt; 5; i&#43;&#43;)
        printf(&#34;Marks in %dth subject: %d\n&#34;,i,stuptr-&gt;marks[i]);
}
```

### typedef

***typedef*** 是C语言中的关键字，功能是为现有数据类型分配别名，例如为long类型声明一个别名

```c
typedef long int64
```

也可以在结构体中使用

```c
#include &lt;stdio.h&gt;
#include &lt;stdlib.h&gt;
struct employee
{
    char *name;
    int salary;
} emp;

typedef struct employee1
{
    char *name;
    int salary;
}emp1;

void main()
{	
    // 不使用typedef定义的结构体,在使用时需要加关键字struct
    struct employee e1;
    e1.salary=10;
    e1.name=&#34;zhangsan&#34;;
	// 使用typedef定义结构体,在使用时，可以直接使用结构体名称
    emp1 e2;
    e2.salary=100;
    e2.name=&#34;lisi&#34;;
    printf(&#34;name1: %s\n&#34; ,e1.name);
    printf(&#34;salary1: %d\n&#34; ,e1.salary);
    printf(&#34;name2: %d\n&#34; ,*(e2.name));
    printf(&#34;salary2: %d\n&#34; ,e2.salary);
}
```

typedef主要功能：

- 别名，简化结构体类型struct关键字

- 区分数据类型

    - ```c
        char * p1,p2; // 声明两个变量 p1为char指针类型，p2为char类型
        ```

    - ```c
        typedef char* charPtr;
        charPtr p1,p2; // 声明两个变量为char*类型
        ```

- 提高代码的可移植性

    - ```c
        typedef long long int64; 
        typedef long long int32; // 在大量别名情况下无需每个替换
        int64 a=10;
        int64 b=20;
        ```

## union

union是类似于结构体的一种用户自定义类型，与结构体最大的区别是，结构体是存储一系列元素的联合体，而union是多个成员，仅有一个元素能被存储。

### 定义

定义union语法：

- **union** &lt;*attr-spec-seq*(optional)&gt; &lt;*name*(optional)&gt; **{** *struct-declaration-list* **}** &lt;union var,...&gt;
- **union** &lt;*attr-spec-seq*(optional)&gt; *name*

```c
union car
{
  char name[50];
  int price;
};
```

定义union不会被分配内存，如果要分配内存则需要创建变量使用它

### 访问

```c
#include &lt;stdio.h&gt;
union Job {
   float salary;
   int workerNo;
} j;

int main() {
   j.salary = 12.3;

   // 当对j.workerNo成员分配了值
   // j.salary持有的12.3将不再拥有
   j.workerNo = 100;

   printf(&#34;Salary = %.1f\n&#34;, j.salary);
   printf(&#34;Number of workers = %d&#34;, j.workerNo);
   return 0;
}
```

总结

- union是用户自定义的数据类型
- union中成员都是相同的内存地址
- union保存的内容仅为最近一次赋值的元素的值（哪个元素被赋值，哪个元素被激活）
- union大小为其占用空间最大的那个成员

## enum

枚举(***enumeration***)是C语言中特殊的数据类型，通常是包含具有共同性数据的集合，例如性别，男，女

```c
enum gender {
    MALE,
    FEMALE
};
```

### 声明

常用的有两种方式来使用枚举类型

```c
enum textEditor {
    BOLD,
    ITALIC,
    UNDERLINE
} feature;
```

与

```c
enum textEditor {
    BOLD,
    ITALIC,
    UNDERLINE
};
int main() {
    enum textEditor feature;
    return 0;
}
```

### 赋值

```c
enum textEditor {
    BOLD,
    ITALIC,
    UNDERLINE
} feature;

int main() {
    // Initializing enum variable
    enum textEditor feature = BOLD;
    printf(&#34;Selected feature is %d\n&#34;, feature);

    // Initializing enum with integer equivalent
    feature = 5;
    printf(&#34;Selected feature is %d\n&#34;, feature);

    return 0;
}
```

总结：

- 枚举是整数常量类型
- 枚举包含的元素即为对应变量可以拥有的值
- 因为是整数常量，可以转换为char，bool等
- 枚举的元素结尾是逗号，最后一个元素没有符号；结构体的元素结尾为分号



&gt; **Reference**
&gt;
&gt; &lt;sup id=&#34;1&#34;&gt;[1]&lt;/sup&gt; [structured data types in c explained](https://www.freecodecamp.org/news/structured-data-types-in-c-explained)
&gt;
&gt; &lt;sup id=&#34;2&#34;&gt;[2]&lt;/sup&gt; [Alignment in C](https://hps.vi4io.org/_media/teaching/wintersemester_2013_2014/epc-14-haase-svenhendrik-alignmentinc-paper.pdf)

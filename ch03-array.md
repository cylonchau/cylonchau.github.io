# ch03 array

[toc]

## Array <sup><a href="#1">[1]</a></sup>

- 数组是由单个元素组成的一组数据类型的变量
- 数组的元素存储在连续的内存位置
- 声明数组时应提及数组的大小
- 数组的计数从0开始
- 数组为一位数组与多维数组
- 数组首元素的地址与数组地址相同
- 数组包含  int, float, char, double 数据类型

### Declaration and Initialization 

| 表达式                                                       | 说明                                                       |
| ------------------------------------------------------------ | ---------------------------------------------------------- |
| int my_array1[20];                                           | 指定大小，来声明一个有20个元素的int数组                    |
| char my_array2[5];                                           | 指定大小，来声明一个有5个元素的char数组                    |
| int my_array[] = {100, 200, 300, 400, 500}                   | 声明时初始化一个数组（编译器自动求数组元素个数）           |
| int my_array1[5] = {100, 200, 300, 400, 500};                | 声明时初始化                                               |
| int my_array2[5] = {100, 200, 300};                          | 声明时初始化（剩余未初始化的元素，默认 0 值）              |
| int my_array2[5] = {0};                                      | 声明时初始化（声明一个全0值的数组）                        |
| int arr[10]; <br/>arr[0] = 5;<br/>arr[1] = 6;<br/>arr[2] = 7; | 声明数组并初始化值（这种方法为初始化部分的默认值为随机数） |
| char str[] = "zhangsan"                                      | 声明一个字符串（字符串是一个char类型数组）                 |

### Advantages and Disadvantages

缺点：**大小限制**：声明（定义）后是固定的大小，不能通过运行时改变其大小

优点：

- 代码优化，可以通过数组更好的对数据进行检索或排序
- 随机存储，可以将数据存储在不同的位置

### muitl-dimensional <sup><a href="#5">[5]</a></sup>

数组中的数组，又称为多维数组(*** multidimensional arrays***)。包含 2D 3D数组。2D是包含行(***rows***), 列(***columns***) 的数组；而3D数组是在2D的基础上，增加了一个维度。包含如下：

- 第一个维度：大小
- 第二个维度：二维数组的行
- 第三个维度：二维数组的列

而更高维度的数组，实际上就是在3D, 4D... 上再增加一个维度。

#### declare 

声明一个多维数组方式如下，声明一个二维数组

```c
float x[3][4];
```

#### Initialization 

| 初始化方式           | 说明                           | 代码                                                         |
| -------------------- | ------------------------------ | ------------------------------------------------------------ |
| 常规初始化           |                                | int arr\[3][5] = {<br/>    {2, 3, 54, 56, 7 },<br/>    {2, 67, 4, 35, 9},<br/>    {1, 4, 16, 3, 78}<br/>}; |
| 不完全初始化         | 未被初始化的数值为 **0**       | int arr\[3][5] = { <br/>    {2, 3}, <br/>    {2, 67, 4, }, <br/>    {1, 4, 16, 78}<br/>}; |
|                      | 初始化一个 初值全为0的二维数组 | int arr\[3][5] = {0};                                        |
|                      | 系统自动分配行列               | int arr\[3][5] = {2, 3, 2, 67, 4, 1, 4, 16, 78};             |
| 不完全指定行列初始化 | 二维数组定义必须指定列值       | int arr\[][] = {1, 3, 4, 6, 7};（==错误示例==）              |
|                      | 二维数组定义可以不指定行值     | int arr\[][2] = { 1, 3, 4, 6, 7 };                           |

示例：遍历一个二维数组

```c
#include <math.h>
#include <time.h>
#include <stdio.h>

void main()
{
    int row,colume;
	int arr[][2] = {1, 3, 4, 6, 7, 10};
    row = sizeof(arr)/ sizeof(arr[0]);
    colume =  sizeof(arr[0])/ sizeof(arr[0][0]);
    
    for(int i=0;i<row;i++)
    {
        for(int j=0;j<colume;j++)
        {
            printf("%d ", arr[i][j]);
        } 
        printf("\n");
    }
}
```

声明和便利一个三维数组

```c
#include <stdio.h>

void main()
{
    int i, j, k;
    int arr[3][3][3]= {
        {
            {11, 12, 13},
            {14, 15, 16},
            {17, 18, 19}
        },
        {
            {21, 22, 23},
            {24, 25, 26},
            {27, 28, 29}
        },
        {
            {31, 32, 33},
            {34, 35, 36},
            {37, 38, 39}
        },
	};
    printf(":::3D Array Elements:::\n");
    for(i=0;i<3;i++)
    {
        for(j=0;j<3;j++)
        {
            for(k=0;k<3;k++)
            {
            printf("%d\t",arr[i][j][k]);
            }
            printf("\n");
        }
        printf("\n");
    }
}
```

## String <sup><a href="#3">[3]</a></sup>

字符串 (***string***) 是一组字符 (***char***)，以 ”\0“ 结尾，抽象来说，C语言中字符串就是数组类型的char

定义，定义一个值为”colour“的字符串。

```c
char message[6] = {'C', 'o', 'l', 'o', 'u', 'r', '\0'};
```

而 "\0" 可以省略，定义可以如下

```c
char message[]= “Colour”; 
```

**C语言初始化字符串的4中方法**

| 表达式                                      | 说明                             |
| ------------------------------------------- | -------------------------------- |
| char str[] = "hello world";                 | 分配不带大小的字符串             |
| char str[50] = "hello world";               | 分配具有预定义大小的字符串       |
| char str[14] = { 'h','e','l','l','o','\0'}; | 按字符分配大小的字符串           |
| char str[] = { 'h','e','l','l','o','\0'};   | 不带大小的按字符分配大小的字符串 |

在C语言中，数组和字符串都是二等公民，一旦声明后，不支持赋值运算符

```c
#include<stdio.h>
int main()
{
    char message[6] = {'C', 'o', 'l', 'o', 'u', 'r'};
    message = "aaaaaa";
    printf("sum = %c\n", message);
  
    return 0;
}
```

上面代码对字符串二次赋值，这种编译器直接报错

```bash
1.c: In function 'main':
1.c:6:13: error: assignment to expression with array type
     message = "aaaaaa";
```

> Notes：复制字符串可以使用函数 strcpy() 

#### 字符串获取

- **scanf**：

    - 存储字符串的空间必须足够大，防止溢出。

    - 获取字符串，`%s`， 遇到 空格 和 ***\n*** 终止。

    - 使用“正则表达式”可以获取带有空格的字符串，如：`scanf("%[^\n]", str);`

- **gets**：类似与 ***scanf*** ；从stdin中读取字符串保存在变量中，遇到换行符终止（可以获取带有“空格”的字符串）。

    - 参数：用来存储字符串的空间地址
    - 返回值：返回实际获取到的字符串首地址。

- **fgets**：从指定流读取一行字符串，遇到换行符或到达结尾终止

    - *str：存储读取字符串的变量指针。
    - n：读取的最大字符
    - *stream：输入流的对象指针，如stdin

#### 字符串写入

- **puts**：将一行字符串写入输出流 (***stdout***)， 输出字符串后会自动添加 \n 换行符。
- `char* str`：被打印的字符串
  
- ***return value***：成功返回非0的integer，失败返回  `EOF` 
- **fputs**：将字符串写入指定流，不包含换行符 `\n`
    - `const char *str`：写入的以NULL字符结尾的字符串
    - `FILE *stream`： FILE 对象的指针，代表要将字符串写入的流
    - ***return value***：成功返回非0的integer，失败返回  `EOF` 


### Array VS String <sup><a href="#2">[2]</a></sup>

- **数据类型不同**：数组可以保存 int, float, doubles类型，字符串只能保存char类型
- **长度不同**：数组长度是固定的，字符串长度可变（通过指针）
- **数据结构不同**：数组可以是一维或多维，字符串是一维数组，结束是一个空字符 ”\0"

### char * VS char []

| char a[10]         | char *a                                                      |
| ------------------ | ------------------------------------------------------------ |
| a是一个数组        | a是一个指针                                                  |
| sizeof为数组的大小 | sizeof为指针类型的大小                                       |
| 存储在内存中的栈段 | a的地址被存储在栈中，但是内容被存储在.rodata中               |
| a不可以被修改      | a可以被修改                                                  |
| a[0]可以被修改     | a[0]不可以被修改，因为内容在.rodata                          |
|                    | char *a="text"; <br>\*a 存储的 text 内容，只读区内容不能修改 <br/>a 代表存储的 .rodata的地址 <br/>a="text1" text1位于内存中其他地方的，将这个地址赋值给a |

### 字符串的拷贝

- 使用指针运算方式

    ```c
    void copy_string02(char* dest, char* src){
    	while (*source != '\0' /* *src != 0 */){
    		*dest = *src;
    		src++;
    		dest++;
    	}
    }
    ```

- 使用数组方式

    ```c
    void copy_string01(char* dest, char* src ){
    	for (int i = 0; src[i] != '\0';i++){
    		dest[i] = src[i];
    	}
    }
    ```

- 使用while循环

    ```c
    void copy_string03(char* dest, char* source){
    	// 判断时赋值结尾 0=0也会退出循环
    	while (*dest++ = *source++){}
    }
    ```

### 字符串的格式化

- 用于将字符串打印在标准输出的：` int printf(const char* str, ...); `
- 用于将字符串格式化打印在缓冲区中的（stdin, stdout, stderr是隐式缓冲资源）：`int fprintf(FILE *fptr, const char *str, ...);`
- 用于格式化而不打印的：`int sprintf(char *str, const char *string,...); `

## array sorting

### 杯子交换

三杯水交换算法 ( ***The Cup Swapping algorithm***) 

有两杯装满水的杯子来代表变量的值，如果需要交换两杯水到对方，就如同交换两个变量的值，此时需要第三个杯子来交换液体，就像第三个变量用作临时存储变量的值一样。

例如，数组的倒序可以使用该方法，也是其他算法中的基础。

```c
#include<stdio.h>
#include<string.h>
int main()
{
	int arr[] = {22,321,56,1,66,23,9,10}; 
    // 数组的长度
	int len = sizeof(arr) / sizeof(arr[0]);
    // 临时变量 
	int tmp;		 

	// 交换 数组元素，做逆序
    for (int i=0;i<len;i++)
	{
        if (i > len/2) {
            break;
        }
		tmp = arr[i]; // 第三杯水
		arr[i] = arr[len-i-1];
		arr[len-i-1] = tmp;
	}
}
```

### 冒泡 <sup><a href="#4">[4]</a></sup>

冒泡排序 (***bubble sort***) 是最简单的排序算法，其核心是==如果两个相邻元素的位置排序不对，就交换相邻的元素==，例如： ***arr[] = {5, 1, 4, 2, 8}*** ，从前两个元素开始比较检查哪个更大

- 第一轮（迭代）：

    - ( **5** **1** 4 2 8 ) -> ( **1** **5** 4 2 8 )，比较前两个元素 5 > 1 交换两个位置。

    - ( 1 **5** **4** 2 8 ) –> ( 1 **4** **5** 2 8 )，5 > 4 交换两个位置
    - ( 1 4 **5** **2** 8 ) –> ( 1 4 **2** **5** 8 )，5 > 2 交换两个位置
    - ( 1 4 2 **5** **8** ) -> ( 1 4 2 **5** **8** )，(8 > 5)，不会交换，至此最后一位排序正确

- 第二轮

    - ( **1** **4** 2 5 8 ) ->  ( **1** **4** 2 5 8 ) 
    - ( 1 **4** **2** 5 8 ) –>  ( 1 **2** **4** 5 8 )，4 > 2 交换两个位置
    - ( 1 2 **4** **5** 8 ) ->  ( 1 2 **4** **5** 8 ) 
    - ( 1 2 4 **5** **8** ) ->  ( 1 2 4 **5** **8** ) 

- 第三轮（没法发生交换）
    - ( **1** **2** 4 5 8 ) -> ( **1** **2** 4 5 8 ) 
    - ( 1 **2** **4** 5 8 ) -> ( 1 **2** **4** 5 8 ) 
    - ( 1 2 **4** **5** 8 ) -> ( 1 2 **4** **5** 8 ) 
    - ( 1 2 4 **5** **8** ) -> ( 1 2 4 **5** **8** ) 

**算法实现**

```c
#include <stdio.h>

// 三杯水交换
void swap(int* x, int* y)
{
    int temp = *x;
    *x = *y;
    *y = temp;
}
 
// 冒泡实现
void bubbleSort(int arr[], int n)
{
    int i, j;
    for (i = 0; i < n - 1; i++)
 
        // Last i elements are already in place
        for (j = 0; j < n - i - 1; j++)
            if (arr[j] > arr[j + 1])
                swap(&arr[j], &arr[j + 1]);
}
 
// 打印数组
void printArray(int arr[], int size)
{
    int i;
    for (i = 0; i < size; i++)
        printf("%d ", arr[i]);
    printf("\n");
}

int main()
{
    int arr[] = { 64, 34, 25, 12, 22, 11, 90 };
    int n = sizeof(arr) / sizeof(arr[0]);
    bubbleSort(arr, n);
    printf("Sorted array: \n");
    printArray(arr, n);
    return 0;
}
```

输出结果为

```c
Sorted array: 
11 12 22 25 34 64 90 
```



> **Reference**
>
> <sup id="1">[1]</sup> [Arrays in c](https://www.cs.uic.edu/~jbell/CourseNotes/C_Programming/Arrays.html)
>
> <sup id="2">[2]</sup> [difference between array and string](https://pediaa.com/what-is-the-difference-between-array-and-string/)
>
> <sup id="3">[3]</sup> [string in c](https://www.tutorialspoint.com/cprogramming/c_strings.htm)
>
> <sup id="4">[4]</sup> [bubble sort with c](https://www.geeksforgeeks.org/bubble-sort/)
>
> <sup id="5">[5]</sup> [3d array in c](https://iq.opengenus.org/3d-array-in-c/)





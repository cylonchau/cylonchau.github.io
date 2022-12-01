# ch08 file handling

[toc]

## 文件类型

文件是指以字节的形式存储的数据源，使用C语言将文件数据以输出输出的形式处理叫做文件处理。

文件在C语言中以两种形式存在：

- 文本文件：文本文件是简单的文件类型，这些文件内容以 ASCII 字符格式存储信息。
- 二进制文件：二进制文件以 0 和 1 的二进制格式存储数据，不是人类可读的文件

## 文件指针

文件指针 (`FILE`) 是一种数据类型，是被定义在 `stdio.h` 中的一种结构体，包含了文件的一些信息

```c
typedef struct
{
    // fill/empty level of buffer
    int    level;
    // File status flags
    unsigned flags;
    // File descripter
    char fd;
    // ungetc char if no buffer
    unsigned char hold;
    // buffer size
    int bsize;
    // data transfer buffer
    unsigned char *buffer;
    // Current active pointer
    unsigned char *curp;
    //Temporary file indicator
    unsigned istemp;
    //Used for validity checking
    short token;
} FILE; // This is FILE object    
```

文件指针通常被用于处理正在访问的文件，`fopen()` 是用于打开文件并返回文件的 FILE 指针，而后通过文件只恨进行I/O操作。`fopen()` 会发生下列事件：

- 文件的内容被加载到缓冲区（操作系统层面）
- 在内存中创建 FILE 的数据结构体，并返回这个结构体指针

## 文件处理函数

| 函数                                 | 功能 |
| ------------------------------------ | --------- |
| fopen()      | 打开现有文件或新文件 |
| fprintf()          | 将数据写入打开的文件       |
| fscanf()         | 读取文件中数据 |
| fputc()        | 向文件写入一个字符         |
| fgetc()       | 从文件中读取一个字符       |
| fclose()             | 关闭打开的文件 |
| fseek() | 设置文件指针的位置 |
| fputw()            | 将一个整数写入到文件       |
| fgetw()      | 从文件中读取一个整数 |
| ftell()        | 文件指针的当前位置 |
| rewind() | 设置文件指针位置为初始位置 |
| fread() | 读取文件内容（二进制与文本） |
| fwrite() | 向文件写入内容（二进制与文本） |
| feof() | 是否到达文件结尾<br>非0 True 到达文件结尾<br>0 False 没有到达文件结尾 |

### fscanf VS fgets

- fscanf读取的是字符，fgets读取的是字符串
- fgets读取换行符结束，fscanf读取到空白就结束，不用换行符
- fgets以行为单位，fscanf以字符为单位（参数2匹配的模式）
- fscanf每次会判断是否匹配，如不匹配则提前退出读取

```c
#include <stdio.h>
#include <stdlib.h>

void main()
{	
    FILE *fp;
    fp = fopen("./k8s restart.txt", "r");
    if (fp == NULL) {
        printf("Error! opening file");
        exit(1);
    }
    char buf[1024];

    while( feof(fp) == 0 ){
        printf("i=%d\n", i);
        
        // fscanf(fp, "%s", &buf);
        fgets(buf, 1024, fp);
        printf("%s\n", buf);
    }
    fclose(fp);
}
```

### 文件的打开模式

| 模式 |含义|当文件不存在时处理方法|
| ---- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| r    | 只读方式打开文件    | 当文件路径不存在时fopen()返回NULL          |
| rb   | 以只读方式打开二进制文件 | 当文件路径不存在时fopen()返回NULL |
| w    | 写入方式                               | 如果文件存在则覆盖，如果不存在则创建新文件 |
| wb   | 写入方式（二进制模式） | 如果文件存在则覆盖，如果不存在则创建新文件 |
| a    | 打开文件并向结尾追加内容 | 如果文件路径不存在则创建新文件 |
| ab   | 打开文件并向结尾追加内容（二进制模式） | 如果文件路径不存在则创建新文件             |
| r+   | 读写方式打开文件                       | 当文件路径不存在时fopen()返回NULL |
| rb+  | 读写方式打开文件（二进制模式） | 当文件路径不存在时fopen()返回NULL |
| w+   | 读写方式打开文件            | 如果文件存在则覆盖，如果不存在则创建新文件 |
| wb+  | 读写方式打开文件（二进制模式） | 如果文件存在则覆盖，如果不存在则创建新文件 |
| a+   | 追加和读取 | 如果文件路径不存在则创建新文件 |
| ab+  | 追加和读取（二进制模式） | 如果文件路径不存在则创建新文件 |

## 文件操作的步骤

1. 打开文件 `fopen()` （ FILE *Pointer）

2. 读写文件 `fputc` , `fgetc`, `fputs`, `fgets`, `fread`, `fwrite` ....

3. 关闭文件 fclose()  

### 打开文件

```c
FILE * ptr
ptr = fopen("file dir","mode");
```

例如

```c
fopen("/etc/hosts","rb");
```

### 关闭文件

```c
fclose(fptr);
```

### 读/写文件

向文本文件中写入数据

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
   int num;
   FILE *fptr; 
   fptr = fopen("~/1.txt","w");

   if(fptr == NULL)
   {
      printf("Error!");   
      exit(1);             
   }

   printf("Enter num: ");
   scanf("%d",&num);
   
   fputc(str[i], fptr); // fputc向文件写入数据
   fputs("fputs向文件写入数据\n", fptr);
   fprintf(fptr, "fprintf向文件写入数据\n");
   fclose(fptr);

   return 0;
}
```

从文本文件中读取内容

```c
#include <stdio.h>
#include <stdlib.h>

int main()
{
   int num;
   FILE *fptr;

   if ((fptr = fopen("~/1.txt","r")) == NULL){
       printf("Error! opening file");
       exit(1);
   }

   fscanf(fptr,"%d", &num);
   printf("Value of n=%d", num);
   fclose(fptr); 
  
   return 0;
}
```

### 写入二进制文件

二进制文件的读取使用，`fwrite()`/`fread()`函数，通常情况下二进制文件读取没有意义，只是做类似文件拷贝的操作。

`fwrite(addressData, sizeData, numbersData, pointerToFile);`

- addressData：写入磁盘的数据的地址
- sizeData：要写入磁盘的数据大小
- numbersData：写出的数据个数
- pointerToFile：FILE指针
- return：
    - 成功：参数3的大小
    - 失败：0

> Notes：通常参数2为1，参数3为写入的总大小。 参2 * 参3 = 写入的总大小

```c
#include <stdio.h>
#include <stdlib.h>

struct threeNum
{
   int n1, n2, n3;
};

int main()
{
   int n;
   struct threeNum num;
   FILE *fptr;

   if ((fptr = fopen("C:\\program.bin","wb")) == NULL){
       printf("Error! opening file");

       // Program exits if the file pointer returns NULL.
       exit(1);
   }

   for(n = 1; n < 5; ++n)
   {
      num.n1 = n;
      num.n2 = 5*n;
      num.n3 = 5*n + 1;
      fwrite(&num, sizeof(struct threeNum), 1, fptr); 
   }
   fclose(fptr); 
  
   return 0;
}
```

### 从二进制文件读取数据

`fread(addressData, sizeData, numbersData, pointerToFile);`：

- addressData：读取到的数据存储的位置
- sizeData：一次读取的字节数
- numbersData：读取次数
- pointerToFile：文件指针
- return：
    - 成功：参数3的大小
    - 失败：0
    - 到达文件结尾：feof(fp)为真

```c
#include <stdio.h>
#include <stdlib.h>

struct threeNum
{
   int n1, n2, n3;
};

int main()
{
   int n;
   struct threeNum num;
   FILE *fptr;

   if ((fptr = fopen("C:\\program.bin","rb")) == NULL){
       printf("Error! opening file");

       // Program exits if the file pointer returns NULL.
       exit(1);
   }

   for(n = 1; n < 5; ++n)
   {
      fread(&num, sizeof(struct threeNum), 1, fptr); 
      printf("n1: %d\tn2: %d\tn3: %d\n", num.n1, num.n2, num.n3);
   }
   fclose(fptr); 
  
   return 0;
}
```

## 缓冲区

缓冲区是操作系统的内存空间中的一部分。操作系统在内存空间中预留了一定的存储空间，在输入或输出达到一定量后进行I/O操作，这部分空间就叫做缓冲区。

程序在启动时，预定义了三种缓冲区，不需要显式开启：

- 标准输入 (***stdin***)：
- 标准输出 (***stdout***)：
- 标准错误 (***stderr***)：标准错误是一个无缓冲

stdio.h 库中提供了三种缓冲模式 <sup><a href="#1">[1]</a></sup>： 

- 无缓冲 (***unbuffered***)：写入到无缓冲的数据会立即被写入到文件
- 行缓冲 (***line buffered***)：当遇到换行符，此类缓冲区内容会被写入到文件
- 全缓冲 (***fully buffered***)：缓冲区满或以任意大小的块被写入到文件

可以通过库函数`setvbuf()`, `setbuffer()`, `setbuf()` 三者之一设置 `stdio` 的缓冲模式，例如

```c
#define BUF_SIZE 4096
static char buf[BUF_SIZE];
FILE *fp;

fp = fopen("test.txt", 'w');
if(setvbuf(fp, buf, _IOFBF, BUF_SIZE) !=0 )
    exit(EXIT_FAILURE);
```

可以使用库函数 `fflush()` 手动刷新缓冲区  <sup><a href="#2">[2]</a></sup>，例如

```c
fp = fopen("test.txt", 'w');

fputs("fputs向文件写入数据\n", fptr);
fflush(fp);
fputs("fputs向文件写入数据\n", fptr);
fflush(fp);
```

如果，全缓冲模式下，缓冲区没满也没刷新，那么只有在文件关闭时， 缓冲区会被自动刷新（写入到文件）

> Tips：内存的隐式回收：关闭文件、刷新缓冲区、释放malloc 



> **Reference**
>
> <sup id="1">[1]</sup> [IO cache](https://www.litreily.top/2018/10/25/io-cache/)
>
> <sup id="2">[2]</sup> [Controlling Buffering](https://www.gnu.org/software/libc/manual/html_node/Controlling-Buffering.html)







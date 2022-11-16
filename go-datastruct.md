# go 数据结构


Go语言将数据类型分为四类：基础类型、复合类型、引用类型和接口类型。

基础数据类型包括：
- 基础类型：
      - 布尔型、整型、浮点型、复数型、字符型、字符串型、错误类型。
- 复合数据类型包括：
      - 指针、数组、切片、字典、通道、结构体、接口。

## 基础数据类型

### 布尔值和布尔表达式

布尔类型的变量取值结果要么是真，要么是假，用`bool`关键字进行定义

布尔类型默认值为 `false`

指定格式的输出 `%t`

| 语法     | 描述/结果                                                    |
| -------- | ------------------------------------------------------------ |
| !b       | 逻辑非操作符 b值为true 则 操作结果为false                    |
| a \|\| b | 短路逻辑或，只要布尔值 a b 中任何一个为true表达式结果都为true |
| a && b   | 短路逻辑与，两个表达式a b都为true，则整个表达式结果为true    |
| x > y    | 表达式x的值小于表达式Y的值，则表达式的结果为true             |

### 数值类型

go语言提供了大内置的数值类型，标准库也提供了big.Int类型的整数，和big.Rat类型的有理数，这些都是大小不限（只限于机器内存）

#### 整型

GO语言提供了11种整型，包含5种有符号，和5种无符号的与一种用于存储指针的整数类型。Go语言允许使用byte来作为无符号uint8类型的同义词，在使用单个字符时提倡使用`rune`来替代 `int32`

| 类型    | 存储空间 | 取值范围                                                     |
| ------- | :------- | ------------------------------------------------------------ |
| byte    | 8-bit    | 同uint8                                                      |
| int     | 系统决定 | 依赖不通平台实现，32位操作系统为`int32`的值范围，64位操作系统为`int64`的值范围 |
| int8    | 8-bit    | [-128, 127] ，表示 UTF-8 字符串的单个字节的值，对应 ASCII 码的字符值 |
| int16   | 16-bit   | [-32678, 32767]                                              |
| int32   | 32-bit   | [2147483648, 2147483647]                                     |
| int64   | 64-bit   | [-9223372036854775808 , 9223372036854775807]                 |
| rune    | 32-bit   | 同uint32，表示 单个 Unicode 字符                             |
| uint    | 系统决定 | 依赖不通平台下的实现，可以是uint32或uint64                   |
| uint8   | 8-bit    |                                                              |
| uint16  | 16-bit   | [0, 65535]                                                   |
| uint32  | 32-bit   | [0, 4294967295]                                              |
| uint64  | 64-bit   | [0, 18446744073709551615]                                    |
| uintptr | 系统决定 | 一个可以恰好容纳指针值得无符号整数类型（32位操作系统为`uint32`的值范围，64位系统为`uint64`的值范围） |

#### 浮点类型

Go语言提供了两种类型的浮点类型和两种类型的复数类型，

| 类型        | 存储空间 | 取值范围                                                     |
| ----------- | -------- | ------------------------------------------------------------ |
| float32     | 32-bit   | [1.401298464324817070923729583289916131280e-45 , 3.402823466385288598117041834516925440e+38] 精确到小数点后7位 |
| float64     | 64-bit   | [4.940656458412465441765687928682213723651e-324 , 1.797693134862315708145274237317043567981e+308]  精确到小数点后15位 |
| complexm64  | 64-bit   | 实部和虚部都是一个float32                                    |
| complexm128 | 128-bit  | 实部和虚部都是一个float64                                    |


```go
fmt.printf("%.2f",num) // 保留两位小数，同时进行了四舍五入
```

#### 字符串 (string)

字符串使用双引号 `"` 或者反引号 ```来创建，双引号可以解析字符串变量

定义字符变量用 `byte` 关键词 `var ch byte = 'a'`  8-bit，代表了 ASCII 码的一个字符。指定格式的输出 `%c`
定义字符变量用 `rune` 关键词 `var ch rune = 'a'`  32-bit  代表了 Unicode（UTF-8）码的一个字符。
定义字符变量用 `string` 关键词 `var ch string = "abc"`指定格式的输出 `%s`

字符串的结束标志 `\0`，Go语言使用的UTF8编码，英文占1个字符，一个汉字占3个字符      

### 自动推导类型

自动推到类型，创建的浮点型默认为 float64，整型为int

```go
      a := "123"
      a := 10
```

### 使用fmt格式化输出

格式指令通常由用于输出单个值，每个值都按格式指令格式化。用于`fmt.Printf()` `fmt.Errorf()`  `fmt.Fprintf()` `fmt.Sprintf()`函数的格式字符串包含一个或多个格式指令

| 格式指令 | 含义/结果                             |
| -------- | ------------------------------------- |
| %b       | 二进制数值                            |
| %c       | 数值对应的 Unicode 编码字符           |
| %d       | 十进制数值                            |
| %o       | 八制数值                              |
| %e       | 科学计数法，e表示                     |
| %E       | 科学计数法，E表示                     |
| %f       | 有小数部分，无指数部分                |
| %g       | 以%f或%e表示浮点数或复数              |
| %G       | 以%f或%E表示浮点数或复数              |
| %s       | 直接输出字符串或者[]byte              |
| %q       | 双引号括起来的字符串或者[]byte        |
| %x       | 每个字节用两字节十六进制表示，a-f表示 |
| %X       | 每个字节用两字节十六进制表示，A-F表示 |
| %t       | 以true或fales输出布尔值               |
| %T       | 输出值得类型                          |
| %v       | 默认格式输出内置或自定义类型的值      |

### 强制类型转换

类型转换用于将一种数据类型转换为另外一种类型

```
var num float64 = 3.15
fmt.Printf("%d", int(num))
```

类型不一致的不能进行运算，在类型转换时，建议将低类型转换为高类型，保证数据精度，高类型转换成地类型，可能会丢失精度，或数据溢出

## 复合数据类型

### 数组

数组，一系列同一类型数据的集合，数组是值类型,在初始化后长度是固定的，无法修改其长度。

#### 数组的定义

`var arr [元素数量]类型`

#### 数组初始化

全部初始化  `var arr [5]int = [5]int{1,2,3,4,5}`

部分初始化 `var arr [5]int = [5]int{1,2}` ,没有指定初值的元素将会赋值为其元素类型(int)的默认值(0)

指定下标初始化 `var arr = [5]int{2:5, 3:6}`  `key:value`的格式

通过初始化确定数组长度，`var arr = [...]int{1, 2, 3}` 长度是根据初始化时指定个数


相同空间大小（类型）的数组可以用 `==` `!=` 来比较是否相同。

```go
package main

import "fmt"

func main() {
	var arr [2]int = [2]int{1, 2}
	var arr2 [2]int = [...]int{2, 2}
	fmt.Println(arr == arr2)
}


D:\go_work\src>go run main.go
false
```

#### 数组的遍历

- 通过 `for .. len()` 完成遍历
- 通过 `for .. range` 完成遍历

#### 作为函数值传递

golang数组是值类型，当数组作为函数参数，函数中修改数组中的值，不会影响原数组

```go
package main

import "fmt"

func test(t [2]int) {
	fmt.Println(&t)
}

func main() {
	var arr [2]int = [2]int{1, 2}
	test(arr)
}
```

#### 二维数组

初始化方式

- 全部初始化 `var arr [2][2]int = [2][2]int{ {1,2},{3,4} }`
- 部分初始化 `var arr [2][2]int = [2][2]int{ {1,2},{3} }`
- 指定元素初始化 `var arr [2][2]int = [2][2]int{ 1:{1} }`，`var arr [2][2]int = [2][2]int{ 1:{1:3} }`
- 通过初始化确定二维数组行数 `var arr [...][2]int = [2][2]int{ {1,2},{3,4} }` 行下标可以用 `...` 列下标不可用 `...`

### map

Go语言中的字典结构是有键和值构成的，所谓的键，就类似于字典的索引，可以快速查询出对应的数据。

map是只用无序的键值对的集合。

map最重要的一点是通过key来快速检索数据，key类似于索引，指向数据的值。

map中key的值是不能重复的

引用类型或包含引用类型的数据类型不能作为key


#### map的创建

- 字面量：`var map_name map[keyType]valType`
- 类型推导： `map_name :=  map[keyType]valType`
- 关键词：`make(map[keyType]valType)`

#### map的初始化

- 字面量：`var maps[int]string = map[int]string{1: "zhangsan", 2: "lisi"}`
- 类型推导： `maps := map[int]string{1: "zhangsan", 2: "lisi"}`
- 关键词：`maps := make(map[string]int,10)`; `maps["zhangsan"] = 14`

#### map的key value

通过key获取值时，判断key是否存在 `var1, var2 := map[key]`，如存在，var1存储对应的值，var2的值为true，var2否则为false

```go
package main

import "fmt"

func initial(t []int) {
	for n := 0; n < 10; n++ {
		t[n] = n
	}
}

func main() {
	var m map[int]string = map[int]string{1: "zhangsan", 2: "lisi"}

	fmt.Println(m)
	v, ok := m[2]

	if ok {
		fmt.Println(v)
	} else {
		fmt.Println(ok)
	}

	v, ok = m[3]

	if ok {
		fmt.Println(v)
	} else {
		fmt.Println(ok)
	}
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004222750318-150497349.png)

`delete(map,2)` 通过key删除某个值

#### 作为函数参数传递

slice  map channel都是引用类型，所以改变是改变的变量的地址。

```go
package main

import "fmt"

func initial(t map[int]string) {
	for n := 0; n < 10; n++ {
		t[n] = fmt.Sprintf("aaa%d", n)
	}
}

func main() {
	var m = make(map[int]string, 10)

	initial(m)
	fmt.Println(m)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004222734115-568397490.png)


### 切片

切片与数组相比，切片的长度是不固定的，可以追加元素，在追加时可能使切片的容量增大，所以可以将切片理解为“动态数组”，但是，它不是数组。

#### 切片定义

`var slice_name []type` 默认空切片，长度为0

`slice_name := []type{}` 默认空切片，长度为0

`make([]type, length, capacity)` length：已初始化的空间，capacity：已开辟的空间（length+空闲）。length不能大于capacity

`len()` 返回长度 `cap()` 返回容量

如果使用字面量的方式创建切片，大部分的工作就都会在编译期间完成，使用 `make()` 关键字创建切片时，很多工作都需要运行时的参与。

#### 切片初始化

通过 `var slice_name []type` 方式创建

```go
import "fmt"

func main() {
	var slices []int

	slices = append(slices, 1, 2, 3, 4, 5, 6)

	fmt.Print(slices)

	slices = append(slices, 100, 99)

	fmt.Print(slices)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004193503175-356412365.png)

通过 `slice_name := []type{}` 方式创建

- 直接在 `{}` 中添加值
- 通过 `append()` 添加

通过 `make([]type, length, capacity)`  方式创建

```go
package main

import "fmt"

func main() {
	var slices []int

	slices = make([]int, 3, 5)

	slices[0] = 1
	slices[1] = 2
	slices[2] = 3
	slices[3] = 4

	fmt.Println(slices)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004193452228-590165965.png)

#### 切片的截取

切片截取

| 操作            | 含义                                                         |
| --------------- | ------------------------------------------------------------ |
| s[n]            | 切片s中索引位置为N的项                                       |
| s[:]            | 从切片s的索引位置到 `len(s) - 1` 处所获得的切片              |
| s[low:]         | 从切片s的索引位置low到 `len(s) - 1` 处所获得的切片           |
| s[:high]        | 从切片s的索引位置0到high处所获得的切片 len=high              |
| s[low:higt]     | 从切片s的索引位置low到high处所获得的切片 len=high-low        |
| s[low:higt:max] | 从切片s的索引位置low到high处所获得的切片，len=high-low，cap=max-low |
| len(s)          | 切片s的长度，<=cap(s)                                        |
| cap(s)          | 切片s的容量，>=len(s)                                        |

`slice[startVal, length, Capacity]` 

容量为：`capacity - startVal`

长度为： `length - startVal`

```go
package main

import "fmt"

func main() {
	var slices []int

	slices = []int{1, 2, 3, 4, 5, 6, 7, 8}
	s := slices[1:3:5]

	fmt.Println(s)
	fmt.Println(len(s))
	fmt.Println(cap(s))
}

```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004201436109-1262934052.png)


在截取时，`capacity` 不能超过原slice的 capacity

```go
package main

import "fmt"

func main() {
	var slices []int

	slices = []int{1, 2, 3, 4, 5, 6, 7, 8}
	s := slices[1:3:10]

	fmt.Println(s)
	fmt.Println(len(s))
	fmt.Println(cap(s))
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004201422642-1224055142.png)


#### 切片值得修改

问：切片截取后返回新切片，对新切片的值修改，会影响原切片吗

当对切片进行截取操作后，产生了新的切片，新的切片是指向原有切片的，对新切片值修改，会影响到原有切片

```go
package main

import "fmt"

func main() {
	var slices []int

	slices = []int{1, 2, 3, 4, 5, 6, 7, 8}
	s := slices[1:3]

	s[1] = 100

	fmt.Println(s)
	fmt.Println(slices)

}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004201354129-1430327244.png)

#### 追加和拷贝

- `append(slice, 1,2,3)` 向切片末尾追加数据
- `copy(slice1, slice2)`  拷贝的长度为两个切片中长度较小的长度值

作为参数值传递，切片是数组的一个引用，因此切片是引用类型，操作会修改原有切片      

```go
package main

import "fmt"

func initial(t []int) {
	for n := 0; n < 10; n++ {
		t[n] = n
	}
}

func main() {
	var slices = make([]int, 10)
	fmt.Println(slices)
	initial(slices)
	fmt.Println(slices)

}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004211110895-1653694753.png)

#### 切片扩容 

切片扩容，一般方式：上一次容量的2倍，超过1024字节，每次扩容上一次的1/4

```go
package main

import "fmt"

func initial(t []int) {
	for n := 0; n < 10; n++ {
		t[n] = n
	}
}

func main() {
	var slices = make([]int, 3)
	var slices1 = []int{4, 5}

	copy(slices, slices1)

	fmt.Println(slices)
	fmt.Println(slices1)

}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004211522893-490726737.png)

### struct 结构体

结构体 `struct` 是由一系列具有相同类型或不同类型的数据构成的数据集合，结构体可以很好的管理一批有联系的数据，使用结构体可以提高程序的易读性。Go中提供了对 `struct` 的支持，与数组一样，**struct属于复合类型，并非引用类型**。

Go语言中结构体包含以下特性

- 值传递，Go语言中结构体和数组一样是值类型，可以声明结构体指针向下传递
- 不可继承，Go语言中没有继承的概念，在结构体中，可以通过组合结构体，来构建更复杂的结构体。
- 结构体不能包含自己。

#### 结构体声明

成员名称前不能加 `var`

```go
type Student struct {
	id int
    name string
    age int
    addr string
}
```

#### 空结构体

结构体也可以不包含任何字段，称为空结构体 `struct{}`

#### 结构体初始化

- 顺序初始化 `var stu = Student{ id:101, name:"zhangsan", age:19, addr:"shanghai" }`
- 指定成员初始化 `var stu = Student{name:"zhangsan", age:19}`
- 通过 `结构体变量.成员` 完成初始化 `var stu Student  stu.age=19 stu.addr="peking"`

#### 结构体传递

结构体与数组一样，都是值传递，将结构体作为函数实参传给函数的形参时，会复制一个副本，所以为了提高性能，在结构体传给函数时，可以使用指针结构体。

指针结构体定义：声明结构体变量时，在结构体类型前加 `*` 号，便声明一个指向结构体的指针。 `var stu *Student`。
指针结构体访问： 由于`.`的优先级高于`*struct_name`,故在使用时，需要对结构体加括号。`(*struct_name).成员属性`  `(*stu).name`

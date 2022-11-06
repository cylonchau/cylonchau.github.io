# go function


## golang保留的函数

`init()`, `main()`是golang的保留函数，有如下特点：

- `main()` 只能用在main包中，仅可定义一个，`init()` 可定义任意包，可重复定义，建议只定义一个
- 两个函数定义时不能有任何返回值
- 只能由go自动调用，不可被引用
- `init()` 先于 `main()` 执行，并不能被其他函数调用，执行时按照main import顺序执行。

### 包的执行顺序

- Go的初始化和执行总是从main.main函数（main包导入其它的包）
- 同包下的不同 `.go` 文件，按照以文件名或包路径名的字符串顺序“**从小到大**”排序顺序执行
- 其他的包只有被`main`包 import 才会执行，按照 import 的先后顺序执行;
- 如果某个包被多次导入的话，在执行的时候只会导入一次;
- 当一个包被导入时，如果它还导入了其它的包，则先将其它的包包含进来;
- 导入顺序与初始化顺序相反 main =&gt; p1 =&gt; p2 | p2 =&gt; p1 =&gt; p
- main被最后一个初始化，因其总是依赖其他包

## 函数

函数是将具有独立功能的代码组织成为一个整体，使其具有特殊功能的代码集。在Go语言中，函数是一种数据类型，其特性有如下：

- 支持匿名函数
- 支持带有变量名的返回值
- 支持多值返回
- 支持匿名函数
- 不支持重载，一个包中不能有两个名字一样的函数。

定义语法

```go
func test(){

}

func test(a int, b int){

}

func test(a,b int){

}

func test(a,b int list...int){

}

func test(a int, b int) int{

}

func test(a int, b int ) (int,int){

}

func test(a,b int) (num int, err error){

}
```

花括号必须与函数声明在同一行，这种写法是错误的

```go
func test()
{

}
```

## 命名返回值名称

```go
package main

import &#34;fmt&#34;

func test(a, b, c int) (he int, cha int) {
	he = a &#43; b &#43; c
	cha = a - b - c
	return
}

func main() {

	a, b := test(15, 10, 5)
	fmt.Println(a)
	fmt.Println(b)

}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004004404068-1876101209.png)


&gt; _标识符，用来忽略返回值

## 函数参数传递方式

\1. 值传递
\2. 引用传递

注意：无论是值传递，还是引用传递，传递给函数的都是变量的副本，不过，值传递是值的持贝。引用传递是地址的持贝，一般来说，地址持贝更为高效。而值持贝取决于拷贝的对象大小，对象越大，则性能越低。

注意2：map、slice、chan、指针、interface默认以引用的方式传递


##  自定义函数类型

```
package main

import &#34;fmt&#34;

type ty_func func(int, int) int

func add(a, b int) int {
	return a &#43; b
}

func operator(op ty_func, a, b int) int {
	return op(a, b)
}

func main() {
	c := add
	sum := operator(c, 100, 200)

	fmt.Println(sum)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004005822762-1019812877.png)


## 不定参数

- 不定参数可以通过下标/循环方式获取参数值
- 不定参数在定义时，固定参数放前面，不定参数放后面
- 在对函数调用时，固定参数必须传值，不定参数可以根据需要来决定是否要传值

语法

```go
func {func_name}({param} ...{type}){
	func_body
}
```

参数的类型是一个 `{type}` 类型的集合

练习：写一个函数add，支持1个或多个int相加，并返回相加结果

```go
package main

import &#34;fmt&#34;

func test(num ...int) {
	var sum int
	for n := 1; n &lt;= len(num); n&#43;&#43; {
		sum &#43;= num[(n - 1)]
	}
	fmt.Println(sum)
}

func main() {
	test(1)
	test(1, 2, 3)
    test(1, 2, 3, 4)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004004529342-1382648432.png)

练习：写一个函数concat，支持1个或多个string相拼接，并返回结果

```go
func concat(age ...string) {
	var str string
	for _,v := range age {
		str &#43;= v
	}

	fmt.Println(str)
}

func main() {
	concat(&#34;hellow&#34;, &#34; world&#34;, &#34; go&#34;)

}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004004636040-1844877115.png)



## 延迟调用defer

- 当函数返回时，执行defer语句。因此，可以用来做资源清理
- 多个defer语句，按LIFO（后进先出）的顺序执行
- defer语句中的变量，在defer声明时就决定了。


用途

- 关闭文件句柄
- 锁资源释放
- 数据库连接释放

defer语句中的变量，在defer声明时就决定了其值

```go
func defer_test() {
	i := 0
	defer fmt.Println(i)

	i = 10

	fmt.Println(i)
}

func main() {
	defer_test()
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004005551020-1793978404.png)


多个defer按LIFO（后进先出）的顺序执行

```
func defer_test() {
	i := 0
	defer fmt.Println(i)

	i = 10

	fmt.Println(i)
}

func main() {
	defer_test()
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004005449280-2056741609.png)


defer的作用域，此处可以看到，defer的传入不是在main的作用域下，测试可发现 `defer`只会在当前函数和方法返回之前被调用。

```GO
package main

import &#34;fmt&#34;

func main() {
   fmt.Println(&#34;block starts&#34;)
   {
      defer fmt.Println(&#34;defer runs&#34;)
      fmt.Println(&#34;block ends&#34;)
   }

   fmt.Println(&#34;main ends&#34;)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004005407717-2125693767.png)


## 函数作用域

全局变量：既能在函数中使用，也能在其他函数中使用，可以称为定义在函数外的变量。

局部变量：定义在函数内部的变量成为局部变量，局部变量的作用域在函数内部。

如果全局变量的名字和局部变量的名字相同，使用的是局部变量

## 匿名函数

```
package main

import &#34;fmt&#34;

var (
	test = func(a, b int) int {
		return a &#43; b
	}(10, 20)
)

var test1 = func(age ...int) int {
	var sum int

	for n := 0; n &lt; len(age); n&#43;&#43; {
		sum &#43;= age[n]
	}
	return sum
}

func main() {
	c := test
	fmt.Println(c)

	d := test1(100, 100, 100)
	fmt.Println(d)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201004004811240-1122578804.png)

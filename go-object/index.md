# go面向对象


所谓的面向对象其实就是找一个专门做这个事的人来做，不用关心具体怎么实现的。所以说，面向过程强调的是过程，步骤。而面向对象强调的是对象，也就是干事的人。

## Go语言：面向对象语言特性

- 方法
- 嵌入
- 接口

- 没有类
- 支持类型。 特别是， 它支持structs。 Structs是用户定义的类型。 Struct类型(含方法)提供类似于其它语言中类的服务。

### Structs

一个struct定义一个状态。 这里有一个strudent struct。 它有一个Name属性和一个布尔类型的标志Real，告诉我们它是一个真实的strudent还是一个虚构的strudent。 Structs只保存状态，不保存行为。

```go
type Creature struct {
  Name string
  Real bool
}
```

### 为结构体添加方法

方法是对特定类型进行操作的函数。 它们有一个**接收器条款**，命令它们对什么样的类型可进行操作。 这里是一个Hello()方法，它可对student结构进行操作，并打印出它们的状态：

```go
func (s Student) Hello() {
  fmt.Printf(&#34;Name: &#39;%s&#39;, Real: %t\n&#34;, s.Name, s.Real)
}
```

`func (s Student) func_name(){}` 这是一个不太常见的语法，但是它非常的具体和清晰，不像this的隐喻性。

一般在定义方法时，需要定义为结构体的指针，值类型的在修改结构体属性时，无法修改其内容

```go
package main

import &#34;fmt&#34;

type human struct {
	Name string
	Real bool
}

type Student struct {
	human
	Id int
}

func (h human) Hello() {
	fmt.Printf(&#34;姓名：%s\n&#34;, h.Name)

}

func (s Student) PrintId() {
	fmt.Printf(&#34;学号：%d\n&#34;, s.Id)
}

func (s Student) EditId(id int) {
	s.Id = id
}

func main() {
	zhangsan := Student{
		human: human{&#34;zhangsan&#34;, true},
		Id:    10,
	}

	zhangsan.Hello()
	zhangsan.EditId(101)
	zhangsan.PrintId()
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201009175856183-293037939.png)

### 嵌入（继承）

可以将匿名的类型嵌入进struct。 如果你嵌入一个匿名的struct那么被嵌入的struct对接受嵌入的struct直接提供它自己的状态（和方法）。 比如，`strudent` 有一个匿名子的被嵌入的 `human` struct，这意味着一个 `student` 就是一个 `hunman`。

```go
package main

import &#34;fmt&#34;

type human struct {
	Name string
	Real bool
}

type Student struct {
	human
	Id int
}

func (h *human) Hello() {
	fmt.Printf(&#34;姓名：%s&#34;, h.Name)

}

func main() {
	zhangsan := &amp;Student{
		human: human{&#34;zhangsan&#34;, true},
		Id:    10,
	}

	zhangsan.Hello()
}
```

![](https://img2020.cnblogs.com/blog/1380340/202010/1380340-20201009174556489-489320062.png)

### 重写

就是子类(结构体)中的方法，将父类中的相同名称的方法的功能重新给改写了

注意：在调用时，默认调用的是子类中的方法

### 方法值和表达式值

方法表达式，即方法对象赋值给变量，方法表达式有两种使用方式：
- 隐式调用：方法值，调用函数时，无需再传递接收者，隐藏了接收者
- 显式调用：方法表达式，显示的把接收者*Student传递过去

实例：

```go
package main

import &#34;fmt&#34;

type human struct {
	Name string
	Real bool
}

type Student struct {
	human
	Id int
}

func (h *human) Hello() {
	fmt.Printf(&#34;姓名：%s\n&#34;, h.Name)

}

func (s Student) PrintId() {
	fmt.Printf(&#34;学号：%d\n&#34;, s.Id)
}

func (s Student) EditId(id int) {
	s.Id = id
}

func main() {
	zhangsan := Student{
		human: human{&#34;zhangsan&#34;, true},
		Id:    10,
	}

	// 常规调用
	zhangsan.Hello()

	// 方法值 无需传递接收者
	hello := zhangsan.Hello
	hello()

	// 方法表达式，调用函数式，传递接收者
	hello1 := (*Student).Hello // 括号是因为 . 的优先级要高于取指符，需要做特殊处理

	hello1(&amp;zhangsan)
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201009181121417-3067588.png)


## Go语言：面向对象的设计

### 接口

接口是Go语言对面向对象支持的标志。 接口是声明方法集的类型。 实现所有接口方法的对象自动地实现接口。 它没有继承或子类或 `implements` 关键字。 

#### 接口的定义

```go
type 接口名字  interface {

	方法声明
}
```

#### 接口的继承

```go
package main

import &#34;fmt&#34;

type Fooer interface {
	Foo1()
}

type Fooerson interface {
	Fooer
	Foo2()
}

type Foo struct {
}

type Fooson struct {
}

func (f Fooson) Foo1() {
	fmt.Println(&#34;Foo1() here&#34;)
}

func (f Fooson) Foo2() {
	fmt.Println(&#34;Foo2() here&#34;)
}

func (f Foo) Foo1() {
	fmt.Println(&#34;Foo1() here&#34;)
}

func main() {
	var fooerson Fooson
	var fooson Fooerson

	fooson = &amp;fooerson
	fooson.Foo1()
	fooson.Foo2()

	var foo Fooer

	foo = fooson
	fooson = foo  // 这样是不允许的，fooson为Fooerson接口的实现，而foo是一个Fooer接口类型的变量，可以子转换为父不能反之
	foo.Foo1() 

}
```
![](https://img2020.cnblogs.com/blog/1380340/202010/1380340-20201009232825993-1946944685.png)


#### 空接口

空接口(interface{})不包含任何的方法，正因为如此，所有的类型都实现了空接口，因此空接口可以存储任意类型的数值

```go
package main

import &#34;fmt&#34;

func main() {

	var arr []interface{}

	arr = append(arr, 1, &#34;zhangsan&#34;, 3.3)
	fmt.Println(arr)
}
```
![](https://img2020.cnblogs.com/blog/1380340/202010/1380340-20201009233003804-1098270083.png)

#### 类型断言

通过类型断言，可以判断空接口中存储的数据类型。

语法：`value, ok := m.(T)`

m:表空接口类型变量

T:是断言的类型

value: 变量m中的值。

ok: 布尔类型变量，如果断言成功为true,否则为false

```go
func main() {
	var a interface{}

	a = 10

	ok, value := a.(int)
	fmt.Println(ok, value)

	ok1, value1 := a.(string)
	fmt.Println(ok1, value1)
}
```

![](https://img2020.cnblogs.com/blog/1380340/202010/1380340-20201009234432928-2114138876.png)


### 封装

Go语言在包的级别进行封装。 以小写字母开头的名称只在该程序包中可见。 可以隐藏私有包中的任何内容，只暴露特定的类型，接口和工厂函数。

例如，在这里要隐藏上面的Foo类型，只暴露接口，你可以将其重命名为小写的foo，并提供一个NewFoo()函数，返回公共Fooer接口：

```go
type foo struct {
}
 
func (f foo) Foo1() {
    fmt.Println(&#34;Foo1() here&#34;)
}
 
func (f foo) Foo2() {
    fmt.Println(&#34;Foo2() here&#34;)
}
 
func (f foo) Foo3() {
    fmt.Println(&#34;Foo3() here&#34;)
}
 
func NewFoo() Fooer {
    return &amp;Foo{}
}
```
在另一个包的代码可以使用`NewFoo()`并访问由内部foo类型实现的Footer接口：
```go
f := NewFoo()
 
f.Foo1()
 
f.Foo2()
 
f.Foo3()
```
### 继承

Go语言没有任何类型层次结构。 它允许你通过组合来共享实现的细节。 但Go语言，允许嵌入匿名组合。

通过嵌入一个匿名类型的组合等同于实现继承，这是它所有意图和目的。 一个嵌入的struct与基类一样脆弱。 你还可以嵌入一个接口， 如果嵌入类型没有实现所有接口方法，它甚至可能导致产生在编译时未被发现的运行错误。

这里SuperFoo嵌入Fooer接口，但是SuperFoo没有实现Foo的方法。 Go编译器会愉快地让你创建一个新的SuperFood并调用Fooer的方法，但很显然这在运行时会失败。 这会编译：

```go
package main

import &#34;fmt&#34;

type Fooer interface {
	Foo1()
	Foo2()
	Foo3()
}

type Foo struct {
}

func (f Foo) Foo1() {
	fmt.Println(&#34;Foo1() here&#34;)
}

func (f Foo) Foo2() {
	fmt.Println(&#34;Foo2() here&#34;)
}

func (f Foo) Foo3() {
	fmt.Println(&#34;Foo3() here&#34;)
}

type SuperFooer struct {
	Fooer
}

func main() {
	s := SuperFooer{}
	s.Foo3()
}
```

![](https://img2020.cnblogs.com/blog/1380340/202010/1380340-20201009231036948-951548379.png)


### 多态

多态性是面向对象编程的本质：只要对象坚持实现同样的接口，Go语言就能处理不同类型的那些对象。 Go接口以非常直接和直观的方式提供这种能力。

Golang当中的接口解决了这个问题，只要接口中定义的方法能对应的上，那么就可以认为这个类实现了这个接口。同一个接口，使用不同的实例而执行不同操作


![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201009233350848-426016267.png)

```go
package main

import (
	&#34;fmt&#34;
)

type animal interface {
	Say()
}

type human struct {
}

type cat struct {
}

func (h human) Say() {
	fmt.Println(&#34;人类&#34;)
}

func (c cat) Say() {
	fmt.Println(&#34;猫&#34;)
}

func main() {
	var a animal

	a = cat{}
	a.Say()

	a = human{}
	a.Say()
}
```

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201009233804687-1199049992.png)

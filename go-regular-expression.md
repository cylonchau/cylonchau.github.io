# 正则表达式在go中使用


正则表达式是一种进行模式匹配和文本操纵的复杂而又强大的工具。虽然正则表达式比纯粹的文本匹配效率低，但是它却更灵活。按照它的语法规则，随需构造出的匹配模式就能够从原始文本中筛选出几乎任何你想要得到的字符组合。

Go语言通过regexp（regular expression）标准包为正则表达式提供了官方支持，包名采用`regular expression`的每个单词的前三个首字母组成。

Go语言的正则表达式实现的是RE2标准，Go语言的正则表达式与其他编程语言之间也有一些小的差异。

## 正则表达式规则

![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201022164330719-1170561795.png)

## go语言中regexp包使用

简单来说，Go语言中使用正则表达式只需要两步即可：
- 解析、编译正则表达式 `regexp.MustCompile()` 返回一个regexp结构体
- 根据解析好的规则（结构体形式），从指定字符串中提取需要的信息。如 `MatchString()` `FindAllSubmatch()`等

```go
package main

import (
	"fmt"
	"regexp"
)

func main() {
	rege := regexp.MustCompile(`(\d{1,3}\.){3}\d{1,3}`)
	str := rege.FindAllString("SLAJDLKAJ192.168.0.1DASDASA1231", -1)
	fmt.Println(str)

}
```
![](https://raw.githubusercontent.com/CylonChau/imgbed/main/img/1380340-20201022165701430-1274324162.png)

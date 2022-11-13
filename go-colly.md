# go colly


## Introduction

本文对colly如何使用，整个代码架构设计，以及一些使用实例的收集。

Colly是Go语言开发的Crawler Framework，并不是一个完整的产品，Colly提供了类似于Python的同类产品（`BeautifulSoup` 或  `Scrapy`）相似的表现力和灵活性。

Colly这个名称源自 `Collector`  的简写，而`Collector` 也是 Colly的核心。

[Colly Official Docs](http://go-colly.org/docs/)，内容不是很多，最新的消息也很就远了，仅仅是活跃在Github

## Concepts

### Architecture

从理解上来说，Colly的设计分为两层，核心层和解析层，

- `Collector` ：是Colly实现，该组件负责网络通信，并负责在`Collector` 作业运行时执行对应事件的回调。
- `Parser `：这个其实是抽象的，官网并未对此说明，goquery和一些htmlquery，通过这些就可以将访问的结果解析成类Jquery对象，使html拥有了，XPath选择器和CSS选择器

通常情况下Crawler的工作流生命周期大致为

&gt; - 构建客户端
&gt; - 发送请求
&gt; - 获取响应的数据
&gt; - 将相应的数据解析
&gt; - 对所需数据处理
&gt; - 持久化

而Colly则是将这些概念进行封装，通过将事件注册到每个步骤中，通过事件的方式对数据进行清理，抽象来说，Colly面向的是过程而不是对象。大概的工作架构如图

![image-20220401003447726](../../images/gocolly/image-20220401003447726.png)

### event

通过上述的概念，可以大概了解到 `Colly ` 是一个基于事件的Crawler，通过开发者自行注册事件函数来触发整个流水线的工作

Colly 具有以下事件处理程序：

- OnRequest：在请求之前调用
- OnError ：在请求期间发生错误时调用
- OnResponseHeaders ：在收到响应头后调用
- OnResponse： 在收到响应后调用
- OnHTML：如果接收到的内容是 HTML，则在 OnResponse 之后立即调用
- OnXML ：如果接收到的内容是 HTML 或 XML，则在 OnHTML 之后立即调用
- OnScraped：在 OnXML 回调之后调用
- OnHTMLDetach：取消注册一个OnHTML事件函数，取消后，如未执行过得事件将不会再被执行
- OnXMLDetach：取消注册一个OnXML事件函数，取消后，如未执行过得事件将不会再被执行

&gt; Reference 
&gt;
&gt; [goquery](https://github.com/PuerkitoBio/goquery)
&gt;
&gt; [htmlquery](https://github.com/antchfx/htmlquery)

## Utilities

### 简单使用

```go
package main

import (
	&#34;fmt&#34;

	&#34;github.com/gocolly/colly&#34;
)

func main() {
	// Instantiate default collector
	c := colly.NewCollector(
		// Visit only domains: hackerspaces.org, wiki.hackerspaces.org
		colly.AllowedDomains(&#34;hackerspaces.org&#34;, &#34;wiki.hackerspaces.org&#34;),
	)

	// On every a element which has href attribute call callback
	c.OnHTML(&#34;a[href]&#34;, func(e *colly.HTMLElement) {
		link := e.Attr(&#34;href&#34;)
		// Print link
		fmt.Printf(&#34;Link found: %q -&gt; %s\n&#34;, e.Text, link)
		// Visit link found on page
		// Only those links are visited which are in AllowedDomains
		c.Visit(e.Request.AbsoluteURL(link))
	})

	// Before making a request print &#34;Visiting ...&#34;
	c.OnRequest(func(r *colly.Request) {
		fmt.Println(&#34;Visiting&#34;, r.URL.String())
	})

	// Start scraping on https://hackerspaces.org
	c.Visit(&#34;https://hackerspaces.org/&#34;)
}
```

### 错误处理

```go
package main

import (
	&#34;fmt&#34;

	&#34;github.com/gocolly/colly&#34;
)

func main() {
	// Create a collector
	c := colly.NewCollector()

	// Set HTML callback
	// Won&#39;t be called if error occurs
	c.OnHTML(&#34;*&#34;, func(e *colly.HTMLElement) {
		fmt.Println(e)
	})

	// Set error handler
	c.OnError(func(r *colly.Response, err error) {
		fmt.Println(&#34;Request URL:&#34;, r.Request.URL, &#34;failed with response:&#34;, r, &#34;\nError:&#34;, err)
	})

	// Start scraping
	c.Visit(&#34;https://definitely-not-a.website/&#34;)
}
```

### 处理本地文件

word.html

```html
&lt;!DOCTYPE html&gt;
&lt;html lang=&#34;en&#34;&gt;
&lt;head&gt;
    &lt;meta charset=&#34;UTF-8&#34;&gt;
    &lt;title&gt;Document title&lt;/title&gt;
&lt;/head&gt;
&lt;body&gt;
&lt;p&gt;List of words&lt;/p&gt;
&lt;ul&gt;
    &lt;li&gt;dark&lt;/li&gt;
    &lt;li&gt;smart&lt;/li&gt;
    &lt;li&gt;war&lt;/li&gt;
    &lt;li&gt;cloud&lt;/li&gt;
    &lt;li&gt;park&lt;/li&gt;
    &lt;li&gt;cup&lt;/li&gt;
    &lt;li&gt;worm&lt;/li&gt;
    &lt;li&gt;water&lt;/li&gt;
    &lt;li&gt;rock&lt;/li&gt;
    &lt;li&gt;warm&lt;/li&gt;
&lt;/ul&gt;
&lt;footer&gt;footer for words&lt;/footer&gt;
&lt;/body&gt;
&lt;/html&gt;
```



```go
package main

import (
    &#34;fmt&#34;
    &#34;net/http&#34;

    &#34;github.com/gocolly/colly/v2&#34;
)

func main() {

    t := &amp;http.Transport{}
    t.RegisterProtocol(&#34;file&#34;, http.NewFileTransport(http.Dir(&#34;.&#34;)))

    c := colly.NewCollector()
    c.WithTransport(t)

    words := []string{}

    c.OnHTML(&#34;li&#34;, func(e *colly.HTMLElement) {
        words = append(words, e.Text)
    })

    c.Visit(&#34;file://./words.html&#34;)

    for _, p := range words {
        fmt.Printf(&#34;%s\n&#34;, p)
    }
}
```

### 使用代理交换器

通过 `ProxySwitcher` , 可以直接使用一批代理IP池进行访问了，然而这里只有RR，如果需要其他的均衡算法，需要有自己实现了

```go
package main

import (
	&#34;bytes&#34;
	&#34;log&#34;

	&#34;github.com/gocolly/colly&#34;
	&#34;github.com/gocolly/colly/proxy&#34;
)

func main() {
	// Instantiate default collector
	c := colly.NewCollector(colly.AllowURLRevisit())

	// Rotate two socks5 proxies
	rp, err := proxy.RoundRobinProxySwitcher(&#34;socks5://127.0.0.1:1337&#34;, &#34;socks5://127.0.0.1:1338&#34;)
	if err != nil {
		log.Fatal(err)
	}
	c.SetProxyFunc(rp)

	// Print the response
	c.OnResponse(func(r *colly.Response) {
		log.Printf(&#34;Proxy Address: %s\n&#34;, r.Request.ProxyURL)
		log.Printf(&#34;%s\n&#34;, bytes.Replace(r.Body, []byte(&#34;\n&#34;), nil, -1))
	})

	// Fetch httpbin.org/ip five times
	for i := 0; i &lt; 5; i&#43;&#43; {
		c.Visit(&#34;https://httpbin.org/ip&#34;)
	}
}

```

### 随机延迟

该功能可以对行为设置一种特征，以免被反扒机器人检测，并禁止我们，如速率限制和延迟

```go
package main

import (
	&#34;fmt&#34;
	&#34;time&#34;

	&#34;github.com/gocolly/colly&#34;
	&#34;github.com/gocolly/colly/debug&#34;
)

func main() {
	url := &#34;https://httpbin.org/delay/2&#34;

	// Instantiate default collector
	c := colly.NewCollector(
		// Attach a debugger to the collector
		colly.Debugger(&amp;debug.LogDebugger{}),
		colly.Async(true),
	)

	// Limit the number of threads started by colly to two
	// when visiting links which domains&#39; matches &#34;*httpbin.*&#34; glob
	c.Limit(&amp;colly.LimitRule{
		DomainGlob:  &#34;*httpbin.*&#34;,
		Parallelism: 2,
		RandomDelay: 5 * time.Second,
	})

	// Start scraping in four threads on https://httpbin.org/delay/2
	for i := 0; i &lt; 4; i&#43;&#43; {
		c.Visit(fmt.Sprintf(&#34;%s?n=%d&#34;, url, i))
	}
	// Start scraping on https://httpbin.org/delay/2
	c.Visit(url)
	// Wait until threads are finished
	c.Wait()
}
```

### 多线程请求队列

```
package main

import (
	&#34;fmt&#34;

	&#34;github.com/gocolly/colly&#34;
	&#34;github.com/gocolly/colly/queue&#34;
)

func main() {
	url := &#34;https://httpbin.org/delay/1&#34;

	// Instantiate default collector
	c := colly.NewCollector(colly.AllowURLRevisit())

	// create a request queue with 2 consumer threads
	q, _ := queue.New(
		2, // Number of consumer threads
		&amp;queue.InMemoryQueueStorage{MaxSize: 10000}, // Use default queue storage
	)

	c.OnRequest(func(r *colly.Request) {
		fmt.Println(&#34;visiting&#34;, r.URL)
		if r.ID &lt; 15 {
			r2, err := r.New(&#34;GET&#34;, fmt.Sprintf(&#34;%s?x=%v&#34;, url, r.ID), nil)
			if err == nil {
				q.AddRequest(r2)
			}
		}
	})

	for i := 0; i &lt; 5; i&#43;&#43; {
		// Add URLs to the queue
		q.AddURL(fmt.Sprintf(&#34;%s?n=%d&#34;, url, i))
	}
	// Consume URLs
	q.Run(c)

}
```

### 异步

默认情况下，Colly的工作模式是同步的。可以使用 `Async` 函数启用异步模式。在异步模式下，我们需要调用`Wait` 等待`Collector` 工作完成。

```go
package main

import (
    &#34;fmt&#34;

    &#34;github.com/gocolly/colly/v2&#34;
)

func main() {

    urls := []string{
        &#34;http://webcode.me&#34;,
        &#34;https://example.com&#34;,
        &#34;http://httpbin.org&#34;,
        &#34;https://www.perl.org&#34;,
        &#34;https://www.php.net&#34;,
        &#34;https://www.python.org&#34;,
        &#34;https://code.visualstudio.com&#34;,
        &#34;https://clojure.org&#34;,
    }

    c := colly.NewCollector(
        colly.Async(),
    )

    c.OnHTML(&#34;title&#34;, func(e *colly.HTMLElement) {
        fmt.Println(e.Text)
    })

    for _, url := range urls {

        c.Visit(url)
    }

    c.Wait()

}
```

### 最大深度

深度是在访问这个页面时，其页面还有link，此时需要采集到入口link几层的link？默认1

```go
package main

import (
   &#34;fmt&#34;

   &#34;github.com/gocolly/colly&#34;
)

func main() {
   // Instantiate default collector
   c := colly.NewCollector(
      // MaxDepth is 1, so only the links on the scraped page
      // is visited, and no further links are followed
      colly.MaxDepth(1),
   )

   // On every a element which has href attribute call callback
   c.OnHTML(&#34;a[href]&#34;, func(e *colly.HTMLElement) {
      link := e.Attr(&#34;href&#34;)
      // Print link
      fmt.Println(link)
      // Visit link found on page
      e.Request.Visit(link)
   })

   // Start scraping on https://en.wikipedia.org
   c.Visit(&#34;https://en.wikipedia.org/&#34;)
}
```

&gt; Reference
&gt;
&gt; [gocolly](https://faun.pub/web-scrapping-using-golang-gocolly-21f824070291)
&gt;
&gt; [colly](https://zetcode.com/golang/colly/)

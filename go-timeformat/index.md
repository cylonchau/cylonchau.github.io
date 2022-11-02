# go时间格式化


该文可以快速在Go语言中获得时间的计算。

## 在Go中获取时间

### 如何获取当前时间

```go
now := time.Now()
fmt.Printf(&#34;current time is :%s&#34;, now)

current time is :2009-11-10 23:00:00 &#43;0000 UTC m=&#43;0.000000001
```

### 如何获取UNIX Timestamp

```go
cur_time := time.Now().Unix()
fmt.Printf(&#34;current unix timestamp is :%v\n&#34;, cur_time )
```

### 如何获取当日0:00:00 0:00:00

```go
now := time.Now()
date := time.Date(now.Year(), now.Month(), now.Day(),0, 0, 0, 0, time.Local)
fmt.Printf(&#34;date is :%s&#34;, date)

date is :2021-04-13 00:00:00 &#43;0800 
```

### 如何获取时区时间

标准时间 `time.Now().UTC()`
本地时区 `time.Now().Local()`

```go

 // 获取0时区时间
fmt.Printf(&#34;date is :%s\n&#34;, time.Now().UTC())

date is :2021-04-13 16:02:33.853254 &#43;0000 UTC

// 快速设置时区
timeLocation, _ := time.LoadLocation(&#34;Asia/Tokyo&#34;) //使用时区码
fmt.Println(time.Now().In(timeLocation).String()) // 快速设置时区

2021-04-14 01:09:18.140997 &#43;0900 JST
```

## Go中的固定时间格式

### 获取月份

```
time.April

type Month int

const (
	January Month = 1 &#43; iota
	February
	March
	April
	May
	June
	July
	August
	September
	October
	November
	December
)
```

### 获取星期

```
time.Sunday


type Weekday int

const (
	Sunday Weekday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
)
```


## Go中的时间格式化

Go中时间格式化的格式为 `2006-01-02 15:04:05` 612345为格式，而不是具体时间


```go
    
    // YYYY-MM-DD
    fmt.Println(time.Now().Format(&#34;2006-01-02&#34;))
 
    //  YYYY-MM-DD hh:mm:ss
    fmt.Println(time.Now().Format(&#34;2006-01-02 15:04:05&#34;))
    
    //  M-DD
    fmt.Println(time.Now().Format(&#34;1-02&#34;))

    //  MM-DD
    fmt.Println(time.Now().Format(&#34;01-02&#34;)
   
    // 获取当前的小时、分钟、秒（整数）
    nowHour, nowMinute, nowSecond = time.Now().Clock()

    // 获取前一天
    //  AddDate(Years, months, days)
    yesterday = time.Now().AddDate(0,0,-1).Format(&#34;01/02&#34;)

    // 显示星期英文简写
    fmt.Println(time.Now().Format(&#34;2006-01-02 15:04:05 Mon&#34;))
 
    // 星期的大写
    fmt.Println(time.Now().Format(&#34;2006-01-02 15:04:05 Monday&#34;))
 
    // 增加微秒
    fmt.Println(time.Now().Format(&#34;2006-01-02 15:04:05.000000&#34;))
 
    // 纳秒
    fmt.Println(time.Now().Format(&#34;2006-01-02 15:04:05.000000000&#34;))
}

// print result
08-10-2018
08-10-2018 21:11:58
08-10-2018 21:11:58 Fri
08-10-2018 21:11:58 Friday
08-10-2018 21:11:58.880934
08-10-2018 21:11:58.880934320
```

## Go中的时间计算

### 如何获取本周日期有哪些？

获取一个星期的第一天是几号

```
t:=time.Now()
fmt.Println(t.Weekday()) // 获取现在时间为本周的星期几

```
得到本日为星期几后，可以对时间进行计算，因为time包内星期的常量都为int，可以直接进行算数运算.
用一周的第一天减去当日为星期几，如果为0既『本日为本周的第一天』

`time.AddDate(year, month, date)`，仅可以添加年月日
`time.Add(Hours, Minutes, Seconds)`，仅可以添加时分秒

```go
offset := int(time.Monday - t.Weekday()) //=-1
```
如不为0，time包提供了，「以当前时间为基点，进行加减运算」

```go
// t.AddDate(year, month, date)
t.AddDate(0,0,offset) // 可以获取到，周一为几月几日
```
综上所属，可以获得每周第一天为几月几日，每周随后一天为几月几日

```go

/**
  *     获取上周周第一天具体年月日
**/

func GetLastWeekFirstDate() (weekMonday string) {
	thisWeekMonday := GetFirstDateOfWeek()
	TimeMonday, _ := time.Parse(&#34;2006-01-02&#34;, thisWeekMonday)
	lastWeekMonday := TimeMonday.AddDate(0, 0, -7)
	weekMonday = lastWeekMonday.Format(&#34;2006-01-02&#34;)
	return
}


/**
  *     获取本周的周一具体年月日
**/

func GetFirstDateOfWeek() (weekMonday string) {
	now := time.Now()
	offset := int(time.Monday - now.Weekday())
	if offset &gt; 0 {
		offset = -6
	}

	weekStartDate := time.Date(now.Year(), now.Month(), now.Day(), 0, 0, 0, 0, time.Local).AddDate(0, 0, offset)
	weekMonday = weekStartDate.Format(&#34;2006-01-02&#34;)
	return
}

/**
  *     获取上周最后一天具体年月日
**/

func GetLastWeekLastDate() (weekMonday string) {
	now := time.Now()

	offset := int(time.Monday - now.Weekday())
	if offset &gt; 0 {
		offset = -6
	}

	weekStartDate := time.Date(now.Year(), now.Month(), now.Day(), 0, 0, 0, 0, time.Local).AddDate(0, 0, offset)
	weekMonday = weekStartDate.AddDate(0, 0, -1).Format(&#34;2006-01-02&#34;)
	return
}

/**
  *     获取上周一星期所有天数的具体年月日
**/

func GetBetweenDates(sdate, edate string) []string {
	d := []string{}
	timeFormatTpl := &#34;2006-01-02 15:04:05&#34;
	if len(timeFormatTpl) != len(sdate) {
		timeFormatTpl = timeFormatTpl[0:len(sdate)]
	}
	date, err := time.Parse(timeFormatTpl, sdate)
	if err != nil {
		return d
	}
	date2, err := time.Parse(timeFormatTpl, edate)
	if err != nil {
		return d
	}
	if date2.Before(date) {
		return d
	}
	// 输出日期格式固定
	timeFormatTpl = &#34;2006-01-02&#34;
	date2Str := date2.Format(timeFormatTpl)
	d = append(d, date.Format(timeFormatTpl))
	for {
		date = date.AddDate(0, 0, 1)
		dateStr := date.Format(timeFormatTpl)
		d = append(d, dateStr)
		if dateStr == date2Str {
			break
		}
	}
	return d
}
```

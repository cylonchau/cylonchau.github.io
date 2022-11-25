# Go语言数据类型转换


## string  in mutual conversion


```go
#string到int
int,err:=strconv.Atoi(string)

#string到int64
int64, err := strconv.ParseInt(string, 10, 64)

#int到string
string:=strconv.Itoa(int)

#int64到string
string:=strconv.FormatInt(int64,10)
```

## slice to struct

> Question: in golang how to convert slice to struct


### scene 1：use reflect convert  slice to struct
```go
func SliceToStruct(array interface{}) (forwardPort *ForwardPort, err error) {
	forwardPort = &ForwardPort{}
	valueOf := reflect.ValueOf(forwardPort)
	if valueOf.Kind() != reflect.Ptr {
		return nil, errors.New("must ptr")
	}
	valueOf = valueOf.Elem()
	if valueOf.Kind() != reflect.Struct {
		return nil, errors.New("must struct")
	}

	switch array.(type) {
	case []string:
		arrayImplement := array.([]string)
		for i := 0; i < valueOf.NumField(); i++ {
			if i >= len(arrayImplement) {
				break
			}
			val := arrayImplement[i]
			if val != "" && reflect.ValueOf(val).Kind() == valueOf.Field(i).Kind() {
				valueOf.Field(i).Set(reflect.ValueOf(val))
			}
		}
	case []interface{}:
		arrayImplement := array.([]interface{})
		for i := 0; i < valueOf.NumField(); i++ {
			if i >= len(arrayImplement) {
				break
			}
			val := arrayImplement[i]
			if val != "" && reflect.ValueOf(val).Kind() == valueOf.Field(i).Kind() {
				valueOf.Field(i).Set(reflect.ValueOf(val))
			}
		}
	}

	return forwardPort, nil
}
```

### struct to anything 


https://github.com/fatih/structs


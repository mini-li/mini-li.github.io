---
title: reflect
layout: default
parent: go
has_children: false
---

## 1. 反射

Go 中的反射是建立在类型系统之上，它与空接口 interface{} 密切相关。

{: .note }
每个 interface{} 类型的变量包含一对值 （type，value），type 表示变量的类型信息，value 表示变量的值信息  
reflect.TypeOf() 获取类型信息，返回 Type 类型  
reflect.ValueOf() 获取数据信息，返回 Value 类型  


struct 反射示例

```go
package main

import (
	"fmt"
	"reflect"
)
type student struct {
	Name string `json:"name"`
	Age  int    `json:"age" id:"1"`
}
func main() {
	stu := student{
		Name: "lee",
		Age:  15,
	}
	valueOfStu := reflect.ValueOf(stu)
	// 获取struct字段数量
	fmt.Println("NumFields: ", valueOfStu.NumField())
	// 获取字段 Name 的值
	fmt.Println("Name value: ", valueOfStu.Field(0).String(), ", ", valueOfStu.FieldByName("Name").String())
	// 字段类型
	fmt.Println("Name type: ", valueOfStu.Field(0).Type())
	typeOfStu := reflect.TypeOf(stu)
	for i := 0; i < typeOfStu.NumField(); i++ {
		// 获取字段名
		name := typeOfStu.Field(i).Name
		fmt.Println("Field Name: ", name)
		// 获取tag
		if fieldName, ok := typeOfStu.FieldByName(name); ok {
			tag := fieldName.Tag

			fmt.Println("tag-", tag, ", ", "json:", tag.Get("json"), ", id", tag.Get("id"))
		}
	}
}
```

{: .important }
反射三大定律：  
第一定律：反射是从接口值到反射对象  
第二定律：从反射对象可以获取接口值  
第三定律：要修改反射对象的值，其值必须可以设置  

## 2. 反射设置变量值

```go
package main

import (
	"fmt"
	"reflect"
)

type student struct {
	Age int
}

func main() {
	var stu student
	fmt.Println("origin age: ", stu.Age)

	valOfStu := reflect.ValueOf(&stu)

	canSet := valOfStu.CanSet()
	fmt.Println("can set: ", canSet)

	valOfStu = valOfStu.Elem()

	field := valOfStu.FieldByName("Age")

	field.SetInt(23)
	fmt.Println("set age: ", field.Int())

	//====================
	fmt.Println("====================")
	var num int32 = 24
	v := reflect.ValueOf(&num)
	fmt.Println("num:", num, ", elem Kind: ", v.Elem().Kind())
	if v.Elem().Kind() == reflect.Int32 {
		v.Elem().SetInt(300)
	}
	fmt.Println("set num: ", num)
}
```

## 3. 用反射实现动态调用

```go
package main

import (
	"fmt"
	"reflect"
)

type Routers struct {
}

func (router *Routers) Hello(args ...interface{}) bool {
	fmt.Println(args[0])
	return true
}
func (router *Routers) World(args ...interface{}) bool {
	fmt.Println(args[0])
	return true
}

type FuncCollection map[string]reflect.Value

func main() {
	_, _ = CallFunc("Hello", "执行Hello方法")
	_, _ = CallFunc("World", "执行World方法")
}

func CallFunc(tableName string, args ...interface{}) (result []reflect.Value, err error) {
	var router Routers
	FuncMap := make(FuncCollection, 0)
	rf := reflect.ValueOf(&router)
	rft := rf.Type()
	funcNum := rf.NumMethod()
	for i := 0; i < funcNum; i++ {
		mName := rft.Method(i).Name
		// rft.Method(i)
		FuncMap[mName] = rf.Method(i)
	}
	parameter := make([]reflect.Value, len(args))
	for k, arg := range args {
		parameter[k] = reflect.ValueOf(arg)
	}
	result = FuncMap[tableName].Call(parameter)
	return
}
```

## 4. 反射优缺点

**优点**

> 可以根据条件灵活的调用函数。最大一个优点就是灵活。  
> 比如函数参数的数据类型不确定，这时可以根据反射来判断数据类型，在调用适当的函数。  
> 还有比如根据某些条件来调用哪个函数。  
> 需要根据动态需要来调用函数，可以用反射。  
> 使用反射的 2 个典型场景：1、操作数据库的 ORM 框架 ，2、依赖注入  

**缺点**

> 用反射编写的代码比较难以阅读和理解  
> 反射是在运行时才执行，所以编译期间比较难以发现错误  
> 反射对性能的影响，比一般正常运行代码慢一到两个数量级。  
> 这里性能其实是你的业务量到了一定时候才要注意。量不大情况，够用。  
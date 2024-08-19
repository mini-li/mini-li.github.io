---
title: reflect
layout: default
parent: go
has_children: false
---

## 1. 动态调用

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
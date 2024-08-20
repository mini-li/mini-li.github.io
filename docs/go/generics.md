---
title: generics
layout: default
parent: go
has_children: false
---

## 1. 泛型基本用法

- Golang 支持泛型函数、泛型类型 ...  
- Golang 内置两个泛型约束关键字:  
    - comparable  代表所有可以进行比较的类型（能进行 !=、== 对比，可用于 map 的键）  
    - any         代表任何类型，它是空接口的别名，即 type any = interface{}  

```go
// 在函数名及其参数列表之间插入方括号，作为表示类型的参数 "type parameter"
// 类型参数跟函数参数一样，需要指定其 "类型"，这种类型的类型称之为约束
// 下例 [T any] 是对类型的约束，它也是函数签名的一部分，T 是类型的形参（它在定义函数时类型是不确定的）
func GenericFunc[T any](args T) { ... }
// 当仅支持几个特定类型时可写为如下，其中的 int64|float64 即类型约束 "type constraint"
// 符号 | 用于告诉编译器，类型形参 T 只接受 int64 或 float64 这两种类型的实参
func GenericFunc[T int64|float64](args T) { ... }

// 如果只想支持几个特定类型，就不需要用 comparable 了
func Compare[T int64|float64](a,b T) bool {
	if a >= b {
		return true
	} else {
		return false
	}
}

// 当要支持很多类型时可以用接口表示，这里的 Number 即自定义的泛型类型
type Number interface {
    int|int32|int64|float64|float32
}
func GenericFunc[T Number](args T) { ... }

// 因为下例的中括号里定义了所有的类型形参，所以称其为类型形参列表 "type parameter list"
func GenericFunc[X int64|float64, Y int32|float32](a X, sum ...Y) {
    // .......
    return sum
}


// 当 ~ 符与类型一起出现时表示基于该基本类型的衍生类型（表示不光是 int，所有以 int 为底层类型的类型都能用于实例化）
type NumStr interface {
    ~int | ~uint | ~float64 | ~string
}

// 在函数签名中直接声明其类型约束
func AddNumStr[T NumStr](params []T) (sum T) {
    for _, param := range params {
        sum += param
    }
    return
}
```

## 2. 结构体泛型

```go

// 先通过接口声明约束的集合，其中包含所有要支持的类型（泛型的类型限制），这里的 Number 即自定义的泛型类型
type Number interface {
	int|int32|int64|float64|float32
}

// 只有在结构体上声明了泛型，结构体的方法才可以使用泛型
type Stack[T Number] struct {
    size int
    values []T  // 泛型类型
}

func (S *Stack[T]) Push(v T) {
    S.values = append(S.values, v)
    S.size++
}

func (S *Stack[T]) Pop() T {
    e := S.values[S.size-1]
    if S.size != 0 {
        S.values = S.values[:S.size-1]
        S.size--
    }
    return e
}


// INT
strS := &Stack[int64]{}
strS.Push(1)
strS.Push(2)
fmt.Println(strS.Pop())
fmt.Println(strS.Pop())

// FLOAT
floatS := &Stack[float64]{}
floatS.Push(1.1)
floatS.Push(2.2)
fmt.Println(floatS.Pop())
fmt.Println(floatS.Pop())

```

## 3. 接口加方法

```go
// 通过接口实现类型加方法的双重约束
type CanSpeak interface {
    ~int|~int32|~int64|~float32|~float64    // 首先，类型自身必须属于该类型或其衍生类型
    Speak() string                          // 其次，类型必须实现 Speak 方法
}

type Mouth int32
func (this Mouth) Speak() string {
    return fmt.Sprintf("speak %v", this)
}

type Nose string
func (this Nose) Speak() string {
    return fmt.Sprintf("speak %v", this)
}

type Ear int

func SpeakLoudly[T CanSpeak](params []T) {
    for _, param := range params {
        fmt.Println(param.Speak())
    }
}

func TestGenerics(t *testing.T) {
    // 类型与方法均符合
    SpeakLoudly([]Mouth{1, 2, 3, 4, 5, 6})

    // 仅方法符合 ./generics_test.go:172:16: Nose does not implement CanSpeak
    // SpeakLoudly([]Nose{"z", "x", "c"})

    // 仅类型符合 ./generics_test.go:175:16: Ear does not implement CanSpeak (missing Speak method)
    // SpeakLoudly([]Ear{1, 2, 3, 4, 5, 6})
}
```


## 4. 泛型组合

```go

type Int interface {
   int|int8|int16|int32|int64
}

type Uint interface {
    uint|uint8|uint16|uint32
}

type Float interface {
    float32|float64
}

// 使用 | 组合
type Slice[T Int|Uint|Float] []T

// 同时，在接口里也能直接在组合其他接口，所以还能像下面这样（组合了三个接口类型并额外增加了一个 string 类型）
type SliceElement interface {
    Int|Uint|Float|string
}

type Slice[T SliceElement] []T

```

## 5. map

```go
type M[K string, V any] map[K]V     // 泛型类型

func main() {
	m1 := M[string, int]{
		"zx": 123,
		"as": 456,
		"qw": 789,
	}
	m1["addKey"] = 10               // map[addKey:10 as:456 qw:789 zx:123]

	m2 := M[string, string]{
		"a": "1",
		"b": "2",
	}
	m2["c"] = "3"                   // map[a:1 b:2 c:3]
}


// 泛型类型
type Number interface {
	int|int32|int64|float64|float32
}

// 泛型类型
type M[K comparable, V Number] map[K]V

// 泛型类型的方法
func (this M[K, V]) Set(key K, value V) M[K, V] {
	this[key] = value
	return this
}

func main() {
	m := M[int, int]{
		1: 11,
		2: 22,
		3: 33,
	}
	fmt.Println(m) // map[1:11 2:22 3:33]
}


```

## 6. channel

```go
// 该泛型通道可以用类型实参 int 或 string 进行实例化
type Ch[T int|string] chan T

func main() {
	a := make(Ch[int], 10)
	a <- 1

	b := make(Ch[string], 10)
	b <- "dudu"

	fmt.Println(<-a) // 1
	fmt.Println(<-b) // dudu
}
```
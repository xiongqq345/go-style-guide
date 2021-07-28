# Go语言风格指南

本指南借鉴了Uber和官方的代码风格，结合工程中的实践，总结了一些大家经常犯的一些"错误"写法以及建议写法，一部分也是我们code review的规则，希望帮助每一个对代码有追求的工程师，写出优雅的代码。

## 目录


- [规范](#规范)
    - [import 分组](#import-分组)
    - [省略不必要的else](#省略不必要的else)
    - [删除多余的布尔类型判断](#删除多余的布尔类型判断)
    - [减少嵌套](#减少嵌套)
    - [用var关键字初始化零值变量](#用var关键字初始化零值变量)
    - [用 := 运算符初始化非零值变量](#用-运算符初始化非零值变量)
    - [相关联变量考虑在同一行定义或赋值](#相关联变量考虑在同一行定义或赋值)
    - [处理类型断言失败](#处理类型断言失败)
- [结构体](#结构体-struct)
    - [初始化 Struct 引用](#初始化-struct-引用)
    - [结构体中的嵌入](#结构体中的嵌入)
- [切片/数组](#切片数组-slicearray)
    - [使用var声明空切片](#使用var声明空切片)
    - [只拿索引的迭代方式](#只拿索引的迭代方式)
    - [二维数组的定义和初始化](#二维数组的定义和初始化)
    - [指定切片容量](#指定切片容量)
- [哈希表](#哈希表-map)
    - [初始化 Maps](#初始化-maps)
    - [指定切片容量](#指定切片容量)
- [字符串](#规范)
    - [使用原始字符串字面值，避免转义](#使用原始字符串字面值避免转义)

## 规范
### import 分组

导入应该分为两组：

- 标准库
- 其他库

默认情况下，这是 goimports 应用的分组。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
import (
    "fmt"
    "os"
    "go.uber.org/atomic"
    "golang.org/x/sync/errgroup"
)
```

</td><td>

```go
import (
    "fmt"
    "os"

    "go.uber.org/atomic"
    "golang.org/x/sync/errgroup"
)
```

</td></tr>
</tbody></table>
### 省略不必要的else
基本规则是`if`里如果肯定会`return`，`else`就不要写了,免得下面多一层嵌套。
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
func max(x, y int) int {
    if x > y {
        return x
    } else {
        return y
    }
}
```

</td><td>

```go
func max(x, y int) int {
    if x > y {
        return x
    }
    return y
}
```

</td></tr>
</tbody></table>

### 删除多余的布尔类型判断
这点其实大家经常会忽略，不够重视。因为bad写法其实反而是顺着大脑思绪的，很多时候会下意识这样写。
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
if success == true {
    // do something
}
if a == b && success == false {
    // do something
}
```

</td><td>

```go
if success {
    // do something
}
if a == b && !success {
    // do something
}
```

</td></tr>
</tbody></table>

### 减少嵌套

减少嵌套! 减少嵌套! 减少嵌套!

当你看到一大堆'}}}}}'像楼梯一样连在一起，绝对会想打人的，我认为除非是多重循环,否则超过4层嵌套，肯定是可以优化的，code review要坚决打回。
（生产环境有的代码远比此处的例子更糟糕）。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for _, v := range data {
    if v.F1 == 1 {
        v = process(v)
        if err := v.Call(); err == nil {
            v.Send()
        } else {
            return err
        }
    } else {
        log.Printf("Invalid v: %v", v)
    }
}
```

</td><td>

```go
for _, v := range data {
    if v.F1 != 1 {
        log.Printf("Invalid v: %v", v)
        continue
    }

    v = process(v)
    if err := v.Call(); err != nil {
        return err
    }
    v.Send()
}
```

</td></tr>
</tbody></table>

### 用var关键字初始化零值变量

<table>
<thead><tr><th>Do</th><th>Better</th></tr></thead>
<tbody>
<tr><td>

```go
flag := false
cnt := 0
user := User{}
```

</td><td>

```go
var flag bool
var cnt int
var user User
```

</td></tr>
</tbody></table>

### 用 := 运算符初始化非零值变量
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var ans int
ans = 100
```

</td><td>

```go
ans := 100
```

</td></tr>
</tbody></table>

### 相关联变量考虑在同一行定义或赋值

<table>
<thead><tr><th>Do</th><th>Better</th></tr></thead>
<tbody>
<tr><td>

```go
var head int
var tail int

left := 0
right := len(nums) - 1
```

</td><td>

```go
var head, tail int

left, right := 0, len(nums)-1
```

</td></tr>
</tbody></table>




### 处理类型断言失败

[type assertion] 的单个返回值形式针对不正确的类型将产生 panic。因此，请始终使用“comma ok”的惯用法。

[type assertion]: https://golang.org/ref/spec#Type_assertions

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
t := i.(string)
```

</td><td>

```go
t, ok := i.(string)
if !ok {
    // 优雅地处理错误
}
```

</td></tr>
</tbody></table>

## 结构体 (Struct)
### 初始化 Struct 引用

在初始化结构引用时，请使用`&T{}`代替`new(T)`，以使其与结构体初始化一致。前面的&可以很直观地体现出取引用，而使用new关键字对不熟悉C或C++的同学可能有点奇怪（个人理解）。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
a := new(Test)

b := new(Test)
b.head = 1
b.tail = 2
```

</td><td>

```go
a := &Test{}

// 少于3个字段可考虑省略字段名
b := &Test{1, 2}
// 更加建议全部包含字段名(不然IDE也会有虚线)
b := &Test{
  head: 1,
  tail: 2,
}

```

</td></tr>
</tbody></table>

### 结构体中的嵌入

嵌入式类型应位于结构体内的字段列表的顶部，并且建议有一个空行将嵌入式字段与常规字段分隔开。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
type Client struct {
  version int
  http.Client
}
```

</td><td>

```go
type Client struct {
  http.Client

  version int
}
```

</td></tr>
</tbody></table>




## 切片/数组 (Slice/Array)
### 使用var声明空切片
个人偏爱使用var声明，认为优雅大气，此处保留意见。
<table>
<thead><tr><th>Do</th><th>Better</th></tr></thead>
<tbody>
<tr><td>

```go
arr := make([]int, 0)  // arr != nil
arr := []int{}         // arr != nil
arr := []int(nil)      // arr == nil
```

</td><td>

```go
var arr []int
```

</td></tr>
</tbody></table>

### 只拿索引的迭代方式
有时候要直接修改数组值，若元素不为指针，则只可通过arr[i] = val这样的语句修改。
还是有一部分同学使用Bad方法来写的，提醒一下。
<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
// 画蛇添足,多半不知道可以省略下划线
for i, _ := range arr {
    arr[i] = val
}
```

</td><td>

```go
for i := range arr {
    arr[i] = val
}

for i := 0; i < len(arr); i++ {
    arr[i] = val
}
```

</td></tr>
</tbody></table>

### 二维数组的定义和初始化
Go语言定于并初始化二维数组是比较麻烦的，两种方法差不多，go语言源码都有使用。
个人偏爱第二个，写起来非常之快。若是对i有要求的话，第一种更直观和方便。
<table>
<thead><tr><th>Do</th><th>Or</th></tr></thead>
<tbody>
<tr><td>

```go
matrix := make([][]int, m)
for i := 0; i < n; i++ {
    matrix[i] = make([]int, n)
}
```

</td><td>

```go
matrix := make([][]int, m)
for i := range matrix {
    matrix[i] = make([]int, n)
}
```

</td></tr>
</tbody></table>

### 指定切片容量

在尽可能的情况下，在使用`make()`初始化切片时提供容量信息。

```go
make([]T, length, capacity)
```

与map不同，slice capacity不是一个提示：编译器将为提供给`make()`的slice的容量分配足够的内存，这意味着后续的append()操作将导致零内存分配。
（直到slice的长度与容量匹配，在此之后，任何append都可能调整大小以容纳其他元素）。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
for n := 0; n < b.N; n++ {
    data := make([]int, 0)
    for k := 0; k < size; k++{
        data = append(data, k)
    }
}
```

</td><td>

```go
for n := 0; n < b.N; n++ {
    data := make([]int, 0, size)
    for k := 0; k < size; k++{
        data = append(data, k)
    }
}
```

</td></tr>
<tr><td>

```
BenchmarkBad-4    100000000    2.48s
```

</td><td>

```
BenchmarkGood-4   100000000    0.21s
```

</td></tr>
</tbody></table>

## 哈希表 (Map)
### 初始化 Maps

对于空 map 请使用 `make(..)` 初始化， 并且 map 是通过编程方式填充的。
这使得 map 初始化在表现上不同于声明，并且它还可以方便地在 make 后添加大小提示。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
var (
    // m1 读写安全;
    // m2 在写入时会 panic
    m1 = map[T1]T2{}
    m2 map[T1]T2
)
```

</td><td>

```go
var (
    // m1 读写安全;
    // m2 在写入时会 panic
    m1 = make(map[T1]T2)
    m2 map[T1]T2
)
```

</td></tr>
<tr><td>

声明和初始化看起来非常相似的。

</td><td>

声明和初始化看起来差别非常大。

</td></tr>
</tbody></table>

在尽可能的情况下，请在初始化时提供 map 容量大小，详细请看 [指定Map容量提示](#指定Map容量提示)。


另外，如果 map 包含固定的元素列表，则使用 map literals(map 初始化列表) 初始化映射。


<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
m := make(map[T1]T2, 3)
m[k1] = v1
m[k2] = v2
m[k3] = v3
```

</td><td>

```go
m := map[T1]T2{
    k1: v1,
    k2: v2,
    k3: v3,
}
```

</td></tr>
</tbody></table>

基本准则是：在初始化时使用 map 初始化列表 来添加一组固定的元素。否则使用 `make` (如果可以，请尽量指定 map 容量)。
## 字符串 (String)
### 使用原始字符串字面值，避免转义

Go 支持使用 [原始字符串字面值](https://golang.org/ref/spec#raw_string_lit)，也就是 " ` " 来表示原生字符串，在需要转义的场景下，我们应该尽量使用这种方案来替换。

可以跨越多行并包含引号。使用这些字符串可以避免更难阅读的手工转义的字符串。

<table>
<thead><tr><th>Bad</th><th>Good</th></tr></thead>
<tbody>
<tr><td>

```go
wantError := "unknown name:\"test\""
```

</td><td>

```go
wantError := `unknown error:"test"`
```

</td></tr>
</tbody></table>

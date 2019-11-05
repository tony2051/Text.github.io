---
title: Go ==的新理解
date: 2019-11-05 18:11:40
categories:
- Go
tags:
- Go
---

`==`操作是用来比较数据的值是否相等，有一些细节需要注意,首先就要对数据类型进行区分

Go 中的数据类型分为以下四类:
* 基本类型：整型、浮点数、字符串、布尔型和复数
* 聚合类型：数组和结构体
* 引用类型：切片、map、channel、指针
* 接口类型：error 等

**`==`操作最重要的前提就是：两个变量的数据类型必须相同**
注：

* Go中没有隐式类型转换
* Go可以通过`type`定义新的类型，但是新类型与底层类型不同，不能直接比较

```go
package main

import "fmt"

func main() {
    type int8 myint8
    var a int8
    var b myint8
    // 编译错误：invalid operation a == b (mismatched types int8 and myint8)
    fmt.Println(a == b)
}
```
上述定义了底层为`int8`新的类型 `myint8`，但是`int8`和`myint8`并不是相同类型

以下是不同数据类型中使用`==`需要注意的一些地方

### 基本数据类型
浮点数的比较问题
```go
var a float64 = 0.1
var b float64 = 0.2
var c float64 = 0.3
fmt.Println(a + b == c) // false
fmt.Println(a + b) // 0.30000000000000004
```
在计算机中有些浮点数不能精确表示，浮点数运算结果会存在误差，所以在计算时尽量不要做浮点数比较。

### 聚合类型
在Go中，聚合类型只有结构体和数组两种类型，他们用`==`比较的时候是逐`字段/元素`比较的。
```
注意：数组的长度视为类型的一部分，长度不同的两个数组是不同的类型，不能直接比较。
```
数组的比较：依次比较各个元素的值，对每个元素类型在进行比较，只有所有元素都相等，那么这两个数组才是相等的
结构体比较：依次比较各个字段的值和类型，所有字段都相等，结构体才相等
```go
a := [4]int{1, 2, 3, 4}
b := [4]int{1, 2, 3, 4}
c := [4]int{1, 3, 4, 5}
fmt.Println(a == b) // true
fmt.Println(a == c) // false

type A struct {
    a int
    b string
}
aa := A { a : 1, b : "test1" }
bb := A { a : 1, b : "test1" }
cc := A { a : 1, b : "test2" }
fmt.Println(aa == bb)// true
fmt.Println(aa == cc)// false
```

### 引用类型
引用类型保存的是指向数据的地址，因此引用类型的比较就是比较两个变量地址指向的是否是同一个数据
```go
type A struct {
    a int
    b string
}

aa := &A { a : 1, b : "test1" }
bb := &A { a : 1, b : "test1" }
cc := aa
fmt.Println(aa == bb) // false
fmt.Println(aa == cc) // true
```
`channel`也是如此
```go
ch1 := make(chan int, 1)
ch2 := make(chan int, 1)
ch3 := ch1

fmt.Println(ch1 == ch2) // false
fmt.Println(ch1 == ch3) // true
```
但是`切片`和`map`是特殊的两个数据类型,他们都只能和`nil`比较。如果两个切片或者两个map比较，会直接编译报错
原因：因为切片是引用类型，它可以间接的指向自己。例如：
```go
a := []interface{}{ 1, 2.0 }
a[1] = a
fmt.Println(a)

// !!!
// runtime: goroutine stack exceeds 1000000000-byte limit
// fatal error: stack overflow
```
上述代码将切片`a`赋值给`a[1]`导致递归引用，`fmt.Println(a)`直接爆栈。切片类型还拥有长度和容量属性，如果切片的长度相同，每个元素也一样，但是他们的容量不同，是否算是相等的切片，容易争议。

`map`的值类型可能是不可比较类型，比如切片，所以`map`类型也不能比较

### 接口类型
接口类型的值称作接口值。一个接口值由两部分组成：动态类型和动态值，只有动态类型和动态值都相等，这两个变量才算相等
```go
var a interface{} = 1
var b interface{} = 2
var c interface{} = 1
var d interface{} = 1.0
fmt.Println(a == b) // false
fmt.Println(a == c) // true
fmt.Println(a == d) // false
```
a和b动态类型相同（都是int），动态值也相同（都是1，基本类型比较），故两者相等。
a和c动态类型相同，动态值不等（分别为1和2，基本类型比较），故两者不等。
a和d动态类型不同，a为int，d为float64，故两者不等。

```
注意：如果接口的动态值不可比较，强行比较会编译错误
```
如动态值是切片类型
```go
var a interface{} = []int{1, 2, 3, 4}
var b interface{} = []int{1, 2, 3, 4}
// panic: runtime error: comparing uncomparable type []int
fmt.Println(a == b)
```
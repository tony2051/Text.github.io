---
title: Go 函数和数组
date: 2019-10-29 09:44:15
categories:
- Go
tags:
- Go
---

在Go语言中，函数与方法有明确的概念区分。
`函数`是指<u>*不属于任何结构体*</u>、<u>*类型*</u>的`方法`，即函数是没有接收者的，而`方法`是有接收者的。


```go
// Add 就是一个函数
func Add(a int)int{
    return a++
    }
```
`func`和方法名之间增加的参数`(p person)`就是接收者。现在类型`person`就有了一个`String`方法
```go
type person struct{
    name string
}

func (p person) String() string{
    return "person name is " + p.name
}
```
`值接收者`与`指针接收者`
在调用方法的时候，传递的接收者本质都是`副本`,只不过一个是`值副本`，一个是`指针副本`。指针具有`指向原始值`的特性,所以修改了指针指向的值，也就修改了原有的值。

##### **`多值返回`**
```go
func add(a,b int)(int,error){
    return a+b,nil
}
```

---
title: Go Map
date: 2019-10-29 09:44:15
categories:
- Go
tags:
- Go
---
#### **`内部实现`**
`Map`是基于`散列表即Hash表`来实现的，所以每次迭代Map的时候，Key和Value都是无序的。
*`Map`的散列表包含一组桶，每次存储和查找键值对的时候，都要先选择一个桶。如何选择桶呢？就是把指定的键传给`散列函数`，就可以索引到相应的桶了，进而找到对应的键值。*

#### **`声明和初始化`**
`Map`的创建有 `make` 函数
```go
// 键类型为 string ， 值类型为 int
dict := make(map[string]int)
```
使用`map字面量`方式创建和初始化map
```go
dict := map[string]int{"张三":23,"李四":24}
// 空 map
dict := map[string]int{}
// nil map
var dict map[string]int
```
#### **`使用Map`**
`Map`本质是一个指针，函数传递时，对map内部修改会影响原来的map
```go
dict := make(map[string]int)
dict["张三"] = 23 // 如果 张三 存在则修改其值为 23 ，否则新增该键值对
```
判断键值对是否存在
```go
dict := make(map[string]int)
age,exists := dict[“李四”]
// age 是键的值，exists是该键是否存在的 boolean 类型的变量
```
删除Map中的键值对使用`delete`函数
```go
delete(dict,"张三")// 第一个参数为要操作的 Map ，第二个是要删除的 Map 的键
```
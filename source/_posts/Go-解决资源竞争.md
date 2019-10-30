---
title: Go 解决资源竞争
date: 2019-10-29 17:33:56
categories:
- Go
tags:
- Go
---
**资源竞争**：如果两个或者多个 goruntine 在`没有相互同步`的情况下，访问某个`共享的资源`，比如同时对该资源进行读写时，就会处于相互竞争的状态，这就是并发中的资源竞争

#### **原子函数**
为了`保证并发安全`，go语言中可以使用`原子操作`。其执行过程不能被中断，这也就保证了同一时刻一个线程的执行不会被其他线程中断，也保证了多线程下`数据操作的一致性`。

##### **sync/atomic包**
在`atomic`包中对几种基础类型提供了原子操作，包括`int32`，`int64`，`uint32`，`uint64`，`uintptr`，`unsafe.Pointer`。
对于每一种类型，提供了五类原子操作分别是

* Add,增加和减少
* CompareAndSwap，比较并交换
* Swap，交换
* Load，读取
* Store，存储

##### **Load和Store并发不安全**
`Load`和`Store`操作对应与变量的原子性读写，许多变量的读写无法在一个时钟周期内完成，而此时执行可能会被调度到其他线程，无法保证并发安全。
Load <u>只保证读取的不是正在写入的值</u>，Store<u>只保证写入是原子操作</u>。
#### **互斥锁**
sync包里提供了一种`互斥型的锁`，可以让我们自己灵活的控制哪些代码，同时只能有一个goroutine访问，被sync互斥锁控制的这段代码范围，被称之为`临界区`，临界区的代码，同一时间，只能有一个goroutine访问。
```go
package main

import (
	"fmt"
	"runtime"
	"sync"
)

var (
	count int32
	wg    sync.WaitGroup
	mutex sync.Mutex
)

func main() {
	wg.Add(2)
	go incCount()
	go incCount()
	wg.Wait()
	fmt.Println(count)
}

func incCount() {
	defer wg.Done()
	for i := 0; i < 2; i++ {
		mutex.Lock()
		value := count
		runtime.Gosched()
		value++
		count = value
		mutex.Unlock()
	}
}
```
实例中，新声明了一个互斥锁`mutex sync.Mutex`，这个互斥锁有两个方法，一个是`mutex.Lock()`,一个是`mutex.Unlock()`,这两个之间的区域就是临界区，临界区的代码是安全的。我们先调用`mutex.Lock()`对有竞争资源的代码加锁，这样当一个`goroutine`进入这个区域的时候，其他`goroutine`就进不来了，只能等待，一直到调用`mutex.Unlock()` 释放这个锁为止。
#### **通道**
***通道有点像两个 goroutine 之间的管道，一个 goroutine 可以往通道里塞数据，另一个可以从里取数据***

##### **声明**
使用关键字`chan`和内置函数`make`进行初始化，第一个参数指定通道中发送和接收数据的`类型`，第二个参数指定通道的`大小`,省略时默认为0
```go
ch := make(chan int)
ch := make(chan int,0)// 等价第一个声明
ch := make(chan int,3)
```
通道用于 goroutine 之间的通信，具有发送和接收两个操作，两个操作运算符都是`<-`
```go
ch <- 2 // 发送数值2给该通道
x := <=ch // 从通道中读取值并赋给x变量
<-ch // 从通道里读取值，并忽略
```
使用`close`函数关闭通道,如果一个通道被关闭了，就不能往通道里发送数据了，否自会引起`painc`异常，但是仍然可以接受数据，若通道内没有数据，则接收到的值为`nil`
```go
close(ch)
```
##### **无缓冲通道**
**无缓冲通道**指的是通道的`大小为0`，这种类型的通道在接收前没有能力保存任何值，要求发送 goroutine 和接收 goroutine 同时准备好才能进行发送和接收操作。如果没有同时准备好的话先执行的操作就会`阻塞等待`，直到相对应的操作准备好为止，这种无缓冲通道也称之为`同步通道`
```go
func main() {
	ch := make(chan int)

	go func() {
		var sum int = 0
		for i := 0; i < 10; i++ {
			sum += i
		}
		ch <- sum
	}()
	
	fmt.Println(<-ch)

}
```
在计算`sum`和的`goroutine`没有执行完，在把值赋给通道`ch`前，`fmt.Println(<-ch)`会一直等待，所以`main`函数不会终止，只有当前的 `goroutine` 执行完成将值并发送到`ch`中后，同时`<-ch`接收到值后，才会打印出来
##### **有缓冲通道**
有缓冲通道其实是一个`队列`，这个队列最大容量就是我们使用`make`函数创建是的第二个参数，对于有缓冲的通道，向其发送操作就是向队列的`尾部插入元素`，接收操作是`删除头部元素并返回刚刚删除的元素`。
当队列满的时候，发送操作就会阻塞；当队列空的时候，接收操作就会阻塞。
有缓冲的通道，不要求发送和接收操作时同步的，相反可以`解耦发送和接收操作`。

`cap`函数返回通道的最大容量，`len`函数返回现在通道里有几个元素

例子：想获取服务端的一个数据，不过这个数据在三个镜像站点上都存在，这三个镜像分散在不同的地理位置，而我们的目的又是想最快的获取到数据。
```go
func mirroredQuery() string {
    responses := make(chan string, 3)
    go func() { responses <- request("asia.gopl.io") }()
    go func() { responses <- request("europe.gopl.io") }()
    go func() { responses <- request("americas.gopl.io") }()
    return <-responses // return the quickest response
}
func request(hostname string) (response string) { /* ... */ }
```
定义了一个容量为3的通道`responses`，然后同时发起3个并发goroutine向这三个镜像获取数据，获取到的数据发送到通道`responses`中，最后我们使用`return <-responses`返回获取到的第一个数据，也就是最快返回的那个镜像的数据。
##### **单向通道**
``` go
var send chan<- int //只能发送
var receive <-chan int //只能接收
```
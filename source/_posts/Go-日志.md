---
title: Go 日志
date: 2019-10-29 17:33:18
categories:
- Go
tags:
- Go
---
#### **`日志使用`**
使用`init`函数，可以在`main`函数执行之前对`log`进行初始化
```go
const (
	Ldate         = 1 << iota     //日期示例： 2009/01/23
	Ltime                         //时间示例: 01:23:23
	Lmicroseconds                 //毫秒示例: 01:23:23.123123.
	Llongfile                     //绝对路径和行号: /a/b/c/d.go:23
	Lshortfile                    //文件和行号: d.go:23.
	LUTC                           //日期时间转为0时区的
	LstdFlags     = Ldate | Ltime   //Go提供的标准抬头信息
)
```
`Lmicroseconds`和`Ltime`；`Lshortfile`和`Llongfile`两组都只能设置其中一个

`LUTC` 比较特殊，如果设置会把输出的日期时间转位0时区的日期时间显示，相对于东八区会减去8个小时。

设置`日志前缀`,可以使用`log.SetPrefix`指定输出日志的前缀。
```go
func init(){
	log.SetPrefix("【UserCenter】")
	log.SetFlags(log.LstdFlags | log.Lshortfile |log.LUTC)
}
```
`log`包除了有`Print`系列的函数，还有`Fatal`以及`Panic`系列的函数，其中`Fatal`表示程序遇到了致命的错误，需要退出，这时候使用Fatal记录日志后，然后程序退出，也就是说`Fatal相当于先调用Print打印日志`，然后再调用`os.Exit(1)`退出程序。

同理`Panic`系列的函数也一样，表示先使用`Print`记录日志，然后调用`panic()`函数抛出一个恐慌，这时候除非使用`recover()`函数，否则程序就会打印错误堆栈信息，然后程序终止。

#### **`定制日志`**
日志记录的根本在于一个日志记录器`Logger`，定制自己的日志，就是创建不同的`Logger`
```go
var (
	Info *log.Logger
	Warning *log.Logger
	Error * log.Logger
)

func init(){
	errFile,err:=os.OpenFile("errors.log",os.O_CREATE|os.O_WRONLY|os.O_APPEND,0666)
	if err!=nil{
		log.Fatalln("打开日志文件失败：",err)
	}

	Info = log.New(os.Stdout,"Info:",log.Ldate | log.Ltime | log.Lshortfile)
	Warning = log.New(os.Stdout,"Warning:",log.Ldate | log.Ltime | log.Lshortfile)
	Error = log.New(io.MultiWriter(os.Stderr,errFile),"Error:",log.Ldate | log.Ltime | log.Lshortfile)

}

func main() {
	Info.Println("infoLogger")
	Warning.Printf("warningLogger")
	Error.Println("errorLogger")
}
```
根据日志级别定义了三种不同的Logger，分别为`Info`,`Warning`,`Error`，用于不同级别日志的输出。这三种日志记录器都是使用`log.New`函数进行创建
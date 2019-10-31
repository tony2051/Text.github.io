---
title: Go 前端下载服务器静态文件问题
date: 2019-10-31 13:38:17
categories:
- Go
tags:
- Go
---
最近有个需求是将数据导出成指定格式的文件，浏览器下载生成的文件，通常直接返回服务器静态文件地址，前端使用`<a>`标签通过属性`href`指向地址就可以下载，但是有些特殊格式的文件浏览器会默认打开而不是下载，比如`.xes`（xml的一种拓展）还有`png`等图片格式。下面是总结了一些解决的方法。


由于存在文件可能较大的情况，不推荐流的方式传递给前端。
## **使用`<a>`标签的`download`属性**
```!
<a>标签的 dowanload 属性只有在同源情况下在能用，且目前只支持 火狐浏览器和谷歌浏览器
```
在`<a>`标签中必须设置 href 属性。
download 属性规定被下载的超链接目标。
该属性也可以设置一个值来规定下载文件的名称。所允许的值没有限制，浏览器将自动检测正确的文件扩展名并添加到文件 (.img, .pdf, .txt, .html, 等等)。
```javascript
// 请将href中的地址改为文件地址，下载下来的文件名称为 ceshi
<a href="www.baidu.com" download="ceshi">
```

## **Iris框架的 ctx.SendFile()方法**

iris框架中有封装方法 ctx.SendFile()方法可以将文件的内容返还给前端。（ps:如果文件特别大的话，内容很多感觉还是不方便）
```go
SendFile(filename string, destinationName string) error
```
两个参数，filename是目标文件路径，destinationName 是赋予文件名
```go
/* 文件目录为
— files
   —— first.xml
—— main.go
*/
package main

import (
    "github.com/kataras/iris"
)

func main() {
    app := iris.New()
    app.Get("/", func(ctx iris.Context) {
        file := "./files/first.xml"
        ctx.SendFile(file, "c.xml")
    })
    app.Run(iris.Addr(":8080"))
}
```

## **压缩指定文件**
可以将浏览器无法下载的文件压缩为`.zip`格式文件，然后将该`.zip`文件返还给浏览器，浏览器再进行下载。本人采用的就是这种方法
```go
/* 压缩文件为zip格式
* filePath 为需要压缩的文件路径，zipPath为压缩后文件路径
*/
func FileToZip(filePath string,zipPath string) error {
	f,err := os.Open(filePath)
	if err !=nil{
		return err
	}
	defer f.Close()

	z,err := os.Create(zipPath)
	if err !=nil{
		return err
	}
	defer z.Close()

	wr := zip.NewWriter(z)
	w,err := wr.Create(filePath)
	if err != nil{
		return err
	}
	_,err = io.Copy(w,f)
	if err != nil{
		return err
	}
	return nil
}
```

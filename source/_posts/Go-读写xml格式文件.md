---
title: Go 读写xml格式文件
date: 2019-10-29 17:41:32
categories:
- Go
tags:
- Go
---
### **etree插件**
地址：<a href="https://github.com/beevik/etree">https://github.com/beevik/etree</a>


etree是一个轻量型的纯go包，以元素树的形式表示XML，简化了Go解析XML格式文件的复杂度

它包含的功能和特性:
* 将XML文档表示为元素树（element tree）以便于遍历
* 以`files`、`[]byte slices`、`strings`和`io interface`读写XML
* 使用空格或者制表符自动缩进XML，提高可读性
* 底层是 go encoding/xml 


### **读取XML文档**
读取XML文档内容到 etree document中
```go
doc := etree.NewDocument()
err := doc.ReadFromFile(url)
if err !=nil {
    panic(err)
}
```

获取节点，文本值，属性以及属性值
```go
element := doc.SelectElement("log") // 查找标签名为 log 的节点
events := element.ChildElements() // 获取 element 的所有子节点
event := events[0]
event.Tag       // 获取该节点的标签名
event.Attr      // 获取该节点下所有属性
event.Attr[0].Value   // 获取该节点下第一个属性对应的值
event.Text() //获取该节点的文本值 返回值是 字符串类型
```

### **创建并写入XML文档**
从头开始创建并写入
```go
doc := etree.NewDocument()
doc.CreateProcInst("xml", `version="1.0" encoding="UTF-8"`)
doc.CreateProcInst("xml-stylesheet", `type="text/xsl" href="style.xsl"`)

people := doc.CreateElement("People")   //创建节点，标签名为 People
people.CreateComment("These are all known people") //写入文本值

jon := people.CreateElement("Person")
jon.CreateAttr("name", "Jon") // 创建属性名为 name ,对应的值为 Jon

sally := people.CreateElement("Person")
sally.CreateAttr("name", "Sally")

doc.Indent(2) // 缩进 2 个单位
err := doc.WriteToFile(path) // 输出到路径为 path 的文件内
if err !=nil {
    panic(err)
}
```
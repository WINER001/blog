---
title: go语言中分号的用法
date: 2023/5/21 19:21
tags: 
  - go语言
  
categories: 开发语言
top_img: ./img/41.jpg
cover: ./img/31.jpg
 

---

## GO语言中分号“；”的用法

```go
if err := assetChaincode.Start(); err != nil {
	log.Panicf("Error starting asset-transfer-basic chaincode: %v", err)
}
```

在给出的代码中，分号并不是分隔符，而是一个特定的 Go 语法结构。这是一种短变量声明和赋值的写法。

在这个语法结构中，`err := assetChaincode.Start()` 是一个函数调用表达式，而 `err` 是一个在该语句中被声明的新变量。这个短变量声明将函数调用的返回值赋值给了 `err` 变量。

接下来的 `if err != nil` 是一个条件语句，用于检查 `err` 变量是否为 `nil`。如果 `err` 不为 `nil`，则表示发生了错误，执行相应的错误处理逻辑。

这种写法是 Go 语言中常见的一种简洁的错误处理方式。它结合了变量声明和赋值，以及条件判断，使得我们可以在一个语句中完成错误检查和处理。
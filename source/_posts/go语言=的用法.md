---
title: go语言— :=的用法
date: 2023/5/21
tags: go语言
categories: 开发语言
top_img: ./img/41.jpg
cover: ./img/41.png

---

# 

## go语言中  :=  的用法

在Go语言中，":=" 是一种特殊的变量声明和赋值语法。它被称为短变量声明（short variable declaration）或简短声明（short declaration）。

使用":=" 可以在声明变量的同时为其赋值，而不需要显式地指定变量的类型。这种语法对于声明和初始化新的局部变量非常方便。

下面是一些使用":=" 的示例：

```go
package main

import "fmt"

func main() {
    // 声明并赋值整数类型的变量
    a := 10
    fmt.Println(a) // 输出: 10

    // 声明并赋值字符串类型的变量
    name := "Alice"
    fmt.Println(name) // 输出: Alice

    // 在同一个短变量声明中声明多个变量并赋值
    x, y := 5, 7
    fmt.Println(x, y) // 输出: 5 7

    // 使用短变量声明进行变量交换
    x, y = y, x
    fmt.Println(x, y) // 输出: 7 5
}
```

在上面的示例中，使用":=" 进行变量声明和赋值的方式非常简洁和方便。注意，":=" 只能用于函数内部的局部变量声明，不能用于全局变量。

需要注意的是，如果变量已经在同一作用域中声明过，那么":=" 将被视为赋值操作而不是声明操作。例如：

```go
package main

import "fmt"

func main() {
    a := 10
    fmt.Println(a) // 输出: 10

    a := 20 // 错误: no new variables on left side of :=
    fmt.Println(a)
}
```

在上面的示例中，第二个使用":=" 的语句会导致编译错误，因为变量"a"已经在同一作用域中声明过了，应该使用"=" 进行赋值操作。

总结起来，":=" 是Go语言中用于声明和赋值变量的简洁语法，它可以在声明变量的同时为其赋值，并且只能在函数内部使用。
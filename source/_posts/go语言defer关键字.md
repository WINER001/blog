---
title: go语言defer关键字
date: 2023/5/21 21:28
tags: go语言
categories: 开发语言
top_img: ./img/41.jpg
cover: ./img/13.jpg

---

# 

## go语言defer关键字

在 Go 语言中，`defer` 是一个关键字，用于延迟（defer）函数的执行。当一个函数内部包含 `defer` 语句时，被 `defer` 修饰的函数会在包含它的函数执行完毕后被调用，无论函数是正常返回还是发生了异常。

`defer` 语句的语法如下：

```go
defer functionCall(arguments)
```

其中，`functionCall` 是需要被延迟执行的函数调用表达式，可以包括函数名和参数。当包含 `defer` 语句的函数执行到 `defer` 语句时，不会立即执行 `functionCall`，而是将其推入一个栈中，等待函数执行完毕后逆序执行这些被延迟的函数调用。

下面是一些关于 `defer` 关键字的重要特性：

1. `defer` 语句按照后进先出（LIFO）的顺序执行，即最后一个 `defer` 语句将最先执行，而最先声明的 `defer` 语句将最后执行。
2. `defer` 语句的执行时机是在包含它的函数执行完毕后，无论是正常返回还是发生了异常。这使得我们可以在函数结束前进行一些清理工作，如关闭文件、释放资源等。
3. 如果包含 `defer` 语句的函数是一个循环体或递归函数，那么每次迭代或递归调用时都会执行 `defer` 语句，但它们的执行顺序仍然遵循后进先出的原则。
4. `defer` 语句可以用来捕获函数的返回值。在 `defer` 语句中，即使修改了函数的返回值，在 `defer` 函数被调用时仍然使用最初的返回值。

下面是一个示例，演示了 `defer` 关键字的使用：

```go
package main

import "fmt"

func main() {
    fmt.Println("Start")

    // 使用 defer 延迟执行函数
    defer fmt.Println("Deferred execution")

    fmt.Println("End")
}
```

在上述示例中，`fmt.Println("Deferred execution")` 使用了 `defer` 关键字，所以它的执行被延迟到包含它的 `main` 函数执行完毕后。因此，先打印 "Start"，然后打印 "End"，最后才会执行被 `defer` 延迟的语句打印 "Deferred execution"。这展示了 `defer` 关键字的效果。
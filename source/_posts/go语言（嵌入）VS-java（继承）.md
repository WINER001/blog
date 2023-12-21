---
title: go语言（嵌入）VS java（继承）
date: 2023/5/21 19:33
tags: 
  - go语言
  - java语言
  
categories: 开发语言
top_img: ./img/41.jpg
cover: ./img/66.jpg
 


---

## 

### 在 Go 语言中的嵌入与 Java 中的继承有一些区别。

1. ##### 嵌入类型 vs 继承关系：在 Go 中，类型嵌入被看作是组合而不是严格的继承关系。嵌入类型允许一个类型包含（嵌入）另一个类型，并获得嵌入类型的字段和方法，但并不会形成一个明确的父子关系。相比之下，Java 中的继承是一种明确的父子关系，子类继承父类的字段和方法。

2. ##### 单继承 vs 多重嵌入：Java 中的类只能单继承，即一个类只能继承自一个父类。而在 Go 中，类型可以通过多重嵌入实现多个类型的组合，一个类型可以嵌入多个其他类型。

3. ##### 方法的重写：在 Java 中，子类可以重写（覆盖）父类的方法，并使用 `@Override` 注解进行标记。在 Go 中，嵌入类型的方法可以被嵌入类型所覆盖或扩展，但没有类似于 `@Override` 的特殊注解。

4. ##### 接口的实现：在 Java 中，通过继承可以实现父类或接口的方法。在 Go 中，通过嵌入类型，一个类型可以获得其他类型的方法，但在语法上并不直接表示实现接口。需要显式声明类型满足某个接口的方法。

总的来说，Go 语言的类型嵌入与 Java 的继承有相似之处，但又有一些不同。Go 更注重组合而非严格的继承关系，通过嵌入类型可以实现代码的重用和组合，使得代码更加灵活和可扩展。



### 总的来看，最大的区别是第二点，java的继承只能有一个父类，但是go可以多重嵌入，下面是一个多重嵌入的例子：

```go
type Animal struct {
    name string
}

func (a Animal) Speak() {
    fmt.Println("Animal speaks")
}

type Dog struct {
    Animal
    breed string
}

func (d Dog) Bark() {
    fmt.Println("Dog barks")
}

type Flyer interface {
    Fly()
}

type Bird struct {
    Animal
}

func (b Bird) Fly() {
    fmt.Println("Bird flies")
}

type FlyingDog struct {
    Dog
    Bird
}

func main() {
    f := FlyingDog{}
    f.Speak() // 继承自 Animal
    f.Bark() // 继承自 Dog
    f.Fly() // 继承自 Bird
}
```

在上面的示例中，有四个类型：`Animal`、`Dog`、`Bird` 和 `FlyingDog`。

- `Animal` 类型包含一个 `name` 字段和一个 `Speak` 方法。
- `Dog` 类型嵌入了 `Animal` 类型，并添加了一个 `breed` 字段和一个 `Bark` 方法。
- `Bird` 类型嵌入了 `Animal` 类型，并实现了 `Flyer` 接口的 `Fly` 方法。
- `FlyingDog` 类型同时嵌入了 `Dog` 和 `Bird` 类型，因此它包含了 `Animal` 的字段和方法，以及 `Dog` 和 `Bird` 的字段和方法。

在 `main` 函数中，我们创建了一个 `FlyingDog` 类型的实例 `f`。通过 `f`，我们可以访问和调用嵌入类型的字段和方法。`f` 继承了 `Animal`、`Dog` 和 `Bird` 的字段和方法，可以使用 `f.Speak()`、`f.Bark()` 和 `f.Fly()` 进行调用。

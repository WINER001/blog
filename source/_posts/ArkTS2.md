---
title: ArkTS-@Builder自定义构造函数
date: 2023/6/20 21:12
tags: 
  - ArkTs
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/31.jpg


---

# 



## @Builder：自定义构建函数

前面章节介绍了如何创建一个自定义组件。该自定义组件内部UI结构固定，仅与使用方进行数据传递。ArkUI还提供了一种更轻量的UI元素复用机制@Builder，@Builder所修饰的函数遵循build()函数语法规则，开发者可以将重复使用的UI元素抽象成一个方法，在build方法里调用。

为了简化语言，我们将@Builder装饰的函数也成为“自定义构建函数”



### 装饰器使用说明

自定义组件内自定义构建函数

定义的语法

```
@Builder myBuilderFunction({...})
```

使用方法：

```
this.myBuilderFunction({...})
```

- 允许在自定义组件内定义一个或多个自定义构建函数，该函数被认为是该组件的私有，特殊类型的成员函数。
- 自定义构建函数额可以在所属组件的build方法和其他自定义构建函数中调用，但不允许在组件外调用。
- 在自定义函数体中，this指代当前所属组件，组件的状态变量可以在自定义构建函数内访问。建议通过this访问自定义组件的状态变量而不是参数传递。

### 全局自定义构建函数

定义的语法：

```
@Builder function MyGlobalBuilderFunction({...})
```

使用方法：

```
MyGlobalBuilderFunction()
```

- 全局自定义构建函数可以被整个应用获取，不允许使用this和bind方法。
- 如果不涉及组件状态变化，建议使用全局的自定义构建方法。

### 参数传递规则

自定义构建函数的参数传递有按值传递和按引用传递两种，均需遵守以下规则：

- 参数的类型必须与参数声明的类型一致，不允许undefined，null和返回undefined，null的表达式。
- 在自定义构建函数内部，不允许改变参数值。如果需要改变参数值，且同步回调用点，建议使用@Link
- @Builder内UI语法遵循UI语法规则

#### 1.按引用传递参数时，传递的参数可为状态变量，且状态变量的改变会引起@Builder方法内的UI刷新。ArkUI提供$$按引用传递参数的范式。

```
ABuilder($$:{paramA1: string,paramB1 : string});
```

```typescript
@Builder function ABuilder($$:{paramA1:string}){
    Row(){
        TExt('UseStateVarByReference:${$$.paramA1}')
    }
}
@Entry
@Component
struct Parent{
    @State label: string = 'Hello';
    build(){
        Column(){
            //在Parent组建中调用ABuilder的时候，将this.label引用传递给ABuilder
            ABuilder({paramA1: this.label})
            Button('Click me').onClick(()=>{
                //点击“Click me”后，UI从"hello"刷新为“ArkUI”
                this.label = 'ArkUI';
            })
        }
    }
}
```

#### 2.按值传递函数

调用@Builder装饰的函数默认按值传递。当传递的参数为状态变量时，状态变量的改变不会引起@Builder方法内的UI刷新。所以当使用状态变量的时候，推荐使用按饮用传递

```typescript
@Builder function ABuilder(paramA1:string){
    Row(){
        Text('UseStateVarByValue: ${paramA1}')
    }
}
@Entry
@Component
struct Parent{
    label: string = 'Hello';
    build(){
        Column(){
            ABuilder(this,label)
        }
    }
}
```


---
title: ArkTS-if-else条件渲染
date: 2023/7/11 16:12
tags: 
  - ArkTs
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/34.jpg

---



## if/else:条件渲染

ArkTS提供了渲染控制的能力。条件渲染可根据应用的不同状态，使用if,else,else if渲染对应状态下的UI内容.

### 使用规则

- 支持if,else和else if语句。
- if，else if后跟随的条件语句可以使用状态变量。
- 允许在容器组件内使用，通过条件渲染语句构建不同的子组件。
- 条件渲染语句在涉及到组件的父子关系时是“透明”的，当父组件和子组件之间存在一个或多个if语句时，必须遵守父组件关于子组件使用的规则。
- 每个分支内部的构件函数必须遵循构件函数的规则，并创建一个或多个组件。无法创建组件的空构件函数会产生语法错误。
- 某些容器组件限制子组件的类型或数量，将条件渲染语句用于这些组件内时，这些限制将同样应用于条件渲染语句内创建的组件。例如，Grid容器组件的子组件仅支持GridItem组件，在Grid内使用条件渲染语句时，条件渲染语句内仅允许使用GridItem组件。

### 更新机制

当if,else if 后跟随的状态判断中使用的状态变量值变化时，条件渲染语句会进行更新，更新步骤如下：

1.评估if和else if的状态判断条件，如果分支没有变化，请无需执行以下步骤。如果分支有变化，则执行2,3步骤：

2.删除此前构建的所有子组件。

3.执行新分支的构造函数，将获取到的组件添加到if父容器中。如果缺少适用的else分支，则不构建任何内容。

条件可以包括Typescript表达式。对于构造函数中的表达式，此类表达式不得更改应用程序状态。



### 适用场景

适用if进行条件渲染

```typescript
@Entry
@Commponent
struct ViewA{
    @State count: number = 0;
    
    build(){
        Column(){
            Text(`count=${this.count}`)
            
            if(this.count>0){
                Text(`count is positive`)
                .fontColor(Color.Green)
            }
            
            Button('increase count')
        }
    }
}
```


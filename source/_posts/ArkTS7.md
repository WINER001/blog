---
title: ArkTS-stateStyles多态样式
date: 2023/6/23 11:12
tags: 
  - ArkTs
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/28.jpg


---



## stateStyles：多态样式

@Styles和@Extend仅仅应用于静态页面的样式复用，stateStyles可以根据组件的内部状态的不同，快速设置不同样式。这就是我们本章要介绍的内容stateStyles（又称为：多态样式）

### 概述

stateStyle是属性方法，可以根据UI内部状态来设置样式，类似于css伪类，但语法不同。ArkUI提供一下四种状态：

- focused: 获焦态
- normal：正常态
- pressed：按压态
- disabled：不可用态

### 使用场景

下面的实例展示了stateStyles最基本的使用场景。Button处于第一个组件，默认获焦，生效focused指定的粉色样式。按压时显示为pressed态指定的黑色。如果在Button前再放一个组件，使其不处于获焦态，就会生效normal态的黄色。

```typescript
@Entry
@Component
struct StateStylesSamole{
    build(){
        Column(){
            Button('Click me')
            	.stateStyles({
                focused:{
                    .backgroundColor(Color.Pink)
                },
                pressed:{
                             .backgroundColor(Color.Black)
                },
                normal:{
                    .backgroundColor(Color.Yellow)
                }
            })
        }.margin('30%')
    }
}
```



### @Styles和stateStyles联合使用

以下示例通过@Styles指定stateStyles的不同状态

```typescript
@Entry
@Component
struct MyComponent{
    @Styles normalStyle(){
        .backgroundColor(Color.Gray)
    }
    
    @Styles pressedStyle(){
        .backgroundColor(Color.Red)
    }
    
    build(){
        Column(){
            Text('Text1')
            .fontSize(50)
            .fontColor(Color.White)
            .stateStyles({
                normal: this.normalStyle,
                pressed: this.pressedStyle,
            })
        }
    }
}
```

### 在stateStyles里使用常规变量和状态变量

stateStyles可以通过this绑定组件内的常规变量和状态变量。

```typescript
@Entry
@Component
struct CompWithInlineStateStyles{
    @State focusedColor: Color = Color.Red;
    normalColor: Color = Color.Green
    
    build(){
        Button('clickMe').height(100).width(100)
        	.stateStyles({
            normal:{
                .backgroundColor(this.normalColor)
            },
            focused:{
                 .backgroundColor(this.focusedColor)
            }
        })
        .onClick(()=>{
            this.focusedColor = Color.Pink
        })
		.margin('30%')
    }
}
```

Button默认获焦显示红色，点击事件触发后，获焦态变为粉色。


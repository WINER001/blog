---
title: ArkTS-@Extend定义扩展组件样式
date: 2023/6/22 12:12
tags: 
  - ArkTs
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/29.jpg


---



## @Extend：定义扩展组件样式

在前文的示例中，可以使用@Styles用于样式的扩展，在@Styles的基础上，我们提供了@Extend，用于扩展原生组件样式。

### 装饰器使用说明

语法

```
@Extend(UIComponentName) function functionName{...}
```

### 使用规则

- 和@Styles不同，@Extend仅支持在全局，不支持在组建内部定义。

- 和@Styles不同，@Extend支持封装指定的组件的私有属性和私有事件和预定义相同组件的@Extend的方法

  ```typescript
  //@Extend(Text)可以支持Text的私有属性fontColor
  @Extend(Text) function fancy(){
      .fontColor(Color.Red)
  }
  // superFancyText可以调用预定义的fancy
  @Extend(Text) function superFancyText(size:number){
      .fontSize(size)
      .fancy()
  }
  ```

- 和@Styles不同，@Extend装饰的方法支持参数，开发者可以在调用时传递参数，调用遵循TS方法传值调用。

  ```typescript
  // xxx.ets
  @Extend(Text) function fancy(fontSize: number){
      .fontColor(Color.Red)
      .fontSize(fontSize)
  }
  
  @Entry
  @Component
  struct FancyUse{
      build(){
          Row({space:10}){
              Text('Fancy')
              .fancy(16)
              Text('Fancy')
              .fancy(24)
          }
      }
  }
  ```

  

- @Extend装饰的方法的参数可以为function，作为Event时间的句柄

```typescript
@Extend(Text) function makeMeClick(onClick: ()=> void){
.backgroundColor(Color.Blue)
.onClick(onClick)
}

@Entry
@Component
struct FancyUse{
	@State label: string = 'Hello World';
    
    onClickHandler(){
        this.label = 'Hello ArkUI';
    }
    
    build(){
        Row({space:10}){
            Text('${this.label}')
            .makeMeClick(this.onClickHandler.bind(this))
        }
    }
}
```



- @Extend的参数可以为状态变量，当状态变量改变时，UI可以正常的被刷新渲染。

```typescript
@Extend(Text) function fancy(fontSize: number){
    .fontColor(Color.Red)
    .fontSize(fontSize)
}

@Entry
@Component
struct FancyUse{
    @State fontSizeValue: number = 20
    build(){
        Row({space:10}){
            Text('Fancy')
            	.fancy(this.fontSizeValue)
            	.onClick(()=>{
                this.fontSizeValue=30
            })
        }
    }
}
```



### 使用场景

以下示例声明了3个Text组件，每个Text组件均设置了fontStyle，fontWeight和backgroundColor样式

```typescript
@Entry
@Component
struct FancyUse{
	@State label: string  = 'Hello World'
    
    build(){
        Row({space:10}){
            Text('${this.label}')
            	.fontStyle(FontStyle.Italic)
            	.fontWeight(100)
            	.backgroundColor(Color.Blue)
            Text('${this.label}')
            	.fontStyle(FontStyle.Italic)
            	.fontWeight(200)
            	.backgroundColor(Color.Pink)
            Text('${this.label}')
            	.fontStyle(FontStyle.Italic)
            	.fontWeight(300)
            	.backgroundColor(Color.Orange)
        }.margin('20%')
    }
}
```

@Extend将样式组合复用，示例如下。

```typescript
@Extend(Text) function fancyText(weightValue: number, color: Color){
    .fontStyle(FontStyle.Italic)
    .fontWeight(weightValue)
    .backgroundColor(color)
}
```



通过@Extend组合样式后，使得代码更加简介，增强可读性

```typescript
@Entry
@Component
struct FancyUse{
	@State label: string = 'Hello World'
    
    build(){
        Row({ space:10 }){
            Text('${this.label}')
            	.fancyText(100,Color.Blue)
            Text('${this.label}')
            	.fancyText(200,Color.Pink)
            Text('${this.label}')
            	.fancyText(200,Color.Orange)
        }.margin('20%')
    }
}
```


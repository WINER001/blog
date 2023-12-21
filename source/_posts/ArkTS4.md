---
title: ArkTS-@Styles定义组件重用样式
date: 2023/6/21 20:12
tags: 
  - ArkTs
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/41.jpg


---



## @Styles：定义组件重用样式

### 1.装饰器使用说明

- 当前@Styles仅支持通用属性和。

- @Styles方法不支持参数，反例如下。

  ```typescript
  //反例：@Styles不支持参数
  @Styles function globalFancy(value:number){
      .width(value)
  }
  ```

- @Styles可以定义在组件内或全局，在全局定义时需在方法名前面添加function关键字，组件内定义时则不需要添加function关键字。

  ```typescript
  //全局
  @Styles function functionName(){...}
  //在组件内
  @Component
  struct FancyUse{
      @Styles fancy(){
          .height(100)
      }
  }
  ```

- 定义在组件内的@Styles可以通过this访问组件的常量和状态常量，并可以在@Styles里通过事件来改变状态变量的值，示例如下：

```typescript
@Component
struct FancyUse{
    @State heightValue: number = 100
    @Styles fancy(){
        .height(this.heightValue)
        .backgroundColor(Color.Yellow)
        .onClick(()=>{
            this.heightValue = 200
        })
    }
}
```

组件内@Styles的优先级高于全局@Styles

框优先找到组件内的@Styles，如果找不到，则会全局查找



### 2.使用场景

以下示例中演示了组件内@Styles和全局@Styles的用法

```typescript
//定义在全局的@Styles封装的样式
@Styles function globalFancy(){
    .width(150)
    .height(100)
    .backgroundColor(Color.Pink)
}

@Entry
@Component
struct fancy(){
    .width(200)
    .height(this.heightValue)
    .backgroundColor(Color.Yellow)
    .onClick(()=>{
        this.heightValue = 200
    })
}

build(){
    Column({space:10}){
        //使用全局的@Styles封装的样式
        Text('FancyA')
        	.globalFancy()
        	.fontSize(30)
        //使用组件内的@Styles封装的样式
        Text('FancyB')
        	.fancy()
        	.fontSize(30)
    }
}
```


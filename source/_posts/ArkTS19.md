---
title: ArkTS-$$语法-内置组件双向同步
date: 2023/7/11 10:12
tags: 
  - ArkTs
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/13.jpg

---



## $$语法:内置组件双向同步

$$运算符为系统内置组件提供TS变量的引用，使得TS变量和系统内置组件的内部状态保持同步。

内部状态具体指什么取决于组件。例如，bindPropup属性方法的show参数。

### 使用规则

- 当前$$支持基础类型变量，以及@State，@Link和@Prop装饰的变量。
- 当前$$支持bindPopup属性方法的show参数，Radio组件的checked属性，Refresh组件的refreshing参数。
- $$绑定的变量变化时，会触发UI的同步刷新。

### 使用示例

#### 以bindPopup属性方法的show参数为例：

```typescript
// xxx.ets
@Entry
@Component
struct bindPopupPage{
    @State customPopup: boolean = false;
    
    build(){
        Column(){
            Button('Popup')
            	.margin(20)
            	.onClick(()=>{
                this.customPopup = !this.customPopup
            })
            .bindPopup($$this.customPopup,{
                message: 'showPopup'
            })
        }
    }
}
```


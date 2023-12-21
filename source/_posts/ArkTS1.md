---
title: ArkTS-自定义组件
date: 2023/6/20 11:12
tags: 
  - ArkTs
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/34.jpg

---

# 

## 手敲选自https://developer.harmonyos.com/cn/docs/documentation/doc-guides-V3/arkts-create-custom-components-0000001473537046-V3

## ArkTS

### 一，创建自定义组件

在ArkUI中，UI显示的内容均为组件，由框架直接提供的称为系统组件，由开发者定义的称为自定义组件，在进行UI界面开发时，通常不是简单的将系统组件进行组合使用，而是需要考虑代码可复用性，业务逻辑与UI分离，后续版本演进等因素。因此，将UI和部分业务逻辑封装成自定义组件是不可或缺的能力。

#### 自定义组件有以下特点：

- 可组合：允许开发者组合使用系统组件，及其属性和方法

- 可重用：定义组件可以被其他组件重用，并座位不同的实例在不同的父组件或者容器中使用，

- 数据驱动UI更新：通过状态变量的改变，来驱动UI的刷新。

  以下实例展示了自定义组件的基本用法。

```typescript
@Component
struct HelloComponent{
	@State message: string = 'hello,world!';
    
    build(){
        //HelloComponent自定义组件组合系统组件Row和Text
        Row(){
            Text(this,message).onClick(()=>{
                //状态变量message的改变驱动UI刷新，UI从‘hello，world!’刷新为'Hello,ArkUI!'
                this.message = 'hello,ArkUI';
            })
        }
    }
}
```

HelloComponent可以在其他自定义组建中的build()函数中多次创建，实现自定义组件的重用。

```typescript
@Entry
@Component
struct ParentComponent{
    build(){
        Column(){
            Text('ArkUI message')
            HelloComponent({message:'Hello,World!'});
            Divider()
            HelloComponent({message:'你好！'});
        }
    }
}
```

#### 自定义组件的基本结构

- struct：自定义组件基于strct实现，strct+自定义组件名+{...}的组合构成自定义组件，不能有继承关系。对于struct的实例化，可以省略new。
- @Component：@Component装饰器仅能装饰struct关键字声明的数据结构。struct被@Component装饰后具备组件化的能力，需要实现build方法描述UI，一个strct只能被一个@Component装饰。

```typescript
@Component
struct MyComponent{
    
}
```

- build()函数：build（）函数用于定义自定义组件的声明式UI描述，自定义组件必须定义build（）函数。

  ```typescript
  @Component
  struct MyComponent{
      build(){
          
      }
  }
  ```

- @Entry:@Entry装饰的自定义组件将作为UI页面的入口。在单个UI页面中，最多可以使用@Entry装饰一个自定义组件。

  @Entry可以接受一个可选的LocalStorage的参数。（从API version9开始，该装饰器支持在ArkTS卡片中使用。）

  ```typescript
  @Entry
  @Component
  struct MyComponent{
      
  }
  ```

#### 成员函数/变量

自定义组件除了必须要实现build()函数外，还可以实现其他成员函数，成员函数具有以下约束：

- 不支持静态函数
- 成员函数的访问始终是私有的。

自定义组件可以包含成员变量，成员变量具有以下约束：

- 不支持静态成员变量
- 所有成员变量都是私有的，变量的访问规则与成员函数的访问规则相同。
- 自定义组件的成员变量初始化有些是可选的，有些是必须的。具体是否需要本地初始化，是否需要从父组件通过参数传递初始化子组件的成员变量，请参考状态管理



#### 自定义组件的参数规定

从上文的示例中，我们已经了解到，可以在build方法或者@Builder装饰的函数里创建自定义组件，在创建的过程中，参数可以被提供给组件

```typescript
@Component
struct MyComponent{
    private countDownFrom: number = 0;
    private color: Color = Color.Blue;
    
    build(){
        
    }
}

@Entry
@Component
struct ParentComponent{
    private someColor: Color = Color.Pink;
    
    build(){
        Column(){
            //创建MyComponent实例，并将创建MyComponent成员变量countDownFrom初始化为10，将成员变量color初始化为this.someColor
            MyComponent({countDownFrom: 10, color: this.someColor})
        }
    }
}
```

build()函数

所有声明在build()函数的语言，我们称为UI描述语言，UI描述语言需要遵循以下规则：

- @Entry装饰的自定义组件，其build()函数下的根节点唯一且必要，且必须为容器组件，其中ForEach禁止作为根节点。

  @Component装饰的自定义组件，其build()函数下的根节点唯一且必要，可以为非容器组件，其中ForEach禁止座位根节点。

  ```typescript
  @Entry
  @Component
  struct MyComponent{
      build(){
          //根节点唯一且必要，必须为容器组件
          Row(){
              ChildComponent()
          }
      }
  }
  
  @Component
  struct ChildComponent{
      build(){
          //根节点唯一且必要，可作为非容器组件
          Image('test.jpg')
      }
  }
  ```

- 不允许声明本地变量，反例如下。

  ```typescript
  build(){
      //反例：不允许声明本地变量
      let a: number = 1;
  }
  ```

- 不允许在UI描述里直接使用console.info,但允许在方法或者函数里使用，反例如下。

  ```typescript
  build(){
      //反例：不允许console.info
      console.info('print debug log');
  }
  ```

- 不允许创建本地的作用域，反例如下。

  ```typescript
  build(){
      //反例：不允许本地作用域
      {
          。。。
      }
  }
  ```

- 不允许调用除了被@Builder装饰以外的方法，允许系统组件的参数是TS方法的返回值。

  ```typescript
  @Component
  struct ParentComponent{
      doSomeCalculations(){
      }
      
      calcTextValue(): string{
          return 'Hello World';
      }
      
      @Builder doSomeRender(){
          Text('Hello World')
      }
      
      build(){
          Column(){
              //反例：不能调用没有用@Builder装饰的方法
              this.doSomeCalculations();
              //正例：可以调用
              this.doSomeRender();
              //正例：参数可以为调用TS方法的返回值
              Text(this.calcTextValue())
          }
      }
  }
  ```

- 不允许switch语法，如果需要使用条件判断，请使用if。反例如下。

  ```typescript
  build(){
      Column(){
          //反例：不允许使用switch语法
          switch(expression){
              case 1:
                  Text('...')
                  break;
              case 2:
                  Image('...')
                  break;
              default:
                  Text('...')
                  break;
          }
      }
  }
  ```

- 不允许使用表达式，反例如下。

  ```typescript
  build(){
      Column(){
          //反例：不允许使用表达式
          (this.aVar > 10) ? Text('...') : Image('...')
      }
  }
  ```



#### 自定义组件通用样式

自定义组件通过“.”链式调用的形式设置通用样式。

```typescript
@Component
struct MyComponent2{
    build(){
        Button('Hello World')
    }
}

@Entry
@Component
struct MyComponent{
    build(){
        Row(){
            MyComponent2()
            	.width(200)
            	.height(300)
            	.backgroundColor(Color.Red)
        }
    }
}
```

#### 自定义属性方法

自定义组件不支持提供自定义属性方法，可以借助类似Controller控制器能力，提供自定义接口

```typescript
//自定义controller
export class MyComponentController{
    item: MyComponent = null;
    
    setItem(item: MyComponent){
        this.item = item;
    }
    
    changeText(value:string){
        this.item.value = value;
    }
}

//自定义组件
@Component
export default struct MyComponent{
    public controller: MyComponentController = null;
    @State value: string = 'Hello World';
    
    build(){
        Column(){
            Text(this.value)
            	.fontSize(50)
        }
    }
    
    aboutToAppear(){
        if(this.controller)
            this.controller.setItem(this);//绑定controller
    }
}

//使用外逻辑
@Entry
@Component
struct StyleExample{
    controller = new MyComponentController();
    
    build(){
        Column(){
            MyComponent({ controller: this.controller })
        }
        .onClick(()=>{
            this.controller.changeText('Text');
        })
    }
}
```

在上面的示例中：

1.通过子组件MyComponent的aboutToAppear方法，把当前的this指针传递给MyComponentController的item成员变量。

2.在StyleExample父组件中持有controller实例，调用controller的changeText方法，及相当于通过controller持有的MyComponent子组件的this指针，改变MyComponent的状态变量value的值。



通过controller的封装，MyComponent对外暴露了changeText的接口，所有持有controller的实例都可以通过调用changeText接口，改变MyComponent的状态变量value的值。



### 二，页面和自定义组件生命周期

在开始之前，我们先明确自定义组件和页面的关系：

- 自定义组件：@Component装饰的UI单元，可以组合多个系统组件实现UI的服用。
- 页面：即应用的UI页面。可以由一个或者多个自定义组件组成，@Entry装饰的自定义组件为页面的入口组件，即页面的根节点，一个页面有且仅能有一个@Entry。只有被@Entry装饰的组件才可以调用页面的生命周期。

页面生命周期，即被@Entry装饰的组件生命周期，提供一下生命周期接口：

- onPageShow：页面每次显示时触发。
- onPageHide：页面每次隐藏时触发一次。
- onBackPress：当用户点击返回按钮时触发。

组件生面周期，即一般用@Component装饰的自定义组件的生命周期，提供一下生命周期金额口：

- aboutToAppear：组件即将出现时回调该接口，具体时机为在创建自定义组件的新实例后，在执行其build（）函数之前执行。

- aboutToDisappear：在自定义组件即将析构销毁时执行。

  生命周期流程如下图所示，下图展示的是被@Entry装饰的组件（首页）生命周期

根据上面的流程图，我们从自定义组件的初始创建，重新渲染和删除来详细解释。

#### 自定义组件的创建和渲染流程

1.自定义组件的创建：自定义组件的实例由ArkUI框架创建。

2.初始化自定义组件的成员变量：通过本地默认值或者构造方法传递参数来初始化自定义组件的成员变量，初始化顺序为成员变量的定义顺序。

3.如果开发者自定义了aboutToAppear，择执行aboutToAppear方法。

4.在首次渲染的时候，执行build方法渲染系统组件，如果有自定义子组件，择创建自定义组件的实例。在执行build（）函数的过程中，框架会观察每个状态变量的读取状态，将保存两个map：

a，状态变量	->	UI组件（包括ForEach和if）。

b，UI组件 -> 此组件的更新函数，即一个lambda方法，作为build()函数的子集，创建对应的UI组件并执行其属性方法，示意如下。

```typescript
build(){
    ...
    this.observeComponentCreation(()=>{
        Button.create();
    })
    
    this.observeComponentCreation(()=>{
        Text.create();
    })
    ...
}
```

当应用在后台启动时，此时应用进程并没有销毁，所以仅需要执行onPageShow。



#### 自定义组件重新渲染

当事件句柄被触发（比如设置了点击事件，即触发点击事件）改变了状态变量时，或者LocalStorage/AppStorage中的属性更改，并导致绑定的状态变量更改其值时：

1.框架观察到了变化，将启动重新渲染

2.根据框架持有的两个map（自定义组件的创建和渲染流程中第四步），框架可以知道该状态变量管理了哪些UI组件，以及这些UI组件对应的更新函数。执行这些UI组件的更新函数，实现最小化更新。



#### 自定义组件的删除

如果if组件的分支改变，或者ForEach循环渲染中数组的个数改变，组件将被删除：

1.在删除组件之前，将调用其aboutToDisappear生命周期函数，标记着该节点将要被销毁。ArkUI的节点删除机制时：后端节点直接从组件树上摘下，后端节点被销毁，对前端节点解引用，当前端节点已经没有引用时，将被JS虚拟机垃圾回收。

2.自定义组件和它的变量将被删除，如果其有同步的变量，比如@Link，@Prop，@StorageLink，将从同步源上取消注册。



不建议在生命周期aboutToDisappear内使用async await，如果在生命周期的aboutToDisappear使用异步操作（Promise或者回调方法），自定义组件将被保留在Promise的闭包中，知道回调方法被执行完，这个行为阻止了自定义组件的垃圾回收。

以下示例展示了生命周期的调用时机：

```typescript
//Index.ets
import router from '@ohos.router';

@Entry
@Component
struct MyComponent{
    @State showChild: boolean = true;
    
    //只有被@Entry装饰的组件才可以调用页面的生命周期
    onPageShow(){
        console.info('Index onPageShow');
    }
    //只有被@Entry装饰的组件才可以调用页面的生命周期
    onPageHide(){
        console.info('Index onPageHide');
    }
    //只有被@Entry装饰的组件才可以调用页面的生命周期
    onBackPress(){
        console.info('Index onBackPress');
    }
    
    //组件生命周期
    aboutToAppear(){
        console.info('MyComponent aboutToAppear');
    }
    //组件生命周期
    aboutToDisappear(){
        console.info('MyComponent aboutToDisappear');
    }
    
    build(){
        Column(){
            //tis.showChild为true，创建Child子组件，执行Child aboutToAppear
            if(this.showChild){
                Child()
            }
            //this.showChild为false,删除Child子组件，执行Child aboutToDisappear
            Button('create or delete Child').onClick(()=> {
                this.showChild = false;
            })
            //push到Page2页面，执行onPageHide
            Button('push to next page')
            	.onClick(()=>{
                router.pushUrl({url:'pages/Page2'})
            })
        }
    }
}

@Component
struct Child{
    @State title: string = 'Hello World';
    //组件生命周期
    aboutToAppear(){
        console.info('[lifeCycle] Child aboutToAppear')
    }
    
    build(){
        Text(this.title).fontSize(50).onClick(()=>{
            this.title = 'Hello ArkUI';
        })
    }
}
```

以上示例中，Index页面包含两个自定义组件，一个是被@Entry装饰的MyComponent，也是页面的入口组件，即页面的根节点；一个时Child，是MyComponent的子组件。只有@Entry装饰的节点才可以生效页面的生命周期方法，所以MyComponent中声明了当前Index页面的页面生命周期函数。

- 应用冷启动的初始化流程为：MyComponent aboutToAppear -->MyComponent build --> Child aboutToAppear --> Child build --> Child build执行完毕 --> MyComponent build执行完毕 -->Index onPageShow。
- 点击“delete Child”，if绑定的this.showChild变成false，删除Child组件，会执行Child aboutToDisappear方法。
- 点击“push to next page”,调用router.pushUrl接口，跳转到另外一个页面，当前Index页面隐藏，执行页面生命周期Index onPageHide。此处调用的是router.pushUrl接口，Index页面被隐藏，并没有销毁，所以只调用onPageHide。跳转到新页面后，执行初始化新页面的生命周期的流程。
- 如果调用的是router.replaceUrl，择当前Index页面被销毁，执行的生命周期流程将变成为：Index onPageHide --> MyComponent aboutToDisappear --> Child aboutToDisappear。上文已经提到，组件的销毁是从组件树上直接摘下子树，所以先调用父组件的aboutToDisappear，再调用子组件的aboutToDisAppear，然后执行初始化新页面的生命周期流程。
- 点击返回按钮，触发页面生命周期Index onBackPress。最小化应用或者应用进入后台，触发Index onPageHide。这两个状态下应用都没有被销毁，所以并不会执行组件的aboutToDisappear。应用回到前台，执行Index 哦那PageShow。
- 推出应用，执行Index onPageHide --> MyComponent aboutToDisappear --> Child aboutToDisappear




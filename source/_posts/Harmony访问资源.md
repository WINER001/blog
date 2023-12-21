---
title: HarmonyOS访问资源
date: 2023/11/23 10:12
tags: 
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/66.jpg

---



# HarmonyOS访问资源



## 1.访问JS模块资源: $r ()

在应用开发的hml和js文件中使用$r的用法，可以对JS模块内的resources目录下的json资源进行格式化，获取相应的资源内容。

| 属性 | 类型                   | 描述                                                         |
| ---- | ---------------------- | ------------------------------------------------------------ |
| $r   | (key:string) => string | 获取资源限定下具体的资源内容例如：this.$r ('strings.hello')<br />参数说明：key:定义在资源限定文件中的键值，如strings.hello<br />{<br/>	strings:{<br/>		hello:'hello world'<br/>	}<br/>} |



### 示例

#### resources/res-dark.json:

```json
{
	"image":{
        "clockFace": "common/dark_face.png"
    },
    "colors":{
        "background":"#000000"
    }
}
```

#### resources/res-defaults.json

```json
{
    "image":{
        "clockFace":"common/face.png"
    },
    "colors":{
        "background":"#ffffff"
    }
}
```



```html
<!-- xxx.hml -->
<div style="hackground-color: {{ $r ('colors.background') }}">
    <image src="{{ $r ('image.clockFace') }}"></image>
</div>
```



## 2.访问应用资源

在hml/css/json文件中，可以引用应用资源，包括颜色，圆角和图片类型的资源。

应用资源由开发者在resources目录中定义，目前仅支持使用在color.json中自定义的颜色资源，在float.json中自定义的圆角资源以及在media目录中的图片资源。

resources目录的基础机构如下图所示，同一个资源，可以在base子目录和dark子目录各定义一个值。浅色模式时用base目录下定义的值，深色模式下用dark目录下定义的值，若其资源尽在base目录中有定义，则其在深浅色模式下的表现相同。

```
  ├─ java
  ├─ js
  └─ resources   -> 与java、js目录同级的resources目录
      ├─ base    -> 定义浅色模式下的颜色、圆角或图片
      │   ├─ element 
      │   │   ├─ color.json
      │   │   └─ float.json
      │   └─ media
      │       └─ my_background_image.png
      └─ dark    -> 定义深色模式下的颜色、圆角或图片（如果未定义，则深色模式下继续使用base目录中的相关定义）
          ├─ element 
          │   ├─ color.json
          │   └─ float.json
          └─ media
              └─ my_background_image.png
```



color.json文件的格式如下。

```
{
    "color": [
        {
            "name": "my_background_color",
            "value": "#ffff0000"
        },
        {
            "name": "my_foreground_color",
            "value": "#ff0000ff"
        }
    ]
}
```



float.json文件的格式如下

```
{
    "float":[
        {
            "name":"my_radius",
            "value":"28.0vp"
        },
        {
            "name":"my_radius_xs",
            "value":"4.0vp"
        }
    ]
}
```



#### 在卡片工程的css文件中，通过“@app.type.resource_id”的形式引用应用资源。根据引用的资源类型不同，“type”可以取“color”（颜色）、“float”（圆角）和“media”（图片）。“resource_id”代表应用资源id，即color.json或float.json中的“name”字段，或者media目录中的图片文件的名称（不包含图片类型后缀）。

```
.divA {
    background-color: "@app.color.my_background_color";
    border-radius: "@app.float.my_radius";
}
.divB {
    background-image: "@app.media.my_background_image";
}
```

#### 

```
<div style="background-color:{{$r ('app.color.my_background_color')}};"></div>
<div style="border-radius:{{$r ('app.float.my_radius')}};"></div>
<div style="background-image:{{$r ('app.media.my_background_image')}};"></div>
```

#### 在json文件中，通过“this.$r('app.type.resource_id')”的形式引用应用资源，各个字段的含义与css文件相同。

```
{
    "data":{
        "myColor": "this.$r ('app.color.my_background_color')",
        "myRadius": "this.$r ('app.float.my_radius')",
        "myImage":"this.$r ('app.media.my_background_image')"
    }
}
```



## 3.访问系统资源



#### 在hml/css/json文件中，可以引用系统预置资源，包括颜色、圆角和图片类型的资源。

#### 在卡片工程的css文件中，通过“@sys.type.resource_id”的形式引用系统资源。根据引用的资源类型不同，“type”可以取“color”（颜色）、“float”（圆角）和“media”（图片）。“resource_id”代表系统资源id，系统资源预置在系统中

```
.divA {
    background-color: "@sys.color.fa_background";
    border-radius: "@sys.float.fa_corner_radius_card";
}
.divB {
    background-image: "@sys.media.fa_card_background";
}
```



#### 

```
<div style="background-color:{{$r ('sys.color.fa_background')}};"></div>
<div style="border-radius:{{$r ('sys.float.fa_corner_radius_card')}};"></div>
<div style="background-image:{{$r ('sys.media.fa_card_background')}};"></div>
```



#### 在json文件中，通过“this.$r('sys.type.resource_id')”的形式引用系统资源，各个字段的含义与css文件相同。

```
{
    "data":{
        "sysColor": "this.$r ('sys.color.fa_background')",
        "sysRadius": "this.$r ('sys.float.fa_corner_radius_card')",
        "sysImage":"this.$r ('sys.media.fa_card_background')"
    }
}
```


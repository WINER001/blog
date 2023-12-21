---
title: HarmonyOS调用百度地图SDK
date: 2023/12/14 10:12
tags: 
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/41.png

---



# HarmonyOS调用百度地图SDK



## 一，工程配置

#### 1.权限配置

在config.json文件中配置HarmonyOS轻量地图SDK所需的相关权限，确保SDK可以正常使用。配置如下：

```js
"reqPermissions":[
    {
        "name":"ohos.permission.INTERNET",
        "reason":"use network"
    },
    {
        "name":"ohos.permission.GET_NETWORK_INFO",
        "reason":"gee network info"
    },
    {
        "name":"ohos.permission.GET_BUNDLE_INFO",
        "reason":"get bundle info"
    },
]
```



#### 2.添加HarmonyOS轻量地图SDK开发包

将har包放入libs目录下，在build.gradle中配置如下：

```js
dependencies{
    implemention fileTree(dir:'libs',include:['*.jar',".har"])
}
```

同步gradle



#### 3.添加三方库依赖

工程的build.gradle中Gson三方库的依赖，配置如下：

```
dependencies{
	implementation fileTree(dir:'libs',include:['*.jar','*.har'])
	implementation 'com.google.code.gson:gson:2.8.8'
}
```



#### 4.获取HarmonyOS应用的appId

注意：在真机运行下获取appId，使用云真机获取到的appId信息不全，会导致SDK鉴权失败，地图功能无法正常使用。正确的appId格式应该为：
包名_签名相关信息。例如：

```
com.baidu.map.demo_AAxy8/bvxxfNHWGXw9EPD/IAE/gCX/Vpy3Htu5YAsQOSnqSRahEl/zszGCunwxvDxoDMrQ+yVJCogPi7kMSouow= 
```

在Ability中调用如下代码来获取appId：

```java
//根据给定的bundle名称获取BundleInfo
//使用此方法需要申请ohos.permission.GET_BUNDLE_INFO权限
BundleInfo info = getBundleManager().getBundleInfo(getBundleName(),0);

//获取appId
String appId = info.getAppId();
```



#### 5.申请AK

申请所需参数：包名+appId。联系百度地图开放平台

https://lbs.baidu.com/apiconsole/userflow/register





## 二，显示地图

### Hello BaiduMap

百度地图SDK为开发者提供了便携的使用百度地图能力的接口，通过以下几步操作，即可在应用中使用百度地图：

### HarmonyOS轻量地图SDK初始化

#### 1.第一步 在MyApplication初始化SDK，如下：

```
SDKInitializer.initialize(this,"your API_KEY");
```

#### 2.第二步 创建mapView

```java
public class ShowMapAbility extends Ability{
    //布局
    private PositionLayout rootLayout;
    
    //mapView
    private MapView mMapView;
    
    //mapView控制器
    private BaiduMap mBaiduMap;
    
    @Override
    public void onStart(Intent intent){
        super.onStart(intent);
        //布局
        initPositionLayout();
        addMapView();
        super.setUIContent(this.rootLayout);
    }
    
    @Override
    public void onActive(){
        super.onActive();
    }
    
    @Override
    public void onForeground(Intent intent){
        super.onForeground(intent);
    }
    
    @Override
    protected void onStop(){
        super.onStop();
        mMapView.onDestroy();
    }
    
    private void initPositionLayout(){
        this.rootLayout = new PositionLayout(this);
        this.rootLayout.setContentPosition(0,0);
        this.rootLayout.setWidth(ComponentContainer.LayoutConfig.MATCH_PARENT);
        this.rootLayout.setHeight(ComponentContainer.LayoutConfig.MATCH_PARENT);
        //背景色
        ShapeElement shapeElement = new ShapeElement();
        shapeElement.setShape(ShapeElement.ALPHA_MIN);
        shapeElement.setRgbColor(new RgbColor(255,255,255));
        this.rootLayout.setBackground(shapeElement);
    }
    
    private addMapView(){
        //创建mapView
        mMapView = new MapView(getContent());
        mMapView.setPosition(0,0);
        mMapView.setWidth(ComponentContainer.LayoutConfig.MATCH_PARENT);
        mMapView.setHeight(ComponentContainer.LayoutConfig.MATCH_PARENT);
        this.rootLayout.addComponent(mMapView);
        mBaiduMap = mMapView.getMap();
    }
}
```



百度地图官方文档参考https://lbs.baidu.com/faq/api?title=harmonyossdk/guide/create-project/project

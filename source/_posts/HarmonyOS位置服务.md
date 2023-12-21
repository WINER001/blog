---
title: HarmonyOS位置服务
date: 2023/11/17 11:12
tags: 
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/41.png

---



# HarmonyOS位置服务（@ohos.geoLocationManager)



## 1.geoLocationManager.on('locationChange')

#### 开启位置变化订阅，并发起定位请求on(type:'locationChange',request:LocationRequest,callback: Callback<Location>):void

#### 需要权限：ohos.permission.APPROXIMATELY_LOCATION

#### 系统能力：SystemCapability.Location.Core

#### 参数：

| 参数名   | 类型               | 必填 | 说明                                                 |
| -------- | ------------------ | ---- | ---------------------------------------------------- |
| type     | string             | 是   | 设置时间类型。type为“locationChange”，表示位置变化。 |
| request  | LocationRequest    | 是   | 设置位置请求参数。                                   |
| callback | Callback<Location> | 是   | 接收位置变化状态变化监听。                           |

#### 错误码：

| 错误码ID | 错误信息                                    |
| -------- | ------------------------------------------- |
| 3301000  | Location service is unavailable.            |
| 3301100  | Thelocation switch is off.                  |
| 3301200  | Failed to obtain the geographical location. |

#### 示例：

```typescript
import geoLocationManager from '@ohos.geoLocationManager';
let requestInfo = {'priority': 0x203, 'scenario': 0x300, 'timeInterval': 0, 'distanceInterval': 0, 'maxAccuracy': 0};
let locationChange = (location) => {
    console.log('locationChanger: data: ' + JSON.stringify(location));
};
try{
    geoLocationManager.on('locationChange',requestInfo,locationChange);
}catch(err){
    console.error("errCode:"+err.code+",errMessage:" + err.message);
}
```



## 2.geoLocationManager.off('locationChange')

### off(type:'locationChange',callback?:<Location>):void

#### 关闭位置变化订阅，并删除对应的走位请求。

#### 需要权限：ohos.permission.APPROXIMATELY_LOCATION

#### 系统能力：SystemCapability.Location.Location.Core

#### 参数：

| 参数名   | 类型               | 必填 | 说明                                                         |
| -------- | ------------------ | ---- | ------------------------------------------------------------ |
| type     | string             | 是   | 设置事件类型。type为"locationChange",表示位置发生变化        |
| callback | Callback<Location> | 否   | 需要取消订阅的回调函数。若无此参数，则取消当前类型的所有订阅。 |

#### 错误码：

| 错误码ID | 错误信息                                    |
| -------- | ------------------------------------------- |
| 3301000  | Location service is unavailable.            |
| 3301100  | The location switch is off.                 |
| 3301200  | Failed to obtain the geographical location. |

#### 示例：

```typescript
import geoLocationManager from '@ohos.geoLocationManager';
let requestInfo = {'priority': 0x203,'scenario': 0x300,'timeInterval': 0,'distanceInterval': 0,'maxAccuracy':0};
let locationChange = (location) => {
    console.log('locationChanger: data: '+JSON.stringify(location));
};
try{
    geoLocationManager.on('locationChange',requestInfo,locationChange);
    geoLocationManager.off('locationChange',locationChange);
}catch(err){
    console.error("errCode:"+err.code+",errMessage:"+err.message);
}
```



## 3.geoLocationManager.on('locationEnabledChange')

#### on(type: 'locationEnabledChange',callback: Callback<boolean>):void

#### 订阅位置服务状态变化。

#### 系统能力：SystemCapability.Location.Location.Core

#### 参数：

| 参数名   | 类型              | 必填 | 说明                                                         |
| -------- | ----------------- | ---- | ------------------------------------------------------------ |
| type     | string            | 是   | 设置事件类型。type为”locationEnabledChange“，表示位置服务状态 |
| callback | Callback<boolean> | 是   | 接受位置服务状态变化监听。                                   |

#### 错误码：

| 错误码ID | 错误信息                         |
| -------- | -------------------------------- |
| 3301000  | Location service is unavailable. |

#### 示例：

```typescript
import geoLocationManager from '@ohos.geoLocationManager';
let locationEnabledChange = (state) =>{
    console.log('locationEnableChange: '+JSON.stringify(state));
}
try{
    geoLocationManager.on('locationEnabledChange',locationEnableChange);
}catch(err){
    console.error("errCode:"+err.code + ",errMessage:"+err.message);
}
```



## 4.geoLocationManager.off('locationEnabledChange')

### off(type:'locationEnabledChange',callback?:Callback<boolean>):void;

取消订阅位置服务状态变化。

系统能力：SystemCapability.Location.Location.Core

#### 参数：

| 参数名   | 类型              | 必填 | 说明                                                         |
| -------- | ----------------- | ---- | ------------------------------------------------------------ |
| type     | string            | 是   | 设置事件类型。type为”locationEnabledChange“，表示位置服务状态。 |
| callback | Callback<boolean> | 否   | 需要取消订阅的回调函数。若无此参数，则取消当前类型的所有订阅。 |

#### 错误码：

| 错误码ID | 错误信息                         |
| -------- | -------------------------------- |
| 3301000  | Location service is unavailable. |

#### 示例：

```typescript
import geoLocationManager from '@ohos.geoLocationManager';
let locationEnableChange = (state) =>{
    console.log('locationEnabledChange: state: '+JSON.stringify(state));
}
try{
    geoLocationManager.on('locationEnabledChange',locationEnabledChange);
    geoLocationManager.off('locationEnabledChange',locationEnabledChange);
}catch(err){
    console.error("errCode:"+err.code+",errMessage:"+err.message);
}
```



## 5.geoLocationManager.on('cachedGnssLocationsChange')

on(type:'cachedGnssLocationsChange',request:CachedGnssLocationsRequest,callback: Callback<Array<Location>>):void;

#### 订阅缓存GNSS定位结果上报事件。

#### 需要权限:ohos.permission.APPROXIMATELY_LOCATION

#### 系统能力：SystemCapability.Location.Gnss



## 6.geoLocationManager.off('cachedGnssLocationsChange')

#### 取消订阅缓存GNSS定位结果上报事件。



## 7.geoLocationManager.on('satelliteStatusChange')

on(type:'satelliteStatusChange',callback: Callback<SatelliteStatusInfo>):void;

#### 订阅GNSS卫星状态信息上报事件



## 8.geoLocationManager.off('satelliteStatusChange')

#### 取消订阅GNSS卫星状态信息上报事件。



## 9.geoLocationManager.on('nmeaMessage')

#### 订阅GNSS NMEA信息上报事件。



## 10.geoLocationManager.off('nmeaMessage')

#### 取消订阅GNSS NMEA信息上报事件。



## 11.geoLocationManager.on('gnssFenceStatusChange')

#### on(type:'gnssFenceStatusChange',request:GeofenceRequest,want:WantAgent):void;

#### 添加一个围栏，并订阅地理围栏事件。

#### 需要权限：ohos.permission.APPROXIMATELY_LOCATION

#### 系统能力：SystemCapability.Location.Location.Geofence

#### 参数：

| 参数名  | 类型            | 必填 | 说明                                                         |
| ------- | --------------- | ---- | ------------------------------------------------------------ |
| type    | string          | 是   | 设置事件类型。type为”gnssFenceStatusChange“，表示订阅围栏事件上报。 |
| request | GeofenceRequest | 是   | 围栏的配置参数。                                             |
| want    | wangAgent       | 是   | 用于接受地理围栏事件上报（进出围栏）。                       |

#### 错误码：

| 错误码ID | 错误信息                         |
| -------- | -------------------------------- |
| 3301000  | Location service is unavailable. |
| 3301100  | The location switch is off.      |
| 3301600  | Failed to operate the geofence.  |

#### 示例：

```typescript
import geoLocationManager from '@ohos.geoLocationManager';
import wantAgent from '@ohos.app.ability.wantAgent';

let wantAgentInfo = {
    wants:[
        {
            bundleName: "com.example.myapplication",
            abilityName: "EntryAbility",
            action: "action1"
        }
    ],
    operationType: wantAgent.OperationType.START_ABILITY,
    requestCode: 0,
    wantAgentFlags: [wantAgent.WantAgentFlags.UPDATE_PRESENT_FLAG]
};

wantAgent.getWantAgent(wantAgentInfo).then((wantAgentObj) =>{
    let requestInfo = {'priority': 0x201,'scenario':0x301,"geofence":{"latitude":26,"radius":100,"expiration":10000}};
    try{
        geoLocationManager.on('gnssFenceStatusChange',requestInfo,wantAgentObj);
    }catch(err){
        console.error("errCode:"+err.code+",errMessage:"+err.message);
    }
});
```



## 12.geoLocationManager.off('gnssFenceStatusChange')

#### 删除一个围栏，并取消订阅该围栏事件。



## 13.geoLocationManager.on('countryCodeChange')

#### 订阅国家码信息变化事件。



## 14.geoLocationManager.off('countryCodeChange')

#### 取消订阅国家码变化事件。



## 15.geoLocationManager.getCurrentLocation

#### 获取当前位置，使用callback回调异步返回结果。



## 16.geoLocationManager.getCurrentLocation

#### 获取当前位置，通过callback方式异步返回结果。

#### 需要权限：ohos.permission.APPROXIMATELY_LOCATION

#### 系统能力：SystemCapability.Location.Core

#### 参数：

| 参数名   | 类型                    | 必填 | 说明                   |
| -------- | ----------------------- | ---- | ---------------------- |
| callback | AsyncCallback<location> | 是   | 用来接受位置信息的回调 |

#### 错误码：

| 错误码ID | 错误信息                                    |
| -------- | ------------------------------------------- |
| 3301000  | Location Service is unavailable.            |
| 3301100  | The location switch is off.                 |
| 3301200  | Failed to obtain the geographical location. |

#### 示例：

```typescript
import geoLocationManager from '@ohos.geoLocationManager';
let locationChange  = (err,location) => {
    if(err){
        console.log('locationChanger: err=' + JSON.stringify(err));
    }
    if(location){
        console.log('locationChanger: location=' + SON.stringify(location));
    }
};

try{
    geoLocationManager.getCurrentLocation(locationChange);
}catch(err){
    console.error("errCode:"+err.code+ ",errMessage:"+err.message);
}
```



## 17.geoLocationManager.getCurrentLocation

#### getCurrentLocation(request?:CurrentLocationRequest): Promise<Location>

#### 获取当前位置，使用Promise方式异步返回结果。

#### 需要权限： ohos.permission.APPROXIMATELY_LOCATION

#### 系统能力：SystemCapability.Location.Location.Core

#### 参数：

| 参数名  | 类型                   | 必填 | 说明               |
| ------- | ---------------------- | ---- | ------------------ |
| request | CurrentLocationRequest | 否   | 设置位置请求参数。 |

#### 返回值：

| 参数名            | 类型     | 必填 | 说明           |
| ----------------- | -------- | ---- | -------------- |
| Promise<Location> | Location | NA   | 返回位置信息。 |

#### 错误码：

| 错误码ID | 错误信息                                    |
| -------- | ------------------------------------------- |
| 3301000  | Location service is unavailable.            |
| 3301100  | The locaiton switch is off.                 |
| 3301200  | Failed to obtain the geographical location. |

#### 示例

```typescript
import geoLocationManager from '@ohos.geoLocationManager';
let requestInfo = {'priority':0x203,'scenario': 0x300,'maxAccuracy':0};
try{
    geoLocationManager.getCurrentLocation(requestInfo).then((result) =>{
        console.log('current location: '+JSON.stringify(result));
    })
    .catch((error) =>{
        console.log('promise,getCurrentLocation: error='+JSON.stringify(error));
    });
}catch(err){
    console.error("errCode:"+err.code + ",errMessage:"+err.message);
}
```



## 18.geoLocationManager.getLastLocation

#### 获取上一次位置



## 19.geoLocationManager.isLocationEnabled

#### 判断位置服务是否已经使能。



## 20.geoLocationManager.getAddressesFromLocation

#### 调用逆地理编码服务，将坐标转换为地理描述，使用Promise方式异步返回结果。



## 21.geoLocationManager.getAddressesFromLocationName

#### 调用地理编码服务，将地理描述转换为具体坐标，使用callback回调异步返回结果。



## 22.geoLocationManager.getAddressesFromLocationName

#### 调用地理编码服务，将地理描述转换为具体坐标，使用Promise方式异步返回结果。



## 23.geoLocationManager.isGeocoderAvaiable

#### 判断（逆）地理编码服务状态。



## 24.geoLocationManager.getCachedGnssLocationsSize

#### 获取GNSS芯片缓存位置的个数



## 25.geoLocationManager.flushCachedGnssLocations

#### 读取并清空GNSS芯片所有缓存位置。



## 26.geoLocationManager.sendCommand

#### 给位置服务子系统的各个部件发送扩展命令。

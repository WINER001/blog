---
title: HarmonyOS访问控制授权申请
date: 2023/11/20 10:12
tags: 
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/41.jpg

---



# HarmonyOS访问控制授权申请



### 场景介绍

应用的APL（Ability Privilege Level）等级分为normal，system_basic和system_core三个等级，默认情况下，应用的APL等级都为normal等级。权限类型分为system_grant和user_grant两种类型。应用可申请的权限项参见应用权限列表。



### 配置文件权限声明

应用需要再工程配置文件中，对需要的权限逐个证明，未在配置文件中声明的权限，应用将无法获得授权。HarmonyOS提供了两种应用模型，分别为FA模型和Stage模型，更多信息可以参考应用模型解读。不同的应用迷行的应用包结构不同，所使用的配置文件不同。



| 标签      | 是否必填 | 说明                                                         |
| --------- | -------- | ------------------------------------------------------------ |
| name      | 是       | 权限名称。                                                   |
| reason    | 否       | 描述申请权限的原因。<br />》说明：当申请的权限为user_grant权限时，此字段必填。 |
| usedScene | 否       | 描述权限使用的场景和时机。<br />》说明：当申请的权限为user_grant权限时，此字段必填。 |
| abilities | 否       | 标识需要使用到该权限的Ability，标签为数组形式。<br />适用模型：Stage模型 |
| ability   | 否       | 标识需要会使用到该权限的Ability，标签为数组形式。<br />适用模型：FA模型 |
| when      | 否       | 标识权限使用的时机，值为inuse/always。<br />- inuse：标识为仅允许前台使用。<br />- always：表示前后台都可使用。 |



### Stage模型

使用Stage模型的应用，需要再module.json5配置文件中声明权限。

```js
{
    "module" : {
        "requestPermissions" :[
            {
                "name" : "ohos.permission.READ_CALENDAR",
                "reason" : "$string:reason",
                "usedScene" : {
                    "abilities" : [
                        "FormAbility"
                    ],
                    "when":"inuse"
                }
            },
            {
                "name" : "ohos.permission.PERMISSION2",
                "reason": "$string:reason",
                "usedScene": {
                    "abilities":[
                        "FormAbility"
                    ],
                    "when":"always"
                }
            }
        ]
    }
}
```



### FA模型

使用FA模型的应用，需要在config.json配置文件中声明权限。

```js
{
    "module" : {
        "rePermissions":[
            {
                "name" : "ohos.permission.PERMISSION1",
                "reason": "$string:reason",
                "usedScene":{
                    "ability":[
                        "FormAbility"
                    ],
                    "when":"inuse"
                }
            },
            {
                "name":"ohos.permission.PERMISSION2",
                "reason":"$string:reason",
                "usedScene":{
                    "ability":[
                        "FormAbility"
                    ],
                    "when":"always"
                }
            }
        ]
    }
}
```



### 向用户申请授权

当应用需要访问用户的隐私信息或使用系统能力时，例如获取位置信息，访问日历，使用相机拍摄照片或录制视频等，应该向用户请求授权。这需要使用user_grant类型权限。在此之前，应用需要进行权限校验，以判断当前调用者是否具备所需的权限。如果权限校验结果表明当前应用尚未被授权该权限，则应使用动态弹窗授权方式，为用户提供手动授权的入口。

```
说明：
每次访问受目标权限保护的接口之前，都需要使用requestPermissionsFromUser（）接口请求相应的权限。用户可能在动态授予权限后通过系统设置来取消应用的权限，因此不能将之前授予的授权状态持久化。
```



### Stage模型

以允许应用读取日历信息为例进行说明。

1.申请ohos.permission.READ_CALENDAR权限

2.校验当前是是否已经授权。

在进行权限申请之前，需要先检查当前应用程序是否已经被授予了权限。可以通过调用checkAccessToken（）方法来校验当前是否已经授权。颗已经授权，则可以直接访问目标操作，否则需要进行下一步操作，即向用户申请授权。

```typescript
import bundleManager from '@ohos.bundle.bundleManager';
import abilityAccessCtrl,{ Permissions } from '@ohos.abilityAccessCtrl';

async function checkAccessToken(permission: Permissions): Promise<abilityAccessCtrl.GrantStatus>{
    let atManager = abilityAccessCtrl.createAtManager();
    let grantStatus: abilityAccessCtrl.GrantStatus;
    
    //获取应用程序的accessTokenID
    let tokenId: number;
    try{
        let bundleInfo: bundleManager.BundleInfo = await bundleManager.getBundleInfoForSelf(bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION);
        let appInfo: bundleManager.ApplicationInfo = bundleInfo.appInfo;
        tokenId  = appInfo.accessTokenId;
    }catch(err){
        console.error(`getBundleInfoForSelf failed, code is ${err.code},message is ${err.message}`);
    }
    
    //校验应用是否被赋予权限
    try{
        grantStatus = await atManager.checkAccessToken(tokenId,permission);
    }catch(err){
        console.error(`checkAccessToken failed,code is ${err.code},message is ${err.message}`);
    }
    
    return grantStatus;
}

async function checkPermissions(): Promise<void>{
    const permissions: Array<Permissions> = ['ohos.permission.READ_CALENDAR'];
    let grantStatus: abilityAccessCtrl.GrantStatus = await checkAccessToken(permission[0]);
    
    if(grantStatus === abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED){
        //已经授权，可以继续访问目标操作
    }else{
        //申请日历权限
    }
}
```

3.动态向用户申请权限。

动态向用户申请权限是指在应用程序运行时向用户请求授权的过程。可以通过调用requestPermissionsFromUser（）方法实现。该方法接受一个权限列表参数，例如位置，日历，相机，麦克风等。用户可以选择授予权限或者拒绝授权。

在UIAbility中向用户申请授权。

```typescript
import UIAbility from '@ohos.app.ability.UIAbility';
import window from '@ohos.window';
import abilityAccessCtrl,{Permissions} from '@ohos.abilityAccessCtrl';

const permissions: Array<Permissions> = ['ohos.permission.READ_CALENDAR'];

export default class EntryAbility extends UIAbility{
    onWindowStageCreate(windowStage: window.WindowStage){
        //Main window is created,set main page for this ability
        let context = this.context;
        let atManager = abilityAccessCtrl.createAtManager();
        //requestPermissionsFromUser会判断权限的授权状态来决定是否唤起弹窗
        
        atManager.requestPermissionsFromUser(context,permissions).then((date) =>{
            let grantStatus: Array<number> = data.authResults;
            let length: number = grantStatus.length;
            for(let i = 0;i<length;i++){
                if(grantStatus[i] === 0){
                    //用户授权，可以继续访问目标操作
                }else{
                    //用户拒绝授权，提示用户必须授权才能访问当前页面的功能，并引导用户到系统设置中打开相应的权限
                    return;
                }
            }
            //授权成功
        }).catch((err)=>{
            console.error(`requestPermissionsFromUser failed,code is ${err.code},message is ${err.message}`);
        })
    }
}
```



在UI中向用户申请授权。

```typescript
import abilityAccessCtrl,{Permissions} from '@ohos.abilityAccessCtrl';
import common from '@ohos.app.ability.common';

const permissions: Array<Permissions> = ['ohos.permission.READ_CALENDAR'];

@Entry
@Component
struct Index{
    reqPermissionsFromUser(permissions: Array<Permissions>):void{
        let context = getContext(this) as common.UIAbilityContext;
    	let atManager = abilityAccessCtrl.createAtManager();
    	// requestPermissionsFromUser会判断权限的授权状态来决定是否唤起弹窗
    	atManager.requestPermissionsFromUser(context,permissions).then((date) =>{
            let grantStatus: Array<number> = data.authResults;
            let length: number = grantStatus.length;
            for(let i = 0;i<length;i++){
                if(grantStatus[i] === 0){
                    //用户授权，可以继续访问目标操作
                }else{
                    //用户拒绝授权，提示用户必须授权才能访问当前页面的功能，并引导用户到系统设置中打开相应的权限
                    return;
                }               
            }
            //授权成功
        }).catch((err) =>{
            console.error(`requestPermissionsFromUser failed,code is ${err.code},message is ${err.message}`);
        })
    }

	//页面展示
	build(){
        //...
    }
}
```



4.处理授权结果

调用requestPermissionsFromUser()方法后，应用程序将等待用户授权的结果。如果用户授权，则可以继续访问目标操作。如果用户拒绝授权，则需要提示用户必须授权才能访问当前页面的功能，并引导用户到系统设置中打开相应的权限。

```typescript
function openPermissionsInSystemSettings(): void{
    let context = getContext(this) as common.UIAbilityContext;
    let wantInfo = {
        action: 'action.settings.app.info',
        parameters:{
            settingsParamBundleName: 'com.example.myapplication' //打开指定应用的详情页面
        }
    }
    context.startAbility(wantInfo).then(() =>{
        //...
    }).catch((err) =>{
        //...
    })
}
```

FA模型

通过调用requestPermissionsFromUser()接口向用户动态申请授权。

```typescript
import featureAbility from '@ohos.ability.featureAbility';

reqPermissions(){
    let context = featureAbility.getContext();
    let array:Array<string> = ["ohos.permission.PERMISSION2"];
    //requestPermissionsFromUser会判断权限的授权状态类决定是否唤起弹窗
    context.requestPermissionsFromUser(array,1).then(function(date){
        console.log("date:"+JSON.stringify(data));
        console.log("date permissions:"+JSON.stringify(date.permissions));
        console.log("date result:"+JSON.stringify(date.authResults));
    },(err)=>{
        console.error('Failed to start ability',err.code)
    });
}
```


---
title: HarmonyOS_Tab组件
date: 2023/12/11 10:12
tags: 
  - Harmony
categories: Harmony
top_img: ./img/41.jpg
cover: ./img/22.jpg

---



# HarmonyOS_Tab组件

## 1.MainPage主页面

```typescript
import CommonConstants from '../common/constants/CommonConstants';
import Home from "../view/Home"
import Setting from "../view/Setting"

@Entry
@Component
struct MainPage{
    @State currentIndex: number = CommonConstants.HOME_TAB_INDEX;
    //1.一个TabsController
    private tabsController: TabsController = new TabsController();
    
    //2.一个TabBuilder（四个参数）
    @Builder TabBuilder(title: string,index: number, selectedImg: Resource,normalImg: Resource){
        //主页内容
        Column(){
            Image(this.currentIndex === index ? selectedImg : normalImg)
            	.width($r('app.float.mainPage_baseTab_size'))
            	.height($r('app.float.mainPage_baseTab_size'))
            Text(title)
            	.margin({ top: $r('app.float.mainPage_baseTab_top')})
            	.fontSize($r('app.float.main_tab_fontSize'))
            	.fontColor(this.currentIndex === index ? $r('app.color.mainPage_selected') : $r('app.color.mainPage_normal'))
        }
        .justifyContent(FlexAlign.Center)
        .height($r('app.float.mainPage_barHeight'))
        .width(CommonConstants.FULL_PARENT)
        .onClick(()=>{
            this.currentIndex = index;
            this.tabsController.changeIndex(this.currentIndex);
        })
    }
    
    //3.build部分
    build(){
        //直接一个Tabs（底部，controller）
        Tabs({
            barPosition: BarPosition.End,
            controller: this.tabsController
        }){
            //第一个TabContent
            TabContent(){
                Home()
            }
            .padding({left: $r('app.float.mainPage_padding'),right:$r('app.float.mainPage_padding')})
            .backgroundColor($r('app.color.mainPage.backgroundColor'))
            //这里用上面声明的builder，四个参数为，TITLE，INDEX，选中图，未选中图
            .tabBar(this.TabBuilder(CommonConstants.HOME_TITLE,CommonConstants.HOME_TAB_INDEX,
                                   $r('app.media.home_selected'),$r('app.media.home.normal')))
            
            //第二个TabContent
            TabContent(){
                Setting()
            }
            .padding({left: $r('app.float.mainPage_padding'),right:$r('app.float.mainPage_padding')})
            .backgroundColor($r('app.color.mainPage_backgroundColor'))
            .tabBar(this.TabBuilder(CommonConstants.MINE_TITLE,CommonConstants.MINE_TAB_INDEX,
                                   $r('app.media.mine_selected'),$r('app.media.mine_normal')))
        }
        .width(CommonConstants.FULL_PARENT)
        .backgroundColor(Color.White)
        .barHeight($r('app.float.mainPage_barHeight'))
        .barMode(BarMode.Fixed)
        //点击触发
        .onChange((index: number) =>{
            this.currentIndex = index;
        })
    }
}
```



## 2.Home子页面

![](./img/90.png)

```typescript
@Component
export default struct Home{
    //Swipper轮转
    private swiperController: SwiperController = new SwiperController();
    
    build({
        Scroll(){
            Column({ space: CommonConstants.COMMONSPACE }){
                Column(){
                    //第一行标题
                    Text($r('app.string.mainPage_tabTitles_home'))
                    	.fontWeight(FontWeight.Medium)
                    	.fontSize($r('app.float.page_title_text_size'))
                    	.margin({ top: $r{'app.float.mainPage_tabTitles_margin'}})
                    	.padding({left: $r('app.float.mainPage_tabTitles_padding')})
                }
                .width(CommonConstants.FULL_PARENT)
                .alighItems(HorizontalAlign.Start)
                
                //轮转图片
                Swiper(this.swiperController){
                    ForEach(mainViewModel.getSwiperImages(),(img: Resource) =>{
                        Image(img).borderRadius($r('app.float.home_swiper_borderRadius'))
                    },(img:Resource)=> JSON.stringify(img.id))
                }
                .margin({top: $r('app.float.home_swiper_margin')})
                .autoPlay(true)
                
                //网格布局
                Grid(){
                    ForEach(mainViewModel.getFirstGridData(),(item: ItemData) =>{
                        GridItem(){
                            Column(){
                                Image(item.img)
                                	.width($r('app.float.home_homeCell_size'))
                                	.height($r('app.float.home_homeCell_size'))
                                Text(item.title)
                                	.fontSize($r('app.float.little_text_size'))
                                	.margin({top: $r('app.float.home_homeCell_margin')})
                            }
                        }
                    },(item: ItemData) => JSON.stringify(item))
                }
                .columnsTemplate('1fr 1fr 1fr 1fr')
                .rowsTemplage('1fr 1fr')
                .columnsGap($r('app.float.home_grid_columnsGap'))
                .rowsGap($r('app.float.home_grid_rowGap'))
                .padding({ top: $r('app.float.home_grid_padding'),bottom: $r('app.float.home_grid_padding')})
                .height($r('app.float.home_grid_height'))
                .backgroundColor(Color.White)
                .borderRadius($r('app.float.home_grid_borderRadius'))
                
                Text($r('app.string.home_list'))
                	.fontSize($r('app.float.normal_text_size'))
                	.fontWeight(FontWeight.Medium)
                	.width(CommonConstants.FULL_PARENT)
                	.margin({top: $r('app.float.home_text_margin')})
                
                Grid(){
                    ForEach(mainViewModel.getSecondGridData(),(secondItem:ItemData) =>{
                        GridItem(){
                            Column(){
                                Text(secondItem.title)
                                	.fontSize($r('app.float.normal_text_size'))
                                	.fontWeight(FontWeight.Medium)
                                Text(secondItem.others)
                                	.margin({top: $r('app.float.home_list_margin')})
                                	.fontSize($r('app.float.little_text_size'))
                                	.fontColor($r('app.color.home_grid_fontColor'))
                            }
                            .alignItems(HorizontalAlign.Start)
                        }
                        .padding({ top: $r('app.float.home_list_padding'),left: $r('app.float.home_list_padding')})
                        .borderRadius($r('app.float.home_backgroundImage_borderRadius'))
                        .align(Alignment.TopStart)
                        .backgroundImage(secondItem.img)
                        .backgroundImageSize(ImageSize.Cover)
                        .width(CommonConwstants.FULL_PARENT)
                        .height(CommonConstants.FULL_PARENT)
                    },(secondItem: ItemData) => JSON.stringify(secondItem))
                }
                .width(CommonConstants.FULL_PARENT)
                .height($r('app.float.home_secondGrid_height'))
                .columnsTemplate('1fr 1fr')
                .rowsTemplate('1fr 1fr')
                .columnsGap($r('app.float.home_grid_columnsGap'))
                .rowsGap($r('app.float.home_frid_rowGap'))
                .margin({bottom: $r('app.float.setting_button_bottom')})
            }
            .height(CommonConstants.FULL_PARENT)
        }
    })
}
```



## 3.Setting子页面

![](./img/91.png)

```typescript
import CommonConstants from '../common/constants/commonConstants;
import ItemData from '../viewmodel/ItemData';
import mainViewModel from '../viewmodel/MainViewModel';

@Component
export default struct Setting{
    
    //设置页面的每一单元
    @Builder settingCell(item: ItemData){
        Row(){
            Row({ space: CommonConstants.COMMON_SPACE}){
                Image(item.img)
                	.width($r('app.float.setting_size'))
                	.height($r('app.float.setting_size'))
                Text(item.title)
                	.fontSize($r('app.float.normal_text_size'))
            }
            
            if(item.others === null){
                Image($r('app.media.right_grey'))
                	.width($r('app.float.setting_jump_width'))
                	.height($r('app.float.setting_jump_height'))
            }else{
                Toggle({ type: ToggleType.Switch, isOn: false})
            }
        }
        .justifyContent(FlexAlign.SpaceBetween)
        .width(CommonConstants.FULL_PARENT)
        .padding({
            left: $r('app.float.setting_settingCell_left'),
            right: $r('app.float.setting_settingCell_right')
        })
    }
    
    build(){
        //首先是个滚轮页面
        Scroll(){
            Column({ space: CommonConstatns.COMMON_SPACE}){
                Column(){
                    Text($r('app.string.mainPage_tabTitles_mine'))
                    	.fontWeight(FontWeight.Medium)
                    	.fontSize($r('app.float.page_title_text_size'))
                    	.margin({top: $r('app.float.mainPage_tabTitles_margin')})
                    	.padding({left: $r('app.float.mainPage_tabTitles_padding')})
                }
                .width(CommonConstants.FULL_PARENT)
                .alignItems(HorizontalAlign.Start)
                
                //用户信息
                Row(){
                    Image($r('app.media.account'))
                    	.width($r('app.float.setting_account_size'))
                    	.height($r('app.float.setting_account_size'))
                    Column(){
                        Text($r('app.string.setting_account_name'))
                        	.fontSize($r('app.float.setting_account_fontSize'))
                        Text($r('app.string.setting_account_email'))
                        	.fontSize($r('app.float.little_text_size'))
                        	.margin({top: $r('app.float.setting_name_margin')})
                    }
                    .alignItems(HorizontalAlign.Start)
                    .margin({ left: $r('app.float.setting_account_margin')})
                }
                .margin({top: $r('app.float.setting_account_margin')})
                .alignItems(VerticalAlign.Center)
                .width(CommonConstants.FULL_PARENT)
                .height($r('app.float.setting_account_height'))
                .backgroundColor(Color.White)
               	.padding({ left: $r('app.float.setting_account_padding')})
                .borderRadius($r('app.float.setting_account_borderRadius'))
                
                List(){
                    ForEach(mainViewModel.getSettingListData(),(item: ItemData) =>{
                        ListItem(){
                            this.settingCell(item)
                        }
                        .height($r('app.float.setting_list_height'))
                    },(item: ItemData) => JSON.stringify(item))
                }
                .backgroundColor(Color.White)
                .width(CommonConstants.FULL_PARENT)
                .height(CommonConstants.SET_LIST_WIDTH)
                .divider({
                    strokeWidth: $r('app.float.setting_list_strokeWidth'),
                    color: Color.Grey,
                    startMargin: $r('app.float.setting_list_startMargin'),
                    endMargin: $r('app.float.setting_list_endMargin')
                })
                .borderRadius($r('app.float.setting_list_borderRadius'))
                .padding({ top: $r('app.float.setting_list_padding'), bottom: $r('app.float.setting_list_padding')})
                Blank()
                
                //退出登录按钮
                Button($r('app.string.setting_button'),{type:ButtonType.Capsule})
                	.width(CommonConstants.BUTTON_WIDTH)
                	.height($r('app.float.login_button_height'))
                	.fontSize($r('app.float.normal_text_size'))
                	.fontColor($r('app.color.setting_button_fontColor'))
                	.fontWeight(FontWeight.Medium)
                	.backgroundColor($r('app.color.setting_button_backgroundColor'))
                	.margin({ bottom: $r('app.float.setting_button_bottom')})
            }
            .height(CommonConstants.FULL_PARENT)
        }
    }
}
```


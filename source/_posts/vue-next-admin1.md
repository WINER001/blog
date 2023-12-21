---
title: vue-next-admin后台管理系统
date: 2023/6/8
tags: vue-next-admin
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/42.jpg
---





## 参考https://lyt-top.gitee.io/vue-next-admin-doc-preview/config/



## 1.安装vue-next-admin

```
# 克隆项目
git clone https://gitee.com/lyt-top/vue-next-admin.git

# 进入项目
cd vue-next-admin

# 安装依赖
cnpm install

# 运行项目
cnpm run dev

# 打包发布
cnpm run build
```



## 2.使用vue-next-admin



### 1.菜单配置router.ts

```js
{
  // 菜单路径，用于跳转
  path: '/home',
  // 菜单 name，用于界面 keep-alive 路由缓存。
  // 此 name 需要与 component 组件中的 name 值相同（唯一）
  name: 'home',
  // 组件路径
  component: () => import('/@/views/home/index.vue'),
  // 附加自定义数据
  meta: {
    // 菜单标题（国际化写法）
    title: 'message.router.home',
    // 菜单外链链接
    // 开启外链条件，`1、isLink: true 2、链接地址不为空（meta.isLink） 3、isIframe: false`
    isLink: '',
    // 菜单是否隐藏（菜单不显示在界面，但可以进行跳转）
    isHide: false,
    // 菜单是否缓存
    isKeepAlive: true,
    // 菜单是否固定（固定在 tagsView 中，不可进行关闭），右键菜单无 `关闭` 项
    isAffix: true,
    // 是否内嵌
    // 开启条件，`1、isIframe: true 2、链接地址不为空（meta.isLink）`
    isIframe: false,
    // 当前路由权限标识，取角色管理。控制路由显示、隐藏。超级管理员：admin 普通角色：common
    // 之前 auth 取用户（角色下有多个用户）
    roles: ['admin', 'common'],
    // 菜单图标
    icon: 'iconfont icon-shouye',
    // 自行再添加
    ...
  },
}

```



### 2.布局配置 /src/stores/themeConfig.ts

包含：菜单栏，顶栏，tagsView标签页，主内容区

#### 1.全局主题 /src/theme

```
├── theme (页面样式)
	├── common (基础样式)
	│	├── transition.scss (页面过渡动画)
	│	└── var.scss (全局主题样式，用于全局改变样式)
	│
	├── media (媒体查询)
	│	├── chart.scss (大数据图表)
	│	├── cityLinkage.scss (Cascader 级联选择器城市选择)
	│	├── dialog.scss (弹窗)
	│	├── error.scss (404、401界面)
	│	├── form.scss (表单)
	│	├── home.scss (首页)
	│	├── index.scss (媒体查询定义主样式)
	│	├── layout.scss (框架布局)
	│	├── login.scss (登录界面)
	│	├── media.scss (媒体查询主出口)
	│	├── pagination.scss (分页)
	│	├── personal.scss (个人中心)
	│	├── scrollbar.scss (页面滚动条)
	│	└── tagsView.scss (tagsView 标签页)
	│
	├── mixins (scss混入)
	│	├── element-mixins.scss (定义重置的element plus混入复用样式)
	│	├── function.scs (全局主题颜色调用混入函数)
	│	└── mixins.scss (定义一些常用的全局混入样式)
	│
	├── app.scss (页面主样式，用于重置浏览器默认样式)
	├── base.scss (基础样式、过渡动画引入等)
	├── dark.scss (深色主题)
	├── element.scss (重置的element plus样式，用于改变主题)
	├── iconSelector.scss (图标选择器)
	├── index.scss (页面样式主出口)
	├── loading.scss (loading样式)
	├── other.scss (其它样式)
	└── waves.scss (按钮波浪样式)

```

#### 2.顶栏 /@/layout/navBars/breadcrumb

#### 3.菜单 /@/layout/navMenu

#### 4.分栏 /@/layout/component/columnsAside.vue

#### 5.固定header /@/layout/main

#### 6.锁屏 /@/layout/lockScreen

#### 7.侧边栏logo  /@/layout/logo

#### 8.面包屑 Breadcrumb   /@/layout/navBars/Breadcrumb/breadcrumb.vue

#### 9.Tagsview标签页   /@/layout/navBars/tagsView

#### 10.Footer版权	/@/layout/footer

#### 11.水印		/@/utils/watermark.ts



### 3.字体图标



#### 1.框架中全局注册svg

/@/utils/other.ts,  main.ts中引入import other from '/@/utils/other'; 添加了ele前缀，防止图标冲突，el前缀已经被使用，可以使用el-xxx。但是不建议以el svg前缀,因为会与element plus内置组件冲突

#### 2.页面中使用svg

使用element plus的图标，可去 https://lyt-top.gitee.io/vue-next-admin-preview/#/pages/element 复制粘贴

svg图标说明

如：ele-User，由两部分组成

1.ele：框架中全局注册svg中添加的svg图标前缀

2.User：为svg图标，请注意它的开头都是大写的，element plus官网Icon图标

```html
<el-input
  type="text"
  :placeholder="$t('message.account.accountPlaceholder1')"
  v-model="ruleForm.userName"
  clearable
  autocomplete="off"
>
  <template #prefix>
    <el-icon class="el-input__icon"><ele-User /></el-icon>
  </template>
</el-input>

```

element plus官网图标

```html
// 引入方法一
<script setup lang="ts">
  import { Calendar } from "@element-plus/icons";
</script>

// 引入方法二
<script lang="ts">
  import { Calendar } from "@element-plus/icons";
  return {
    Calendar
  }
</script>

```

```html
<el-input v-model="input1" placeholder="Pick a date" :suffix-icon="Calendar" />

```

#### 3.全局获取svg



### 4.服务端交互

#### 1.env文件

##### 	1.配置文件

```
.env              # 全局默认配置文件，不论什么环境都会加载合并
.env.development  # 开发环境下的配置文件
.env.production   # 生产环境下的配置文件
```

##### 	2.命名规则

为了防止意外地将一些环境变量泄漏到客户端，只有以 `VITE_` 为前缀的变量才会暴露给经过 vite 处理的代码。例如下面这个文件中：

```
DB_PASSWORD = foobar
VITE_SOME_KEY = 123
```

只有 `VITE_SOME_KEY` 会被暴露为 `import.meta.env.VITE_SOME_KEY` 提供给客户端，而 `DB_PASSWORD` 则不会。



#### 2.axios封装	/@/utils/request.ts

##### 	1.配置新建一个axios实例

```js
const service = axios.create({
  baseURL: import.meta.env.VITE_API_URL as any,
  timeout: 50000,
  headers: { "Content-Type": "application/json" },
});
```

##### 	2.添加请求拦截器

```js
service.interceptors.request.use(
  (config) => {
    // 在发送请求之前做些什么 token
    if (Session.get("token")) {
      // config.headers!['Authorization'] = `${Session.get('token')}`;
      config.headers.common["Authorization"] = `${Session.get("token")}`;
    }
    return config;
  },
  (error) => {
    // 对请求错误做些什么
    return Promise.reject(error);
  }
);
```

##### 	3.添加响应拦截器

```js
service.interceptors.response.use(
  (response) => {
    // 对响应数据做点什么
    const res = response.data;
    if (res.code && res.code !== 0) {
      // `token` 过期或者账号已在别处登录
      if (res.code === 401 || res.code === 4001) {
        Session.clear(); // 清除浏览器全部临时缓存
        window.location.href = "/"; // 去登录页
        ElMessageBox.alert("你已被登出，请重新登录", "提示", {})
          .then(() => {})
          .catch(() => {});
      }
      return Promise.reject(service.interceptors.response);
    } else {
      return res;
    }
  },
  (error) => {
    // 对响应错误做点什么
    if (error.message.indexOf("timeout") != -1) {
      ElMessage.error("网络超时");
    } else if (error.message == "Network Error") {
      ElMessage.error("网络连接错误");
    } else {
      if (error.response.data) ElMessage.error(error.response.statusText);
      else ElMessage.error("接口路径找不到");
    }
    return Promise.reject(error);
  }
);

```

#### 3.使用说明

##### 1.统一api文件夹

/src下新建/src/api文件夹。建议每一个模块，都新建一个文件夹（文件夹名称与模块名称相同，方便维护）

如：login模块，/@/api/login  文件夹下会有一个index.ts

```js
import request from '/@/utils/request';

/**
 * （不建议写成 request.post(xxx)，因为这样 post 时，无法 params 与 data 同时传参）
 *
 * 登录api接口集合
 * @method signIn 用户登录
 * @method signOut 用户退出登录
 */
export function all() {
  return request({
    responseType: "blob",
    url: "/fabric/gateway",
    method: "post",
    params :{
		cmd: {
		  channel: "winechannel",
		  chaincode: "mycontract",
		  action: "query"
		}
	  },
  });
}
```

### 5.打包

1. ####  .env文件

   ```bash
   # public path 配置线上环境路径（打包）、本地通过 http-server 访问时，请置空即可
   VITE_PUBLIC_PATH = /vue-next-admin-preview/
   ```

2. #### 打包命令

   ```bash
   # 项目根目录运行
   cnpm run build
   
   # 或者，package.json 中，鼠标放入 build 上点击 `运行脚本`
   "scripts": {
     "build": "vite build",
   },
   ```

   

## 3.高级用法

### 1.权限管理   /src/router

`后端控制` 与 `前端控制` 文件是相互独立的（互不影响），`后端控制` 不需要 `roles` 字段

```
├── router (存放路由信息)
	├── backEnd.ts (后端控制)
	├── frontEnd.ts (前端控制)
	├── index.ts (公用函数，路由拦截等)
	└── route.ts (路由菜单数据)
```

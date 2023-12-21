---
title: vue-next-admin中使用pinia来管理用户信息
date: 2023/6/18 14:25
tags: vue-next-admin
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/42.jpg
---



# vue-next-admin中使用pinia来管理用户信息



### 1.先介绍一下pinia

Pinia 是一个基于 Vue.js 的状态管理库，专注于提供简单、直观和可扩展的状态管理解决方案。它是为了替代 Vue 2.x 中的 Vuex 库而创建的，旨在提供更好的类型安全性和开发体验。

以下是 Pinia 库的一些主要特点和功能：

1. 基于 Vue 3：Pinia 是专门为 Vue 3 开发的状态管理库，与 Vue 2 不兼容。它利用了 Vue 3 的 Composition API 特性来提供更灵活、强大的状态管理功能。

2. 类型安全：Pinia 在设计之初就考虑到了类型安全性，使用 TypeScript 编写，并提供了强大的类型推断和类型检查。这样可以在开发过程中提供更好的代码补全、错误检测和调试体验。

3. 分模块状态：Pinia 支持将应用状态拆分为多个模块，每个模块都有自己的状态、操作和 getter。这种模块化的状态管理使得应用的状态更具可维护性和可扩展性。

4. Composition API 集成：Pinia 与 Vue 3 的 Composition API 紧密集成，允许在组件中使用 `useStore` 函数来获取状态存储库实例，并利用响应式 API、生命周期钩子等功能来管理状态。

5. 插件生态系统：Pinia 提供了丰富的插件生态系统，用于扩展和增强其功能。例如，它提供了 Devtools 插件用于调试和监控状态变化，以及 Persist 插件用于持久化存储状态。

6. 可测试性：Pinia 的设计使得在测试应用状态时更加简单。它提供了一套简洁的 API 和工具，以便编写单元测试、集成测试和端到端测试。

总的来说，Pinia 是一个现代化的 Vue 3 状态管理库，提供了类型安全、模块化、灵活和易于测试的状态管理解决方案。它适用于中小型到大型的 Vue.js 应用程序，并与 Vue 3 的 Composition API 紧密集成，使开发者能够更好地组织和管理应用的状态。



### 2.pinia在vue-next-admin里的应用

#### 1.入口useUserInfo(pinia).setUserInfos();

```typescript
import pinia from '/@/stores/index';
import { Session } from '/@/utils/storage';
import { useUserInfo } from '/@/stores/userInfo';
import { storeToRefs } from 'pinia';
/**
 * 前端控制路由：初始化方法，防止刷新时路由丢失
 * @method  NextLoading 界面 loading 动画开始执行
 * @method useUserInfo(pinia).setUserInfos() 触发初始化用户信息 pinia
 * @method setAddRoute 添加动态路由
 * @method setFilterMenuAndCacheTagsViewRoutes 设置递归过滤有权限的路由到 pinia routesList 中（已处理成多级嵌套路由）及缓存多级嵌套数组处理后的一维数组
 */
export async function initFrontEndControlRoutes() {
	// 界面 loading 动画开始执行
	if (window.nextLoading === undefined) NextLoading.start();
	// 无 token 停止执行下一步
	if (!Session.get('token')) return false;
	// 触发初始化用户信息 pinia
	// https://gitee.com/lyt-top/vue-next-admin/issues/I5F1HP
	await useUserInfo(pinia).setUserInfos();
	// 无登录权限时，添加判断
	// https://gitee.com/lyt-top/vue-next-admin/issues/I64HVO
	if (useUserInfo().userInfos.roles.length <= 0) return Promise.resolve(true);
	// 添加动态路由
	await setAddRoute();
	// 设置递归过滤有权限的路由到 pinia routesList 中（已处理成多级嵌套路由）及缓存多级嵌套数组处理后的一维数组
	setFilterMenuAndCacheTagsViewRoutes();
}
```

#### 2.userInfo.ts

```typescript
import { defineStore } from 'pinia';
import Cookies from 'js-cookie';
import { Session } from '/@/utils/storage';

/**
 * 用户信息
 * @methods setUserInfos 设置用户信息
 */
export const useUserInfo = defineStore('userInfo', {
	state: (): UserInfosState => ({
		userInfos: {
			userName: '',
			photo: '',
			time: 0,
			roles: [],
			authBtnList: [],
		},
	}),
	actions: {
		async setUserInfos() {
			// 存储用户信息到浏览器缓存
			if (Session.get('userInfo')) {
				this.userInfos = Session.get('userInfo');
			} else {
				const userInfos = <UserInfos>await this.getApiUserInfo();
				this.userInfos = userInfos;
			}
		},
		// 模拟接口数据
		// https://gitee.com/lyt-top/vue-next-admin/issues/I5F1HP
		async getApiUserInfo() {
			return new Promise((resolve) => {
				setTimeout(() => {
					// 模拟数据，请求接口时，记得删除多余代码及对应依赖的引入
					const userName = Cookies.get('userName');
					// 模拟数据
					let defaultRoles: Array<string> = [];
					let defaultAuthBtnList: Array<string> = [];
					// admin 页面权限标识，对应路由 meta.roles，用于控制路由的显示/隐藏
					let adminRoles: Array<string> = ['admin'];
					// admin 按钮权限标识
					let adminAuthBtnList: Array<string> = ['btn.add', 'btn.del', 'btn.edit', 'btn.link'];
					// test 页面权限标识，对应路由 meta.roles，用于控制路由的显示/隐藏
					let testRoles: Array<string> = ['common'];
					// test 按钮权限标识
					let testAuthBtnList: Array<string> = ['btn.add', 'btn.link'];
					// 不同用户模拟不同的用户权限
					const role = Cookies.get('userrole')
					console.log(role);
					if (role === 'admin') {
						defaultRoles = adminRoles;
						defaultAuthBtnList = adminAuthBtnList;
					} else {
						defaultRoles = testRoles;
						defaultAuthBtnList = testAuthBtnList;
					}
					// 用户信息模拟数据
					const userInfos = {
						userName: userName,
						photo:
							userName === 'admin'
								? 'https://img2.baidu.com/it/u=1978192862,2048448374&fm=253&fmt=auto&app=138&f=JPEG?w=504&h=500'
								: 'https://img2.baidu.com/it/u=2370931438,70387529&fm=253&fmt=auto&app=138&f=JPEG?w=500&h=500',
						time: new Date().getTime(),
						roles: defaultRoles,
						authBtnList: defaultAuthBtnList,
					};
					Session.set('userInfo', userInfos);
					resolve(userInfos);
				}, 0);
			});
		},
	},
});

```


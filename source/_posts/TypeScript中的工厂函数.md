---
title: typescript中的工厂函数
date: 2023/10/8 13:17
tags: typescript
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/29.jpg

---



## TypeScript中的工厂函数（登录登出）

工厂函数是一种特殊的函数，用于创建和返回对象或其他数据结构。它通常用于封装和组织代码，允许动态地创建多个实例或对象，每个实例可能具有不同的属性或行为。



### 例子：

```typescript
import request from '/@/utils/request';

/**
 *
 * 登录api接口集合
 * @method login 用户登录
 * @method logout 用户退出登录
 */
export function useLoginApi() {
	return {
		signIn: (data: object) => {
			return request({
				url: '/login',
				method: 'post',
				data,
			});
		},
		signOut: (data: object) => {
			return request({
				url: '/logout',
				method: 'post',
				data,
			});
		},
	};
}

```

在提供的例子中，`useLoginApi` 就是一个工厂函数。详细解释它的特点和用法：



1. ### **目的：**
   
   - `useLoginApi` 的目的是创建一个包含两个方法的对象，用于处理登录和登出操作。这样可以将登录和登出的逻辑封装到一个单独的函数中，使代码更有组织性和可重用性。
   
   
   
2. ### **返回值：**
   
   - 该函数返回一个对象，该对象有两个属性 `signIn` 和 `signOut`，分别对应登录和登出操作的方法。
   
   
   
3. ### **参数：**
   
   - `useLoginApi` 函数本身没有接受任何参数。它只是一个工厂函数，用于创建对象。
   
   
   
4. ### **对象属性和方法：**
   
   - `signIn`: 一个函数，接受一个参数 `data`（一个对象），该函数用于发起登录请求。
   - `signOut`: 一个函数，接受一个参数 `data`（一个对象），该函数用于发起登出请求。
   
   
   
5. ### **使用方法：**

1. **导入函数：**
   首先，在你想要使用这个工厂函数的文件中，导入它：
   
   ```typescript
   import { useLoginApi } from './path/to/your/util/file';
   ```
   
2. **调用工厂函数：**
   使用 `useLoginApi` 函数来创建一个对象，该对象包含 `signIn` 和 `signOut` 方法：
   
   ```typescript
   const loginApi = useLoginApi();
   ```
   
3. **调用登录和登出方法：**
   现在，你可以调用 `signIn` 和 `signOut` 方法来执行登录和登出操作：
   ```typescript
   // 登录示例
   const loginData = { username: 'example', password: 'password' };
   loginApi.signIn(loginData)
     .then(response => {
       console.log('Login successful:', response);
     })
     .catch(error => {
       console.error('Login failed:', error);
     });
   
   // 登出示例
   const logoutData = { /* 数据对象，如果需要的话 */ };
   loginApi.signOut(logoutData)
     .then(response => {
       console.log('Logout successful:', response);
     })
     .catch(error => {
       console.error('Logout failed:', error);
     });
   ```

这样，你可以使用 `signIn` 和 `signOut` 方法来进行登录和登出操作，传递适当的数据对象给这些方法。
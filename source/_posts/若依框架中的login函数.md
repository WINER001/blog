---
title: 若依框架中的login函数
date: 2023/10/9 13:17
tags: Spring
categories: 若依框架
top_img: ./img/41.jpg
cover: ./img/28.jpg
---

# 若依框架中的login函数



## 1.先上代码

```js
	handleLogin() {
      this.$refs.loginForm.validate(valid => {
        if (valid) {
          this.loading = true;
          if (this.loginForm.rememberMe) {
            Cookies.set("username", this.loginForm.username, { expires: 30 });
            Cookies.set("password", encrypt(this.loginForm.password), { expires: 30 });
            Cookies.set('rememberMe', this.loginForm.rememberMe, { expires: 30 });
          } else {
            Cookies.remove("username");
            Cookies.remove("password");
            Cookies.remove('rememberMe');
          }
          this.$store
            .dispatch("Login", this.loginForm)
            .then(() => {
              this.$router.push({ path: this.redirect || "/" });
            })
            .catch(() => {
              this.loading = false;
              this.getCode();
            });
        }
      });
```



## 2.逐句分析

这段代码是一个处理登录的函数 `handleLogin()`：

1. `handleLogin() {`:
   定义了一个名为 `handleLogin` 的方法。

2. `this.$refs.loginForm.validate(valid => {`:
   使用 Vue.js 的 `this.$refs` 访问名为 "loginForm" 的组件引用，并调用其 `validate` 方法。传递了一个回调函数 `valid => { ... }` 给 `validate` 方法，用于处理验证的结果。

3. `if (valid) {`:
   在验证结果为真时执行以下代码块，即表单验证通过。

4. `this.loading = true;`:
   将 Vue 实例中的 `loading` 属性设置为 `true`，用于显示加载状态。

5. 条件判断 `if (this.loginForm.rememberMe) { ... } else { ... }`:
   检查是否需要记住用户登录信息，根据条件执行不同的代码块。

6. Cookies 操作：
   根据记住登录信息的需求，设置或删除相应的 Cookies。

7. `this.$store.dispatch("Login", this.loginForm)`:
   调用 Vuex store 中名为 "Login" 的 action，并传递 `this.loginForm` 对象作为参数。

8. `.then(() => { this.$router.push({ path: this.redirect || "/" }); })`:
   如果登录成功，执行 `.then` 回调函数，将用户重定向到指定路径。

9. `.catch(() => { this.loading = false; this.getCode(); });`:
   如果登录失败，执行 `.catch` 回调函数，将 `loading` 状态重置为 `false`，然后调用 `this.getCode()` 来获取验证码。

总体来说，这段代码处理了用户登录的逻辑，包括表单验证、加载状态显示、Cookies 操作和 Vuex action 的调用。在验证通过后，尝试登录，根据登录结果进行相应操作。如果登录失败，会调用 `this.getCode()` 来获取验证码。



## 3.$refs详细用法

`$refs` 是 Vue.js 中的一个特殊属性，用于访问组件上的引用（references）。它允许您直接访问组件实例，而无需通过事件或 props 传递数据。

在Vue.js组件中，可以给组件设置一个 `ref` 属性，然后可以通过 `this.$refs` 访问这个组件实例。这对于需要在父组件中操作或访问子组件的属性、方法或数据时非常有用。

以下是 `$refs` 的一些常见用法：

1. **访问子组件的属性和方法**:
   ```html
   <template>
     <child-component ref="myComponent"></child-component>
   </template>
   
   <script>
   import ChildComponent from './ChildComponent.vue';
   
   export default {
     components: {
       ChildComponent
     },
     mounted() {
       this.$refs.myComponent.someMethod();
     }
   }
   </script>
   ```

2. **访问DOM元素**:
   
   ```html
   <template>
     <button ref="myButton" @click="handleClick">Click me</button>
   </template>
   
   <script>
   export default {
     methods: {
       handleClick() {
         this.$refs.myButton.innerText = 'Button clicked';
       }
     }
   }
   </script>
   ```
   
   在这个例子中，`$refs` 可以用来访问DOM元素并修改其内容。

需要注意的是，使用 `$refs` 时应该确保组件已经被渲染，否则 `this.$refs.myComponent` 可能为 `undefined`。通常在 `mounted` 生命周期钩子或之后才能安全地使用 `$refs`

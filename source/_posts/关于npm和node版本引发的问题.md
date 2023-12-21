---
title: npm和node版本引发的一系列问题
date: 2023/8/21
tags: 疑难杂症
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/41.jpg

---

## npm和node版本引发的一系列问题

### 一，问题描述

vue-next-admin在cnpm run dev报错

之前还好用，所以排除代码的问题，只能是环境的问题。因为我之前因为跑别的项目改过npm和node的版本。

这个问题对我来说确实很棘手。

引发了一系列问题，所以简单记录下

### 二，问题相关



#### 1.npm和cnpm是什么关系

Npm（Node Package Manager）是 JavaScript 的包管理工具，用于安装、管理和分享代码包。而 cnpm（Chinese Npm）是一个在中国开发的 npm 镜像，旨在提供更快的安装速度和更稳定的访问，尤其是对于位于中国的开发者来说。cnpm 使用淘宝镜像来加速下载和安装过程。它们之间的关系是，cnpm 是 npm 的一个替代镜像，旨在改善中国地区用户的包管理体验。



#### 2.node清理缓存

清理npm缓存

```bash
npm cache clean --force
```

清理yarn缓存（如果使用yarn）

```bash
yarn cache clean
```

清理node.js模块缓存

```bash
rm -rf node_modules
rm package-lock.json
npm install
```



#### 3.执行 cnpm install 版本冲突

![](./img/89.png)

然后我全局搜索7.9.0发现有这样一行

```npm
npm_config_user_agent: 'npminstall/7.9.0 npm/? node/v16.13.0 win32 x64',
```

那好，我把版本换成node 16.13.0   npm 7.9.0

cnpm install还是不行

```
× Install fail! Error: run postinstall error, please remove node_modules before retry!
Command failed with exit code 1: node ./scripts/postinstall.js
Error: Command failed with exit code 1: node ./scripts/postinstall.js
    at makeError (C:\Users\yuchengji\AppData\Roaming\npm\node_modules\cnpm\node_modules\execa\lib\err
or.js:60:11)
    at handlePromise (C:\Users\yuchengji\AppData\Roaming\npm\node_modules\cnpm\node_modules\execa\ind
ex.js:118:26)
```

试了下npm install --force，公司网一直不行，等我回家试试。。。

### 四，后续

后来，我把node升级成18.17.1，这是官网最新的稳定版本， npm9.6.7是匹配的版本

然后清了一下缓存，删除了node_modules文件夹，再执行npm install。



我是在windows系统的git bash命令行中操作的，虽然是管理员权限，但是还是报了了权限问题。

最后用管理员权限打开cmd操作，操作成功了。
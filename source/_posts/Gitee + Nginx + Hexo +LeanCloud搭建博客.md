---
title: Gitee + Nginx + Hexo +LeanCloud搭建博客
date: 2023/5/7
tags: 网站搭建
categories: 建站
top_img: ./img/41.jpg
cover: ./img/41.jpg
---

# 个人网站搭建记录

## 1.确定需求：

​	需求一：首先呢，当然是在浏览器中输入ip（101.42.229.55），就可以访问页面~。
​					1.需要有自己的Linux云服务器（我用的腾讯云服务器，几十块）
​					2.在云服务器上部署nginx（部署个人博客，总不能一直session挂着进程吧，需要nginx来代理服务）

​	需求二：博客使用hexo框架和butterfly主题，萝卜青菜各有所爱，喜欢就行	(^_^)

​	需求三：代码要放在git里，这样的话发布就不用局限在本地一台电脑上，随便哪台电脑，git拉下来代码就能用
​					1.有自己的git库，并且本地的windows系统电脑和linux服务器都得有git

## 2.实现过程：

​	参考：https://zhuanlan.zhihu.com/p/120743882
​	主要的参考就是这个博客

### 	1.买个服务器。

### 	2.Linux云服务器上安装Git和Nginx

```shell
apt-get update
apt-get install git nginx -y
```

​	创建在home目录下新建git项目目录，并初始化

```sh
cd /home
git init --bare myblog.git 
```

​	配置Nginx托管文件目录并修改权限

```sh
mkdir -p /var/www/hexo
chown -R $USER:$USER /var/www/hexo
chmod -R 755 /var/www/hexo
```

​	修改Nginx的default文件使得root指向/var/www/hexo目录

```sh
vim /etc/nginx/sites-available/default
#进入编辑模式
```

![](./img/1.png)



重启nginx服务

```sh
service nginx restart
```

到这一步，在浏览器输入101.42.229.55就可以访问到nginx了，但是还没有指向我们的博客（刚才创建的git仓库）

然后要配置git钩子,vim编辑post-receive文件

```sh
vim /home/myblog.git/hooks/post-receive
```

打开文件后，加入下面代码

```shell
#!/bin/bash

git --work-tree=/var/www/hexo --git-dir=/var/repo/ganahBlog.git checkout -f
```

修改文件可执行权限

```sh
shmod +x /home/myblog.git/hooks/post-receive
```



$$$$$$$$$$$$$$$$$$$$$$$

到此为止在云服务（linux系统）上的操作就告一段落了，接下来是在本地的电脑（windows系统）操作

$$$$$$$$$$$$$$$$$$$$$$$



## 3.windows中配置环境

### 1.git官网搜索下载git

```sh
#cmd中输入下面命令看git是否安装成功
git --version
```

![](./img/2.png)	

### 2.nodejs官网下载node.js和npm



```sh
#cmd中输入下面命令看nodejs和npm是否安装成功
node -v
npm -v
```

![](/img/3.png)



### 3.打开gitbash，并安装hexo

创建一个文件夹xxxx/xx/myblog,在myblog文件夹中邮件打开gitbash

把云服务器上建好的git库拉下来

```sh
git clone root@{云服务器ip}:/home/myblog.git
```

然后初始化hexo博客

```sh
npm install -g hexo-cli
npm install hexo-deployer-git --save
hexo init blog
hexo clean && hexo g -d
```

到此为止本地hexo博客就有了，浏览器输入localhost:4000就能看到初始化博客了

此时，在myblog文件夹下就是这样的

![](./img/4.png)

### 4.安装并修改butterfly主题

在hexo根目录中输入

```sh
git clone -b master https://gitee.com/immyw/hexo-theme-butterfly.git themes/butterfly
npm install hexo-renderer-pug hexo-renderer-stylus --save
```



### 5.编辑站点配置文件_config.yml

```sh
vim _config.yml
```

将url改成https://{云服务器IP}/,以及设置网站标题，作者等信息

![](./img/5.png)

修改theme为butterfly，还有设置远程仓库

![](./img/6.png)

### 6.最后再部署

```sh
hexo clean && hexo g -d
```

使用IP访问

![](./img/7.png)







## 4.把本地创建的git仓库推送到远程gitee仓库中代码管理

### 	1.gitee官网申请注册并创建一个新仓库myblog

![](./img/8.png)

### 	2.在gitbash中进入/home/myblog目录，并将本地仓库与gitee上的远程仓库关联

```sh
git remote add origin https//gitee.com/yu-chengji/myblog.git
```

### 	3.把butterfly项目目录还原成普通目录

由于我们在本地创建的myblog项目中使用了butterfly主题，这也是一个git项目，这会导致butterfly项目会编程myblog项目的子项目，然后butterfly目录下的所有文件都传不上去，gitee仓库上的butterfly目录是个链接。所以我们要把butterfly项目目录还原成普通目录

```sh
git rm --cached themes/butterfly
rm -rf themes/butterfly/.git
rm themes/butterfly/.gitmodules
rm themes/butterfly/.gitignore
```

### 	4.提交代码到gitee线上仓库

```sh
git add .
git commit -m "first commit"
git push -u origin master
```

本人邮箱：1076901647@qq.com

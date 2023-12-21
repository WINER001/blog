---
title: 服务端配置mysql权限
date: 2023/7/10 14:18
tags: mysql
categories: mysql
top_img: ./img/41.jpg
cover: ./img/28.jpg
---



# 服务端配置mysql权限

### Mysql连接报错：1130-host ... is not allowed to connect to this MySql server

## 一，问题分析



### 1.首先，我的服务器已经开通了3306端口



### 2.我的Mysql配置已经允许所有ip访问

```
bind-address            = 0.0.0.0
mysqlx-bind-address     = 0.0.0.0
```



### 3.命令行连接数据库正常



## 二，问题解决



### 问题出在user表的Host一项

![](./img/88.png)



### 这里要修改以下允许所有ip

```mysql
update user set Host='%' where User='wenyongsheng';
```



### 再刷新一下

```mysql
flush privileges;
```



### 接着报了Access denied for user 'wenyongsheng'@'%' to database 'JIULOU'



### 这里要查看用户授权信息：

```mysql
show grants;
GRANT ALL PRIVILEGES ON JIULOU.* TO 'wenyongsheng'@'%' WITH GRANT OPTION;
flush privileges;
```

### 然后问题解决了

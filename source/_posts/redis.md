---
title: 在Ubuntu上部署redis
date: 2023/6/10 20:00
tags: redis
categories: redis
top_img: ./img/41.jpg
cover: ./img/34.jpg
---



# 在Ubuntu上部署Redis

### 1.安装Redis：在终端中运行以下命令来安装Redis：

```
sudo apt update
sudo apt install redis-server
```

### 2.配置Redis：Redis的默认配置文件位于`/etc/redis/redis.conf`。你可以使用文本编辑器打开该文件进行配置：

```
vim /etc/redis/redis.conf
```
```
#配置密码
requirepass xxxxxxxx
#配置ip和端口
bind 127.0.0.1
#启用AOF持久化
appendonly yes
```

在配置文件中，可以进行一些常见的配置更改，例如绑定地址、端口号等。确保保存更改并关闭配置文件。

### 3.启动Redis：安装完成后，Redis服务会自动启动。你可以使用以下命令来检查Redis是否正在运行：

```
sudo systemctl status redis-server
```
如果状态显示为“active”表示Redis正在运行。

```
#重启redis
sudo systemctl restart redis-server
```

### 4.测试连接：使用Redis客户端工具测试Redis连接。运行以下命令启动Redis客户端：

```
redis-cli
```
可以使用Redis命令来进行操作，例如：
```
set key value   # 设置键值对
get key         # 获取键的值
```
测试完毕后，可以使用`exit`命令退出Redis客户端。






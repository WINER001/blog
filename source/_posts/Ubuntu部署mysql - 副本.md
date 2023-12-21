---
title: Ubuntu部署mysql数据库
date: 2023/7/7 9:18
tags: mysql
categories: mysql
top_img: ./img/41.jpg
cover: ./img/66.jpg
---



# Ubuntu部署mysql数据库



### 要在Ubuntu上部署MySQL数据库，可以按照以下步骤进行操作：



1. ### 更新软件包列表：打开终端，运行以下命令更新软件包列表。

   ```
   sudo apt update
   ```

2. ### 安装MySQL服务器：运行以下命令安装MySQL服务器。

   ```
   sudo apt install mysql-server
   ```

3. ### 设置密码。

   

   1. #### 连接到MySQL服务器：
      
      ```
      sudo mysql
      ```
      
   2. #### 更改root用户的密码：
      
      ```
      ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
      ```
      
      将"新密码"替换为您希望设置的新密码。
      
   3. #### 刷新权限：
      
      ```
      FLUSH PRIVILEGES;
      ```
      
   4. #### 退出MySQL服务器：
      
      ```
      exit
      ```

   

4. ### 配置MySQL服务器：安装完成后，MySQL服务器会自动启动。以运行以下命令进行基本的安全配置。

   ```
   sudo mysql_secure_installation
   ```

   此命令将引导您完成一些安全设置，例如删除匿名用户、禁用远程登录root用户等。暂时全都N。

   

5. ### 连接到MySQL服务器：运行以下命令连接到MySQL服务器。

   ```
   sudo mysql -u root -p
   ```

   这将以root用户身份连接到MySQL服务器。

   

6. ### 创建数据库和用户：可以使用以下命令创建新的数据库和用户，并为该用户授予适当的权限。在下面的示例中，将数据库名设置为"mydatabase"，用户名设置为"myuser"，密码设置为"mypassword"，但可以根据需要进行修改。

   ```
   CREATE DATABASE mydatabase;
   CREATE USER 'myuser'@'localhost' IDENTIFIED BY 'mypassword';
   GRANT ALL PRIVILEGES ON mydatabase.* TO 'myuser'@'localhost';
   FLUSH PRIVILEGES;
   ```

   这将创建一个名为"mydatabase"的数据库，创建一个名为"myuser"的用户，并授予该用户对"mydatabase"数据库的全部权限。

   

7. ### 退出MySQL服务器：运行以下命令退出MySQL服务器。

   ```
   exit
   ```

现在，已经在Ubuntu上成功部署了MySQL数据库。可以使用刚创建的用户凭据连接到数据库并进行操作。


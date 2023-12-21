---
title: 用redis作为存用户信息的数据库
date: 2023/6/10 21:18
tags: redis
categories: redis
top_img: ./img/41.jpg
cover: ./img/31.jpg
---



# 用redis作为存用户信息的数据库



## 一，数据库设计



### 1.使用哈希表（Hash）存储每个用户的信息，键为用户ID，值为用户的详细信息。用户ID可以是自动生成的唯一标识符，例如使用 UUID。

```plaintext
Hash:
"user:<user_id>": {
  "account_name": "user123",
  "nickname": "JohnDoe",
  "role": "admin",
  "department": "IT",
  "phone": "1234567890",
  "email": "user@example.com",
  "status": "active",
  "description": "This is a user",
  "created_at": "2023-06-10T10:00:00Z"
}
```

这样的设计允许以用户ID为键快速访问和修改用户信息。可以通过用户ID直接获取用户的详细信息，并使用哈希表提供的操作来对用户信息进行增、删、改、查等操作。

### 2.使用集合（Set）存储不同角色和部门的用户集合，方便按角色和部门进行查询。

```plaintext
Set: "role:admin"  // 存储拥有 "admin" 角色的用户ID集合
Set: "role:common"  // 存储拥有 "common" 角色的用户ID集合
Set: "department:Develop"  // 存储所属 开发 部门的用户ID集合
Set: "department:Test"  // 存储所属 测试 部门的用户ID集合
```

使用集合可以方便地对用户进行角色和部门的分类。可以将用户ID添加到相应的集合中，以便根据角色和部门进行查询和统计。



## 二，数据库操作

### 1.对Hash进行操作

#### 1.添加/更新用户信息：

使用 Redis 的 HSET 命令来设置用户的字段和值。

```
HSET user:<user_id> account_name user123 nickname JohnDoe role admin department IT phone 1234567890 email user@example.com status active description "This is a user" created_at "2023-06-10T10:00:00Z"
```



#### 2.获取用户信息：

使用 Redis 的 HGETALL 命令来获取用户的所有字段和值。

```
HGETALL user:<user_id>
```



#### 3.获取特定字段的值：

使用 Redis 的 HGET 命令来获取用户的指定字段的值。

```
HGET user:<user_id> account_name
```

#### 4.更新用户字段的值：

使用 Redis 的 HSET 命令来更新用户的指定字段的值。

```
HSET user:<user_id> nickname NewNickname
```

#### 5.删除用户信息：

使用 Redis 的 DEL 命令来删除用户的哈希键。

```
DEL user:<user_id>
```



### 2.对Set进行操作

#### 	1.将用户ID添加到 "role:admin" 集合中：

- 使用 Redis 的 SADD 命令将用户ID添加到 "role:admin" 集合中。
- 示例命令：SADD role:admin 11111 22222 33333

#### 	2.查询 "admin" 角色下的用户：

- 使用 Redis 的 SMEMBERS 命令可以获取 "role:admin" 集合中的所有成员（用户ID）。
- 示例命令：SMEMBERS role:admin




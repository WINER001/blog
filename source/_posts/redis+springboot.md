---
title: Redis+Springboot
date: 2023/6/15 21:18
tags: redis
categories: redis
top_img: ./img/41.jpg
cover: ./img/41.jpg
---





# Redis+Springboot



### 1.当在 Spring Boot 项目中使用 Redis，需要在 `application.yml` 或 `application.properties` 文件中配置 Redis 连接信息。以下是一个示例的 `application.yml` 配置示例：

```yaml
spring:
  redis:
    host: localhost
    port: 6379
    password: your_password
    database: 0
```

在上面的配置中，可以将 `host` 设置为 Redis 服务器的主机地址，`port` 设置为 Redis 服务器的端口号，`password` 设置为 Redis 服务器的访问密码（如果有的话），`database` 设置为 Redis 服务器的数据库索引号（默认为 0）。



### 2.新建UserRepository接口实现CrudRepository：

```java
public interface UserRepository extends CrudRepository<RedisUser,String>{
    
}
```



### 3.创建一个RedisHash实体类RedisUser

```java
package com.yuchengji.cn.common.bo;

import org.springframework.data.annotation.Id;
import org.springframework.data.redis.core.RedisHash;

@RedisHash("user")
public class RedisUser {
    @Id
    private String id;
    
    private String accountname;

    private String nickname;
    
    private String role;

    private String department;

    private String phone;

    private String email;

    private String status;

    private String description;

    private String createdtime;

    public String getId() {
        return id;
    }

    public void setId(String id) {
        this.id = id;
    }

    public String getAccountname() {
        return accountname;
    }

    public void setAccountname(String accountname) {
        this.accountname = accountname;
    }

    public String getCreatedtime() {
        return createdtime;
    }

    public void setCreatedtime(String createdtime) {
        this.createdtime = createdtime;
    }

    public String getDescription() {
        return description;
    }

    public void setDescription(String description) {
        this.description = description;
    }

    public String getNickname() {
        return nickname;
    }

    public void setNickname(String nickname) {
        this.nickname = nickname;
    }

    public String getRole() {
        return role;
    }

    public void setRole(String role) {
        this.role = role;
    }

    public String getDepartment() {
        return department;
    }

    public void setDepartment(String department) {
        this.department = department;
    }

    public String getPhone() {
        return phone;
    }

    public void setPhone(String phone) {
        this.phone = phone;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public String getStatus() {
        return status;
    }

    public void setStatus(String status) {
        this.status = status;
    }

}

```



### 4.使用UserRepository.findById(userId).orElse(null)调用

```java
    @Autowired
    UserRepository redisService;

    //private static final String KEY_PREFIX = "user:";

    @PostMapping("/query")
    public String query(@RequestBody UserMessage msg) {
        FabricLogger.info(this,"****** step into redis query ******");
        String request = JsonUtils.toJson(msg);
        FabricLogger.info(this,"redis controller query receive request :"+request);

        String result = "query fail...";
        String action = msg.getAction();
        String userId = msg.getUserId();

        FabricLogger.info(this,"query user id : "+userId);
        if("query".equals(action)&&null != userId){
            //查询Redis
            RedisUser resultUser =redisService.findById(userId).orElse(null);
            
            result = JsonUtils.toJson(resultUser);
        }else{
            result = "request format error";
        }
        FabricLogger.info(this,"****** step out redis query ******");
        return result;
    }
```



### 5.发送HTTP请求调用

response body :
{"id":"100002","accountname":"100002","nickname":"Tom","role":"admin","department":"IT","phone":"1234567890","email":"user@example.com","status":"active","description":"This is a user","createdtime":"2023-06-10T10:00:00Z"}

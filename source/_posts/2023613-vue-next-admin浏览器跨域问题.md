---
title: vue-next-admin浏览器跨域问题
date: 2023/6/13 9:17
tags: vue-next-admin
categories: vue-next-admin
top_img: ./img/41.jpg
cover: ./img/13.jpg

---

# vue-next-admin浏览器跨域问题



### 默认情况下，vue-next-admin后台管理系统启动后，调用后台api会报400，这是由于默认情况下客户端和服务端跨域是不允许的。



### 解决方式：

```java
package com.yuchengji.cn.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurer;

@Configuration
public class WebConfig implements WebMvcConfigurer{
    
    @Override
    public void addCorsMappings(CorsRegistry registry){
        //匹配所有的请求路径
        registry.addMapping("/**")
            //配置允许访问资源的域名
            .allowedOrigins("*")
            //配置允许访问的HTTP方法
            .allowedMethods("*")
            //配置允许访问的请求头
            .allowedHeaders("*");
    }
}

```

直接在springboot中加个这个配置，可以根据自己的需要进行配置。
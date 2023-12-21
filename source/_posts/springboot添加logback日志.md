---
title: SpringBoot添加logback日志
date: 2023/5/25 13:17
tags: Spring
categories: Spring
top_img: ./img/41.jpg
cover: ./img/29.jpg
---

## SpringBoot添加logback日志

### 1.pom文件中新增依赖

```xml
		<dependency>
			<groupId>ch.qos.logback</groupId>
    		<artifactId>logback-classic</artifactId>
    		<version>1.2.3</version>
		</dependency>
		<dependency>
			<groupId>org.slf4j</groupId>
			<artifactId>slf4j-api</artifactId>
			<version>1.7.32</version>
		</dependency>
```

### 2.在src/main/resources目录新增配置文件logback.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="true">
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- <file>${user.dir}/app.log</file> -->
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>${user.dir}/logback/app.%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level %logger{35} - %msg%n</pattern>
        </encoder>
    </appender>
    <logger name="com.yuchengji" level="DEBUG"/>
    <root level="INFO">
        <appender-ref ref="FILE"/>
        <appender-ref ref="STDOUT"/>
    </root>

</configuration>

```


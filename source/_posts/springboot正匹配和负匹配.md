---
title: SpringBoot正匹配和负匹配
date: 2023/6/13 13:17
tags: Spring
categories: Spring
top_img: ./img/41.jpg
cover: ./img/29.jpg
---



## SpringBoot正匹配和负匹配

在Spring框架的自动配置中，"Positive matches"（正匹配）和"Negative matches"（负匹配）是用于条件化配置的概念。

- Positive matches（正匹配）指的是满足条件的情况。当某个条件（使用`@ConditionalOn...`注解）得到满足时，相应的自动配置将会生效。这意味着条件的结果为true，符合条件的类、依赖或配置存在，从而允许相应的自动配置加载和应用。

- Negative matches（负匹配）指的是不满足条件的情况。当某个条件不满足时，相应的自动配置将不会生效。这意味着条件的结果为false，或者符合条件的类、依赖或配置不存在，从而阻止相应的自动配置加载和应用。

这些正负匹配的机制用于根据项目的实际情况自动启用或禁用某些配置，以满足特定的需求。

在Spring Boot中，可以通过使用`@ConditionalOn...`系列注解来定义条件，如`@ConditionalOnClass`、`@ConditionalOnBean`、`@ConditionalOnProperty`等。这些条件可以根据类的存在与否、Bean的存在与否、属性的值等来判断是否满足条件。

通过正负匹配的机制，Spring Boot可以智能地自动配置应用程序的各个部分，根据项目的依赖和配置情况来进行灵活的自动装配，提供了方便的开发体验和可扩展性。

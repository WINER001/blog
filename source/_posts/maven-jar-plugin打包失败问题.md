---
title: maven-jar-plugin打包失败问题
date: 2023/5/10
tags: 疑难杂症
categories: 开发语言
top_img: ./img/41.jpg
cover: ./img/66.jpg

---



## 1.问题描述

事情的最开始是因为在maven打包jar以后，用java -jar执行jar文件的时候找不到main，因此pom加了个这个

```xml
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-jar-plugin</artifactId>
          <version>2.4</version>
          <configuration>
            <archive>
              <manifest>
                <mainClass>com.yuchengji.cn.WebApplication</mainClass>
              </manifest>
            </archive>
          </configuration>
        </plugin>
```

加了这个以后在我的笔记本上可以打包成功，但是在我的台式电脑上就不行了

![](./img/43.png)

## 2.问题解决过程

首先，我确认了一下笔记本和电脑上的java版本和maven版本，都是jdk1.8和maven3.9.1，因此排除了开发环境的版本问题

然后，代码都是从git库中更新的最新代码，代码是没有不一样的，也不是代码的问题。

接着，我注意到报错信息里2.4的字样，怀疑maven-jar-plugin的2.4版本是不是有问题，然后我换成了3.2.0，打包成功！

![](./img/43.png)

## 3.问题后续

那问题就来了，为什么我的笔记本上能打包成功？

BOMB!你猜怎么着，经过我细细的考察，其实pom里面根本就不用maven-jar-plugin，这个是打包spring项目的，springboot项目用的是spring-boot-maven-plugin，两者的关系：spring-boot-maven-plugin是在maven-jar-plugin的基础上做的，是maven-jar-plugin的儿子。

而之前的找不到main，其实是应该在spring-boot-maven-plugin中添加repackage，加了这个，springboot项目所依赖的那些jar包会在项目构建的时候也打包进最后生成的jar里，要不然的话，在服务器上使用java -jar 来运行项目的时候得把本项目依赖的其他jar包也放在这个命令行的后面！

![](./img/45.png)

还有在构建的过程中很恶心的一点是springboot的版本，要和java版本匹配，我用的是jdk1.8，与之匹配的版本是2.6.6



最后，就是pom文件的格式问题，一定要好好检查，不要随便使用空格，要用tab，一个格式不对，就会编译失败，格式化有时候也不好用，要自己去确认。

## 4.问题总结

这个故事告诉我们，在pom文件中添加东西一定要谨慎，要先了解好新添加的依赖与自己已经有的依赖是否重复，是什么关系，是否冲突，pom文件一定要干净，每个依赖都是必须，可溯源的。要不然就很酸爽了。

还有，遇到一个问题，在网上找原因的时候，不能直接把解决方式套过来用，一个不小心，之前的问题没解决，有生出了其他问题，构建不了了...
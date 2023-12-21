---
title: 初始化modules过程中遇到的问题
date: 2023/5/21 18:01
tags: 
  - go语言
  - 疑难杂症
categories: 开发语言
top_img: ./img/41.jpg
cover: ./img/41.png
 
---

## 在执行 go mod init初始化modules命令时报错

## go: cannot determine module path for source directory /home/fabric_contract (outside GOPATH, module path must be specified)




该错误通常是由于Go模块的路径问题引起的。在Go 1.11及更高版本中，Go引入了模块化开发的概念，需要明确指定模块路径。

根据您提供的错误消息，它表明您的项目位于`/home/fabric_contract`目录下，但它不在您的GOPATH内，因此需要指定模块路径。

尝试一下加路径 

```sh
go mod init /home/fabric_contract
```

不行！

后来又试了一下

```sh
go mod init fabric_contract
```

好用了，这个初始化不用写绝对路径，写当前目录就行。
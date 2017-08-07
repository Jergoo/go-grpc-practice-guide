# Go-gRPC 实践指南

## 初衷

2016年年中左右，因工作需要开始了解gRPC，当时gRPC还是beta版本，在国内搜索到的资料基本上都是一些介绍性质的文章，最完整的资料就是官方文档了，但是碍于英文水平有限，并且刚刚接触golang不久，所以学习的过程还是花了些时间的。最开始就被protobuf的编译给难住了，按照官网的说明一步步操作总是有问题。经过和领导的一起努力，总算是把它在公司的业务中用起来了。后来领导鼓励我把学习的过程写出来，分享给更多的人，因拖延症一直快到年底了我才整理出了一些内容，涵盖了protobuf语法介绍，gRPC的基本使用方式等内容。再后来就发到了segmentfault上连载了 **Golang gRPC实践** 系列文章，收到了好多朋友的好评。

现在一年过去了，gRPC也更新了好多个版本，用户也越来越多，生态也越来越丰富，回过头再看以前写的东西，感觉应该重新整理一下了，所以就开始了这个项目。和原来一样，内容会保持尽可能的简单，写作的目的就是给广大和我一样的小白开发者提供一个实用的指南。


## 内容安排

这次重新整理稍微调整了一下目录结构，删掉了部分多余的内容，补充了以前遗留下的TODO。计划介绍更多的gRPC实践及生态的用法和多语言支持。重点还是留在实践的章节部分，多语言支持部分暂时计划简单介绍使用方法。内容中的示例源码都放在项目 **[go-grpc-example](https://github.com/jergoo/go-grpc-example)** 内，如无特殊说明，内容的源码的目录说明将以此为根目录。

**众人拾柴火焰高，我一个人的能力和时间是有限的，欢迎有心人加入一起完成这个小事业~**

## 进度

* 介绍
    * [x] gRPC简介
    * [x] 安装
    * [x] Protobuf语法
    * [x] Protobuf⇢Go转换
* 实践
    * [x] Hello gRPC
    * [x] 认证
    * [ ] 拦截器
    * [ ] 流式传输
    * [ ] 内置Trace
    * [ ] 负载均衡
* gRPC生态
    * [ ] Http转换
* 多语言支持
    * [ ] Java
    * [ ] PHP
    * [ ] Python
    * [ ] Node.js

## 推广

### Go web框架

* [go-baa](https://github.com/go-baa/baa)


### segmentfault连载

* [Golang gRPC实践 连载一 gRPC介绍与安装](https://segmentfault.com/a/1190000007880647)
* [Golang gRPC实践 连载二 Hello gRPC](https://segmentfault.com/a/1190000007909829)
* [Golang gRPC实践 连载三 Protobuf语法](https://segmentfault.com/a/1190000007917576)
* [Golang gRPC实践 连载四 gRPC认证](https://segmentfault.com/a/1190000007933303)
* [Golang gRPC实践 连载五 拦截器 Interceptor](https://segmentfault.com/a/1190000007997759)
* [Golang gRPC实践 连载六 内置Trace](https://segmentfault.com/a/1190000008087436)  
* [Golang gRPC实践 连载七 http转换](https://segmentfault.com/a/1190000008106582)

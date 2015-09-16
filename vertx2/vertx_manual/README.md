# Vertx Manual

# Introduction

## What is Vert.x?

`Vert.x` 是一个运行在JVM里多语言实现的,非阻塞的,事件驱动的应用程序平台.

Some of the key highlights include:

* 多语言：你可以在一个应用程序里混合地使用`JavaScript, Ruby, Groovy, Java or Python`等多种语言实现应用程序组件.

* 与`actor`类似的并发模型. `Vert.x`允许你写代码时不用考虑并发问题, 让你避免了在编写多线程程序中遇到的各种坑

* `Vert.x` 利用JVM的高级特性能根据可用核心数上进行无缝扩展,而不必要手动地交叉部署多个服务器,再让服务器通过进程通信.

* `Vert.x` 有一个的简单异步编程模型,可以轻松实现非阻塞,可扩展的应用程序, 通过最少数量的os线程就可以轻松百万并发连接.

* `Vert.x` 内嵌了一个分布式的event bus, 它横跨客户端和服务器, 你的应用程序组件可以轻松实现交互.

* `Vert.x` 提供的功能非常强大又不失简洁, 而且尽量将配置之类的东西做到最小化

* `Vert.x` 提供了一个非常强大的`module`系统以及一个公共的`module`仓库,所以你可以轻松的与他们复用, 分享`module`

* `Vert.x` 也可以内嵌进你自己现有的java应用程序中.


## Internals

`Vert.x`使用了下面的开源项目

* `Netty` for much of its network IO
* `JRuby` for its Ruby engine
* `Groovy`
* `Mozilla Rhino` for its JavaScript engine
* `Jython` for its Python engine
* `Hazelcast` for group management of cluster members


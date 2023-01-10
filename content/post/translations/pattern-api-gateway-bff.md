---
title: "[译] 模式：API网关和BFF"
description: 试想，你的团队正在搭建一个在线商城，它使用了微服务架构模式，现在需要你实现产品的信息细节页面。针对这个产品信息页面，你需要开发多个版本的用户接口：
date: 2020-09-03T14:50:29+08:00
image: 
math: 
license: 
hidden: false
comments: true
categories:
    - 译文
tags:
    - Web
---

> - 原文地址：[Pattern: API Gateway / Backends for Frontends](https://microservices.io/patterns/apigateway.html)
> - 原文作者：[Chris Richardson](https://microservices.io/about.html)
> - 译文出自：[Microservice Architecture](https://microservices.io/)
> - 译者：[Arrackisarookie](https://github.com/Arrackisarookie)

## 情景

试想，你的团队正在搭建一个在线商城，它使用了[微服务架构模式](https://microservices.io/patterns/microservices.html)，现在需要你实现产品的信息细节页面。针对这个产品信息页面，你需要开发多个版本的用户接口：

+ 基于 `HTML5/JavaScript` 适用于桌面和手机浏览器的用户接口 - HTML 由服务器端的 Web 应用程序生成
+ 本地的 `Android` 和 `iPhone` 客户端 - 这些客户端通过 `REST APIs` 与服务器进行交互

另外，这个在线商城必须可以通过一个 `REST API` 暴露产品信息，以供第三方应用使用获取。

产品信息页会展示大量有关该产品的信息。比如 _Amzon.com_ 上 [POJOs in Action](http://www.amazon.com/POJOs-Action-Developing-Applications-Lightweight/dp/1932394583) 这本书的信息页展示了：

+ 这本书的基本信息，像标题，作者，价格等等
+ 你对于这本书的支付历史
+ 是否可以购买(库存)
+ 购买选项
+ 和这本书经常一起购买的书
+ 买了这本书的其他买家还买了哪些书
+ 买家评论
+ 卖家排行
+ ...

由于这个在线商城使用了微服务架构模式，产品的具体信息被分布在了多个微服务上。比如：

+ 产品信息服务 - 该产品的基本信息，类似标题，作者
+ 价格服务 - 产品的定价
+ 订单服务 - 产品的支付历史
+ 库存服务 - 产品是否可购买
+ 评论服务 - 买家评论

这样一来，展示产品信息的代码需要从以上所有服务中获取信息。

## 问题

一个基于微服务的应用程序客户端如何访问一个个独立的服务呢？

## 痛点

+ 由微服务提供的 APIs，它的粒度经常和客户端所需的粒度有所不同。微服务一般会提供细粒度的 APIs，这也就意味着客户端需要和很多微服务产生交互。就像上面情境中描述的，客户端如果需要展示的产品信息细节，就需要从大量的服务中获取数据
+ 不同的客户端需要不同的数据。比如，桌面浏览器版本的产品信息页要比手机版本的更加细致丰富。
+ 对于不同类型的客户端，网络性能也是不同的。比如，手机网络一般要比非手机网络更慢而且延时更高。当然，广域网也会比局域网慢很多。这也就意味着使用手机网络的移动客户端和使用局域网的服务器端 Web 应用程序将有着差异非常大的性能特点。服务器端的 Web 应用程序可以同时处理多个传到后端的请求，并且不会影响到用户的体验，而手机客户端只能做到一点点
+ 服务实例的数量和他们地址(主机地址+端口)的动态改变
+ 拆分成的多个服务可能会随着时间改变拆分方式，这些对于客户端应该是隐藏的
+ 众多的服务可能会使用各种各样的协议，其中有些可能对网络不友好

## 解决

实现一个 API 网关，让它作为所有的客户端访问服务器的唯一入口。API 网关处理请求一般有两种方式，一些请求会被简单的代理到或路由到合适的服务，而对于另外一些服务，网关可能会同时分发给多个服务([fan out to multiple services](https://en.wikipedia.org/wiki/Fan-out_(software)))。

![Use an API gateway](https://microservices.io/i/apigateway.jpg)

不同于提供一套适用于所有类型的通用 API，我们的 API 网关可以为每一种客户端暴露不同的 API 接口。比如，[Netflix API]() 运行着一套客户端识别适配代码，它可以为每一种客户端提供其所需的最合适的 API 接口。

API 网关可能也会实现安全措施，例如验证客户端是否被授权可以执行该请求。

## 变种：服务于前端的后端(Backends for frontends - BFF)

上面所说的这种模式有个变种形式，也就是 `BFF` 模式。它为每个类型的客户端定义了一套专门的 API 接口。

![Variation: Backends for frontends](https://microservices.io/i/bffe.png)

在这个例子中，有三种客户端：Web 应用，移动应用和外部第三方应用。同样，也有三种不同的 API 网关，它们和之前提到的三种客户端一一对应。

## 结论

使用 API 网关有以下优势：

+ 客户端将与应用后台如何划分微服务完全隔离
+ 客户端将与决定服务实例位置的问题完全隔离
+ 为每一种客户端提供最佳的 API
+ 削减了大量的请求和往返次数。比如，API 网关允许客户端在一次往返中从多个服务中获取数据。更少的请求次数也意味着更少的资源开销，和更优质的用户体验。API 网关对于移动应用来说十分必要
+ 简化了客户端。改变了客户端的运行逻辑，客户端不再需要调用多个服务，而是把这一切工作移给了 API 网关
+ 它可以将标准的公共网络友好的 API 协议转换成内部使用的任意协议

API 网关也有它的劣势：

+ 复杂性提升了 - API 网关事实上是一个独立的可插拔部件，它也必须需要开发，发布和管理
+ 增加了响应时间 - 由于经过 API 网关时增加了额外的网络跳转，所以响应时间会有所增加。但是对于大部分应用来说，这一点增加的开销是微不足道的。

议题

+ 如何实现 API 网关呢？如果该网关必须按照比例缩放以应对高负载，那么最好采用事件驱动或响应式的方法。在 `JVM`，或者像 `Netty`，`Spring Reactor` 这样的基于 `NIO` 的库效果很好。`NodeJS` 也是一种选择。

## 相关的模式

+ [微服务架构模式(Microservice architecture pattern)](https://microservices.io/patterns/microservices.html) 为 API 网关模式创造了需求
+ API 网关必须使用 [客户端探索模式(Client-side Discovery pattern)](https://microservices.io/patterns/client-side-discovery.html) 或者 [服务器端探索模式(Server-side Discovery pattern)](https://microservices.io/patterns/server-side-discovery.html) 来将请求路由到可用的服务实例
+ API 网关可以验证用户身份，并且会将一个包含用户信息的 [访问令牌(Access Token)](https://microservices.io/patterns/security/access-token.html) 传送给服务
+ API 网关需要使用 [断路器模式(Circuit Breaker)](https://microservices.io/patterns/reliability/circuit-breaker.html) 来调用服务
+ API 网关经常会实现 [API 组合模式(API Composition pattern)](https://microservices.io/patterns/data/api-composition.html)

## 已知在使用的

+ [Netflix API 网关](http://techblog.netflix.com/2012/07/embracing-differences-inside-netflix.html)

## 应用范例

请参阅微服务模式中 [应用程序范例](https://github.com/microservice-patterns/ftgo-application) 的 API 网关部分。它使用 Spring Cloud Gateway 实现。

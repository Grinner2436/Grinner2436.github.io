---
layout: post
title: Netty学习
categories:
  - Netty
tags:
  - 《Netty》
note: Netty学习
---
# Netty的组件模型

## ChannelHandler 的用法
### 车辆共享
```
因为一个ChannelHandler 可以从属于多个ChannelPipeline，所以它也可以绑定到多个ChannelHandlerContext 实例。对于这种用法指在多个ChannelPipeline 中共享同一个ChannelHandler，对应的ChannelHandler 必须要使用@Sharable 注解标注；否则，试图将它添加到多个ChannelPipeline 时将会触发异常。显而易见，为了安全地被用于多个并发的Channel（即连接），这样的ChannelHandler 必须是线程安全的。
```
车辆是可以在多条道路上跑的，当车辆被租用，则可能被其他司机开，因为租车的司机才熟悉当地的路，和当地交通部门有联系。

租赁分享的车必须是线程安全的，不能因为某一个司机把车子弄脏，导致别人没法用这辆车。

### 
```
为何要共享同一个ChannelHandler 在多个ChannelPipeline中安装同一个ChannelHandler
的一个常见的原因是用于收集跨越多个Channel 的统计信息。
```

## ChannelHandlerContext的用法
### Netty事件流
当某一辆车的司机（ChannelHandlerContext）通过与交通部门（ChannelPipeline）或者道路部门（Channel ）联系通知一处塌方时，对方总是从第一个司机开始，让他们口口相传直到全部司机知晓。

如果想阻止这个全线流程，则需要直接联系对应的司机（ChannelHandlerContext），让他向后传递给必要的人就行了。

### 自发的司机协会
```
 另一种高级的用法是缓存到ChannelHandlerContext 的引用以供稍后使用，这可能会发生在任何的ChannelHandler 方法之外，甚至来自于不同的线程。
```
也就是说可以脱离Netty的范围管理ChannelHandlerContext ，使各位司机成立独立组织来进行各种控制。

## 紧急情况处理
```
如果在处理入站事件的过程中有异常被抛出，那么它将从它在ChannelInboundHandler里被触发的那一点开始流经ChannelPipeline。

因为异常将会继续按照入站方向流动（就像所有的入站事件一样），所以实现了前面所示逻辑的ChannelInboundHandler 通常位于ChannelPipeline 的最后。这确保了所有的入站异常都总是会被处理，无论它们可能会发生在ChannelPipeline 中的什么位置。

如果你不实现任何处理入站异常的逻辑（或者没有消费该异常），
那么Netty将会记录该异常没有被处理的事实①
```
入站异常类似于一辆车上有个人病倒了，但是这辆车上没有医生，怎么办？到站之后转给下一辆车。

异常处理不是强制的，在一般情况下总安排给最后一辆车，保底。


## Netty协议示例
https://github.com/netty/netty/tree/4.1/example

Netty是对网络通信的封装，是取代java.net之类的包的东西。这意味着Netty是很底层的框架，而不是具体的方法。

如何使用Netty可以结合Jedis对于Netty的使用来学习，里面实现了Redis协议。
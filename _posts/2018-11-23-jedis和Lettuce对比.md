---
title: jedis和Lettuce对比
categories:
 - DB
tags: 
 - redis
 - Java
---

## Jedis
Jedis在实现上是直接连接Redis-Server，
在多个线程间共享一个Jedis实例时是线程不安全的，
如果想要在多线程场景下使用Jedis，
需要使用连接池，每个线程都使用自己的Jedis实例，
当连接数量增多时，会消耗较多的物理资源。

## Lettuce
Lettuce是一个可伸缩的线程安全的Redis客户端，
支持同步、异步和响应式模式。
多个线程可以共享一个连接实例，而不必担心多线程并发问题。
它基于优秀Netty NIO框架构建，支持Redis的高级功能，
如Sentinel，集群，流水线，自动重新连接和Redis数据模型。

具体的实现细节需要再看下，这里记录一下
//TODO
---
title: SpringCloud(4)hystrix相关
categories:
 - Java
tags: 
 - Java
---

hystrix的功能以及内容比较多，涉及到资源隔离(限流)，服务降级，超时以及熔断。
一点一点的来说。

## 资源隔离(限流)实现
线程池，信号量

## 服务降级
fallbackMethod

## 熔断
异常次数，异常比例


![hystrix-dashboard](https://raw.githubusercontent.com/xuguangwu/xuguangwu.github.io/master/img/in-post/springcloud/hystrix-dashboard.png)


## 参数
1. Execution相关的属性的配置

    |配置|说明|
    |----|----|
    |hystrix.command.default.execution.isolation.strategy | 隔离策略，默认是Thread, 可选Thread|Semaphor |
    |hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds  |命令执行超时时间，默认1000ms |
    |hystrix.command.default.execution.timeout.enabled |执行是否启用超时，默认启用true |
    |hystrix.command.default.execution.isolation.thread.interruptOnTimeout |发生超时是是否中断，默认true|
    |hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests | 最大并发请求 数，默认10，该参数当使用ExecutionIsolationStrategy.SEMAPHORE策略时才有效。如果达到最大并发请求数，请求会被拒绝。理论上选择semaphore size的原则和选择thread size一致，但选用semaphore时每次执行 的单元要比较小且执行速度快(ms级别)，否则的话应该用thread。 semaphore应该占整个容器(tomcat)的线程 池的一小部分。 Fallback相关的属性 这些参数可以应用于Hystrix的THREAD和SEMAPHORE策略|
    |hystrix.command.default.fallback.isolation.semaphore.maxConcurrentRequests | 如果并发数达到 该设置值，请求会被拒绝和抛出异常并且fallback不会被调用。默认10
    |hystrix.command.default.fallback.enabled |当执行失败或者请求被拒绝，是否会尝试调用 hystrixCommand.getFallback() 。默认true

2. Circuit Breaker相关的属性

    |配置|说明|
    |----|----|
    |hystrix.command.default.circuitBreaker.enabled | 用来跟踪circuit的健康性，如果未达标则让 request短路。默认true |
    |hystrix.command.default.circuitBreaker.requestVolumeThreshold |一个rolling window内最小的 请 求数。如果设为20，那么当一个rolling window的时间内(比如说1个rolling window是10秒)收到19个请求， 即使19个请求都失败，也不会触发circuit break。默认20
    |hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds |触发短路的时间值，当该值设 为5000时，则当触发circuit break后的5000毫秒内都会拒绝request，也就是5000毫秒后才会关闭circuit。 默认5000
    |hystrix.command.default.circuitBreaker.errorThresholdPercentage| 错误比率阀值，如果错误率>=该值，circuit会被打开，并短路所有请求触发fallback。默认50
    |hystrix.command.default.circuitBreaker.forceOpen |强制打开熔断器，如果打开这个开关，那么拒绝所 有request，默认false
    |hystrix.command.default.circuitBreaker.forceClosed |强制关闭熔断器 如果这个开关打开，circuit将 一直关闭且忽略circuitBreaker.errorThresholdPercentage

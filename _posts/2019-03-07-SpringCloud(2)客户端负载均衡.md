---
title: SpringCloud(2)客户端负载均衡
categories:
 - Java
tags: 
 - Java
---

官网提示引入ribbon需要引入spring-cloud-starter-netflix-ribbon依赖，
其实在eureka-client中已经做好了依赖处理，所以无需再次引入。

## 配置负载均衡策略
````java
@Bean
public IRule iRule() {
    //默认是RoundRobinRule
    return new RandomRule();
//    return new RetryRule(n ew RandomRule(), 1);
}
````

## 自定义负载均衡策略
实现IRule接口
````java
public interface IRule{
    public Server choose(Object key);
}
````

## 客户端针对不同服务配置不同负载均衡策略
@RibbonClients(value = {@RibbonClient(name = "SERVER-NAME1", configuration = Consumer1Configure.class),
                        @RibbonClient(name = "SERVER-NAME2", configuration = Consumer2Configure.class)})

## 客户端请求封装
1. 启动类添加@EnableFeignClients
2. 定义FeignClient
    ````java
    @FeignClient("SERVER-POWER")
    public interface PowerFeignClient {
    
        @RequestMapping("user")
        R getUser();
    
    }
    ````
3. controller中调用
    ````java
    @Resource
    private PowerFeignClient powerFeignClient;
    
    @RequestMapping("feign")
    public R getFeignUser() {
        return powerFeignClient.getUser();
    }
    ````

## feign整合hystrix
0. 配置feign.hystrix.enabled: true
1. @EnableHystrix
2. 定义降级方法
````java
@Component
public class FallBackMethod implements PowerFeignClient {

    @Override
    public R getUser() {
        return R.error("getUser fail");
    }
}
````
3. Feign实现类
````java
@FeignClient(value = "SERVER-POWER", fallback = FallBackMethod.class)
public interface PowerFeignClient {

    @RequestMapping("user")
    R getUser();

}
````




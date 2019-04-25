---
title: SpringCloud(5)sleuth以及zipkin相关
categories:
 - Java
tags: 
 - Java
---

分布式服务必须要处理好的几个问题：
* 如何串联调用链，快速定位问题
* 如何理清微服务之间的依赖关系
* 如何进行各个服务接口的性能分折
* 如何跟踪业务流的处理

springcloud中使用sleuth来收集数据，zipkin来展示数据。

1. zipkin server依赖
    ````xml
    <dependencies>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-autoconfigure-ui</artifactId>
            <version>2.8.4</version>
        </dependency>
        <dependency>
            <groupId>io.zipkin.java</groupId>
            <artifactId>zipkin-server</artifactId>
            <version>2.8.4</version>
        </dependency>
    </dependencies>
    ````
2. zipkin server的application.yml配置
    ````yaml
    management:
      metrics:
        web:
          server:
            autoTimeRequests: false
    ````
3. zipkin server启动类
    ````java
    import org.springframework.boot.SpringApplication;
    import org.springframework.boot.autoconfigure.SpringBootApplication;
    import zipkin.server.internal.EnableZipkinServer;
    
    @EnableZipkinServer
    @SpringBootApplication
    public class AppSleuth {
        public static void main(String[] args) {
            SpringApplication.run(AppSleuth.class);
        }
    }
    ````
4. 各个微服务上需要配置zipkin server地址以及采样数据
    ````yaml
    spring:
      zipkin:
        base-url: http://localhost:8080 #指定Zipkin server地址
      sleuth:
        sampler:
          probability: 1.0 # #request采样的数量的标准 默认是0.1，也就是10%，因为在分布式系统中，数据量可能会非常大，因此采样非常重要。我们示例数据少最好配置为1全采样

    ````
5. 使用elasticSearch做持久化
在zipkin server的yml中添加一下配置
    ````yaml
    zipkin:
      storage:
        type: elasticsearch
        elasticsearch:
          cluster: elasticsearch
          hosts: http://127.0.0.1:9200
          index: zipkin
    ````

我使用es7.0会抛出如下错误：
**Caused by: java.lang.IllegalStateException: response for update-template failed: {"error":{"root_cause":[{"type":"illegal_argument_exception","reason":"Setting index.mapper.dynamic was removed after version 6.0.0"}],"type":"illegal_argument_exception","reason":"Setting index.mapper.dynamic was removed after version 6.0.0"},"status":400}**
待解决

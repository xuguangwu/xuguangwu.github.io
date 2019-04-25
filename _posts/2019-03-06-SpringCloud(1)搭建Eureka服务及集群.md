---
title: SpringCloud(1)搭建Eureka服务及集群
categories:
 - Java
tags: 
 - Java
---

我将使用maven来构建整个工程。一步一步搭建spring-cloud全家桶。

## 单点配置
parent-pom文件配置
````xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
</parent>

<dependencies>
    <dependency>
        <groupId>com.google.code.gson</groupId>
        <artifactId>gson</artifactId>
    </dependency>
</dependencies>


<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-dependencies</artifactId>
            <version>Finchley.SR2</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
````

eureka项目的pom配置
````xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
</dependencies>
````

eureka的application.yml配置
````yaml
server:
  port: 3000
eureka:
  server:
    enable-self-preservation: false #关闭自我保护机制
    eviction-interval-timer-in-ms: 4000 #设置清理间隔(单位:毫秒 默认是60*1000)
  instance:
      hostname: localhost
  client:
    registerWithEureka: false #不把自己作为一个客户端注册到自己身上
    fetchRegistry: false #不需要从服务端获取注册信息(因为在这里自己就是服务端，而且已经禁用自己注册了)
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka
````

服务启动类
````java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class EurekaApplication {

    public static void main(String[] args) {
        SpringApplication.run(EurekaApplication.class);
    }

}
````

client端配置
````xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
````

application.yml配置
````yaml
server:
  port: 5000
eureka:
  client:
    serviceUrl:
      defaultZone: http://127.0.0.1:3000/eureka  #eureka服务端提供的注册地址 参考服务端配置的这个路径
  instance:
    instance-id: user-1 #此实例注册到eureka服务端的唯一的实例ID
    prefer-ip-address: true #是否显示IP地址
    leaseRenewalIntervalInSeconds: 10 #eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒 (与下面配置的单位都是秒)
    leaseExpirationDurationInSeconds: 30 #Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒

spring:
  application:
    name: client-user #此实例注册到eureka服务端的name
````

客户端启动类
````java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class UserApplication {

    public static void main(String[] args) {
        SpringApplication.run(UserApplication.class);
    }

}
````

在配置客户端的依赖的时候需加上spring-boot-starter-web依赖，否则会提示
**2019-04-24 10:42:07.346  WARN 18064 --- [      Thread-19] .s.c.a.CommonAnnotationBeanPostProcessor : Invocation of destroy method failed on bean with name 'scopedTarget.eurekaClient': org.springframework.beans.factory.BeanCreationNotAllowedException: Error creating bean with name 'eurekaInstanceConfigBean': Singleton bean creation not allowed while singletons of this factory are in destruction (Do not request a bean from a BeanFactory in a destroy method implementation!)**

以上是eureka单点配置。

## 配置集群
eureka服务端配置
````yaml
eureka:
  server:
    enable-self-preservation: true #关闭自我保护机制
    eviction-interval-timer-in-ms: 4000 #设置清理间隔(单位:毫秒 默认是60*1000)
  instance:
      hostname: eureka3000.com
  client:
    serviceUrl:
      defaultZone: http://eureka3001.com:3001/eureka,http://eureka3002.com:3002/eureka

eureka:
  server:
    enable-self-preservation: true #关闭自我保护机制
    eviction-interval-timer-in-ms: 4000 #设置清理间隔(单位:毫秒 默认是60*1000)
  instance:
      hostname: eureka3001.com
  client:
    serviceUrl:
      defaultZone: http://eureka3000.com:3000/eureka,http://eureka3002.com:3002/eureka
````

客户端配置
````yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka3000.com:3000/eureka,http://eureka3001.com:3001/eureka,http://eureka3002.com:3002/eureka #eureka服务端提供的注册地址 参考服务端配置的这个路径
````
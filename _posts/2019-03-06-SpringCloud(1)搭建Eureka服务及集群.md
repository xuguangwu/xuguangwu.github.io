---
title: SpringCloud(1)搭建Eureka服务及集群
categories:
 - Java
tags: 
 - Java
---

我将使用maven来构建整个工程。一步一步搭建spring-cloud全家桶。
完成该系列操作的项目地址：
https://github.com/xuguangwu/spring-cloud-learning.git

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

## zk与eureka的区别
4.1 Zookeeper保证CP
当向注册中心查询服务列表时，我们可以容忍注册中心返回的是几分钟以前的注册信息，
但不能接受服务直接down掉不可用。也就是说，服务注册功能对可用性的要求要高于一致性。
但是zk会出现这样一种情况，当master节点因为网络故障与其他节点失去联系时，
剩余节点会重新进行leader选举。问题在于，选举leader的时间太长，30 ~ 120s, 
且选举期间整个zk集群都是不可用的，这就导致在选举期间注册服务瘫痪。
在云部署的环境下，因网络问题使得zk集群失去master节点是较大概率会发生的事，
虽然服务能够最终恢复，但是漫长的选举时间导致的注册长期不可用是不能容忍的。

4.2 Eureka保证AP
Eureka看明白了这一点，因此在设计时就优先保证可用性。Eureka各个节点都是平等的，
几个节点挂掉不会影响正常节点的工作，剩余的节点依然可以提供注册和查询服务。
而Eureka的客户端在向某个Eureka注册或时如果发现连接失败，则会自动切换至其它节点，
只要有一台Eureka还在，就能保证注册服务可用(保证可用性)，只不过查到的信息可能不是最新的(不保证强一致性)。
除此之外，Eureka还有一种自我保护机制，如果在15分钟内超过85%的节点都没有正常的心跳，
那么Eureka就认为客户端与注册中心出现了网络故障，此时会出现以下几种情况： 
1. Eureka不再从注册列表中移除因为长时间没收到心跳而应该过期的服务 
2. Eureka仍然能够接受新服务的注册和查询请求，但是不会被同步到其它节点上(即保证当前节点依然可用) 
3. 当网络稳定时，当前实例新的注册信息会被同步到其它节点中

因此， Eureka可以很好的应对因网络故障导致部分节点失去联系的情况，而不会像zookeeper那样使整个注册服务瘫痪。

5. 总结
Eureka作为单纯的服务注册中心来说要比zookeeper更加“专业”，因为注册服务更重要的是可用性，
我们可以接受短期内达不到一致性的状况。不过Eureka目前1.X版本的实现是基于servlet的java web应用，
它的极限性能肯定会受到影响。期待正在开发之中的2.X版本能够从servlet中独立出来成为单独可部署执行的服务。
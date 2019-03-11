---
title: idea开发springboot热更新
categories:
 - springboot
tags: 
 - springboot
---

## idea自带的springboot代码热更新

On Update action : 顾名思义，当代码改变的时候，需要IDEA为你做什么；

On Frame deactivation : 当失去焦点（比如你最小化了IDEA窗口），需要IDEA为你做什么。
![idea_springboot_hotdeploy](https://github.com/xuguangwu/blog/blob/master/_posts/images/idea_springboot_hotdeploy.png?raw=true)


--------------------分割线----------------

## spring-boot-devtools方式
devtools 由于是双类加载机制，
启用相应的RestartClassLoader来加载class， 而jetcache会使用默认的jdk AppClassLoader，
最终导致从缓存中反序列化后得到的对象和项目中的对象class不同，最终产生ClassCastException.

网上有解决方案如下,但是我本人测试没用。
````
在 src/main/resources 中创建 META-INF 目录，在此目录下添加 spring-devtools.properties 配置，内容如下：

　　restart.include.mapper=/mapper-[\\w-\\.]+jar
　　restart.include.pagehelper=/pagehelper-[\\w-\\.]+jar
````

## 配置Idea
````
command + shift + a,查找到registry,勾选compiler.automake.allow.when.app.running
command + ,   ->   build  -> compile -> 勾选build project automatically
````

## 添加devtools
````
 <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <optional>true</optional>
    <scope>true</scope>
</dependency>
````

## 添加plugin
````
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <fork>true</fork>
    </configuration>
</plugin>
````

















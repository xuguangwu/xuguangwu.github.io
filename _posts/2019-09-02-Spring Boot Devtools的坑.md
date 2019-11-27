---
title: Spring Boot Devtools的坑
categories:
 - Java
---

今天遇到一个非常奇怪的问题，写了一个工具类实现ApplicationContextAware接口来获取Spring上下文，
代码如下：
````java
@Component
public class ApplicationContextUtils implements ApplicationContextAware {


    private static ApplicationContext APPLICATION_CONTEXT = null;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        APPLICATION_CONTEXT = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return APPLICATION_CONTEXT;
    }
}
````
项目启动的时候，APPLICATION_CONTEXT是初始化了的，但是当定时任务调用的时候，获取到的就为null。
后来仔细排查，将Spring Boot Devtools依赖去了，就正常了。这里我的分布式定时任务调用框架是elastic-job，springboot版本为2.0.3。

查了下Spring Boot Devtools的热部署原理：
深层原理是使用了两个ClassLoader，一个Classloader加载那些不会改变的类（第三方Jar包），
另一个ClassLoader加载会更改的类，称为restart ClassLoader,这样在有代码更改的时候，
原来的restart ClassLoader被丢弃，重新创建一个restart ClassLoader，由于需要加载的类相比较少，所以实现了较快的重启时间。








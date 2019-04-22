---
title: dubbo分布式日志追踪
categories:
 - Java
tags: 
 - dubbo
---

分布式应用都会有服务调用链路的问题，需要记录日志来定位问题，
如果是方法或者接口层面进行参数传递。对代码的耦合度太高。
我们可以通过dubbo的filter 结合slf4j的MDC或者log4j2的ThreadContext的进行参数的注入，
可以直接在日志文件中配置被注入的参数，这样就对系统和日志id打印进行了解耦。
其中当用logback日志的时候是需要调用MDC的方法，而log4j2则需要调用ThreadContext的方法。

消费端实现如下：
````java
import org.apache.dubbo.common.Constants;
import org.apache.dubbo.common.extension.Activate;
import org.apache.dubbo.common.utils.StringUtils;
import org.apache.dubbo.rpc.*;
import org.apache.log4j.MDC;

import java.util.UUID;


/**
 * 下游从RpcContext中获取参数
 * String traceId = RpcContext.getContext().getAttachment("trace_id");
 *
 * @author xuguangwu
 */
@Activate(group = Constants.CONSUMER)
public class ConsumerRpcTraceFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        String traceId = (String) MDC.get("traceId");
        if (StringUtils.isBlank(traceId)) {
            traceId = this.getUUID();
        }

        RpcContext.getContext().setAttachment("trace_id", traceId);
        return invoker.invoke(invocation);
    }

    /**
     * 获取UUID
     *
     * @return String UUID
     */
    public String getUUID() {
        String uuid = UUID.randomUUID().toString();
        return uuid.replaceAll("-", "");
    }
}
````

服务提供端实现：
````java
import org.apache.dubbo.common.Constants;
import org.apache.dubbo.common.extension.Activate;
import org.apache.dubbo.common.utils.StringUtils;
import org.apache.dubbo.rpc.*;
import org.apache.log4j.MDC;

import java.util.UUID;

@Activate(group = {Constants.PROVIDER}, order = 1)
public class ProviderRpcTraceFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        String traceId = RpcContext.getContext().getAttachment("trace_id");
        if (StringUtils.isBlank(traceId)) {
            traceId = this.getUUID();
        }
        MDC.put("traceId", traceId);
        RpcContext.getContext().setAttachment("trace_id", traceId);
        try {
            return invoker.invoke(invocation);
        } finally {
            //在方法调用完成后移除该ID
            MDC.remove("traceId");
        }
    }

    public String getUUID() {
        String uuid = UUID.randomUUID().toString();
        return uuid.replaceAll("-", "");
    }

}
````

在/src/main/resources/META-INF/dubbo/com.apache.dubbo.rpc.Filter文件中配置filter

最后在日志配置中加上%X{traceId} 即可。

以上为单线程情况，多线程则需将子线程id也传入到MDC中。建一个最简单的spring项目。
* spring配置文件，声明一个线程池
````xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xmlns="http://www.springframework.org/schema/beans" xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.3.xsd
       http://dubbo.apache.org/schema/dubbo http://dubbo.apache.org/schema/dubbo/dubbo.xsd http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">

    <context:component-scan base-package="org.apache.dubbo.demo.consumer"/>

    <bean id="threadPool"
          class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
        <!-- 核心线程数 -->
        <property name="corePoolSize" value="3"/>
        <!-- 最大线程数 -->
        <property name="maxPoolSize" value="10"/>
        <!-- 队列最大长度 >=mainExecutor.maxSize -->
        <property name="queueCapacity" value="25"/>
        <!-- 线程池维护线程所允许的空闲时间 -->
        <property name="keepAliveSeconds" value="300"/>
        <!-- 线程池对拒绝任务(无线程可用)的处理策略 ThreadPoolExecutor.CallerRunsPolicy策略 ,调用者的线程会执行该任务,如果执行器已关闭,则丢弃.  -->
        <property name="rejectedExecutionHandler">
            <bean class="java.util.concurrent.ThreadPoolExecutor$CallerRunsPolicy"/>
        </property>
    </bean>

</beans>
````
* 声明线程上下文，并交由声名的线程池来管理。
````java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;
import java.util.ArrayList;
import java.util.List;

@Component
public class TreadContext {

    @Autowired
    private ThreadPoolTaskExecutor executor;

    private static TreadContext context;

    private List<AbstractRunnable> threads = new ArrayList();

    @PostConstruct
    private void init() {
        context = this;
    }

    /**
     * 关闭线程池
     */
    public void close() {
        executor.shutdown();
    }

    /**
     * 执行所有线程池
     */
    public void open() {
        for (Runnable thread : this.threads) {
            executor.execute(thread);
        }
    }

    /**
     * 添加多个线程任务
     *
     * @param threads
     * @return
     */
    public TreadContext source(List<AbstractRunnable> threads) {
        this.threads.addAll(threads);
        return context;
    }

    /**
     * 添加单个线程任务
     *
     * @param thread
     */
    public void source(AbstractRunnable thread) {
        this.threads.add(thread);
    }

    /**
     * 获取线程池管理对象
     *
     * @return
     */
    public static TreadContext getContext() {
        return context;
    }
}
````
* 上下文中内容定制
````java
import org.slf4j.MDC;

public abstract class AbstractRunnable implements Runnable {
    private String traceId;
    private String ip;

    public AbstractRunnable(String traceId, String ip) {
        this.traceId = traceId;
        this.ip = ip;
    }

    private void log() {
        MDC.put(Constant.TRANC_ID, traceId);
        MDC.put(Constant.LOCAL_IP, ip);
    }

    protected abstract void thread();

    @Override
    public void run() {
        log();
        thread();
    }

    public String getTraceId() {
        return traceId;
    }

    public void setTraceId(String traceId) {
        this.traceId = traceId;
    }

    public String getIp() {
        return ip;
    }

    public void setIp(String ip) {
        this.ip = ip;
    }
}
````
* 涉及到的常量
````java
public class Constant {
    public static final String TRANC_ID = "TRANC_ID";
    public static final String LOCAL_IP = "LOCAL_IP";
}
````

* 测试
````java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.slf4j.MDC;
import org.springframework.context.support.AbstractApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.ArrayList;
import java.util.List;

public class Main {

    private static Logger logger = LoggerFactory.getLogger(Main.class);

    private static String ID = "10000000000001";
    private static String IP = "192.168.80.123";

    public static void main(String[] args) {
        AbstractApplicationContext appContext = new ClassPathXmlApplicationContext("spring.xml");
        TreadContext context = appContext.getBean(TreadContext.class);
        MDC.put(Constant.TRANC_ID, ID);
        MDC.put(Constant.LOCAL_IP, "192.168.80.123");
        AbstractRunnable thread1 = new MyThread(ID, IP);
        AbstractRunnable thread2 = new MyThread(ID, IP);
        List<AbstractRunnable> threads = new ArrayList<AbstractRunnable>();
        threads.add(thread1);
        threads.add(thread2);
        context.source(threads).open();
        logger.info("thread name:{}  ,trace_id:{}  ,local_ip:{}", Thread.currentThread().getName(), MDC.get(Constant.TRANC_ID), MDC.get(Constant.LOCAL_IP));
        appContext.registerShutdownHook();
    }
}
````




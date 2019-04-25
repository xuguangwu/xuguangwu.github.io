---
title: SpringCloud(3)网关
categories:
 - Java
tags: 
 - Java
---

1. 引入依赖
    ````xml
    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        </dependency>
    
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
    </dependencies>
    ````

2. @EnableZuulProxy
3. 配置zuul的yml文件
    ````yaml
    server:
      port: 9000
    eureka:
      client:
        serviceUrl:
          defaultZone: http://eureka3000.com:3000/eureka  #eureka服务端提供的注册地址 参考服务端配置的这个路径
      instance:
        instance-id: zuul #此实例注册到eureka服务端的唯一的实例ID
        prefer-ip-address: true #是否显示IP地址
        leaseRenewalIntervalInSeconds: 10 #eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒 (与下面配置的单位都是秒)
        leaseExpirationDurationInSeconds: 30 #Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒
    
    spring:
      application:
        name: zuul #此实例注册到eureka服务端的name
    zuul:
    #  prefix: /api
      ignored-services: "*"
      routes:
        power:
          serviceId: server-user
          path: /**
    ````
4. 自定义Filter
    ````java
    @Component
    public class MyFallbackProvider implements FallbackProvider {
    
        @Override
        public String getRoute() {
            return "*";
        }
    
        @Override
        public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
            if (cause instanceof HystrixTimeoutException) {
                return response(HttpStatus.GATEWAY_TIMEOUT);
            } else {
                return response(HttpStatus.INTERNAL_SERVER_ERROR);
            }
        }
    
        private ClientHttpResponse response(final HttpStatus status) {
            return new ClientHttpResponse() {
                @Override
                public HttpStatus getStatusCode() { //返回一个HttpStatus对象 这个对象是个枚举对象， 里面包含了一个status code 和 reasonPhrase信息
                    return status;
                }
    
                //返回status的code 比如 404，500等
                @Override
                public int getRawStatusCode() {
                    return status.value();
                }
    
                //返回一个HttpStatus对象的reasonPhrase信息
                @Override
                public String getStatusText() {
                    return status.getReasonPhrase();
                }
    
                @Override
                public void close() {
                }
    
                //返回降级信息
                @Override
                public InputStream getBody() {
                    return new ByteArrayInputStream("降级信息".getBytes());
                }
    
                //需要对响应报头设置的话可以在此设置
                @Override
                public HttpHeaders getHeaders() {
                    HttpHeaders headers = new HttpHeaders();
                    headers.setContentType(MediaType.APPLICATION_JSON);
                    return headers;
                }
            };
        }
    }
    ````
5. 服务降级
    ````java
    @Component
    public class MyFallbackProvider implements FallbackProvider {
    
        @Override
        public String getRoute() {
            return "*";
        }
    
        @Override
        public ClientHttpResponse fallbackResponse(String route, Throwable cause) {
            if (cause instanceof HystrixTimeoutException) {
                return response(HttpStatus.GATEWAY_TIMEOUT);
            } else {
                return response(HttpStatus.INTERNAL_SERVER_ERROR);
            }
        }
    
        private ClientHttpResponse response(final HttpStatus status) {
            return new ClientHttpResponse() {
                @Override
                public HttpStatus getStatusCode() { //返回一个HttpStatus对象 这个对象是个枚举对象， 里面包含了一个status code 和 reasonPhrase信息
                    return status;
                }
    
                //返回status的code 比如 404，500等
                @Override
                public int getRawStatusCode() {
                    return status.value();
                }
    
                //返回一个HttpStatus对象的reasonPhrase信息
                @Override
                public String getStatusText() {
                    return status.getReasonPhrase();
                }
    
                @Override
                public void close() {
                }
    
                //返回降级信息
                @Override
                public InputStream getBody() {
                    return new ByteArrayInputStream("降级信息".getBytes());
                }
    
                //需要对响应报头设置的话可以在此设置
                @Override
                public HttpHeaders getHeaders() {
                    HttpHeaders headers = new HttpHeaders();
                    headers.setContentType(MediaType.APPLICATION_JSON);
                    return headers;
                }
            };
        }
    }
    ````
6. 网关的高可用
yaml配置如下：
    ````yaml
    #客户端
     server:
       port: 9000
     eureka:
       client:
         serviceUrl:
           defaultZone: http://eureka3000.com:3000/eureka  #eureka服务端提供的注册地址 参考服务端配置的这个路径
       instance:
         instance-id: zuul #此实例注册到eureka服务端的唯一的实例ID
         prefer-ip-address: true #是否显示IP地址
         leaseRenewalIntervalInSeconds: 10 #eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒 (与下面配置的单位都是秒)
         leaseExpirationDurationInSeconds: 30 #Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒
     
     spring:
       application:
         name: zuul #此实例注册到eureka服务端的name
     zuul:
       prefix: /api
       ignored-services: "*"
       routes:
         power:
           serviceId: zuul-server
           path: /zuul/**
 
    ----
    # 服务端1
    server:
      port: 9001
    eureka:
      client:
        serviceUrl:
          defaultZone: http://eureka3000.com:3000/eureka  #eureka服务端提供的注册地址 参考服务端配置的这个路径
      instance:
        instance-id: zuul-server-1 #此实例注册到eureka服务端的唯一的实例ID
        prefer-ip-address: true #是否显示IP地址
        leaseRenewalIntervalInSeconds: 10 #eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒 (与下面配置的单位都是秒)
        leaseExpirationDurationInSeconds: 30 #Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒
    
    spring:
      application:
        name: zuul-server #此实例注册到eureka服务端的name
    zuul:
    #  prefix: /api
      ignored-services: "*"
      routes:
        power:
          serviceId: server-power
          path: /power/**
    
    ----
    # 服务端2
    server:
      port: 9002
    eureka:
      client:
        serviceUrl:
          defaultZone: http://eureka3000.com:3000/eureka  #eureka服务端提供的注册地址 参考服务端配置的这个路径
      instance:
        instance-id: zuul-server-2 #此实例注册到eureka服务端的唯一的实例ID
        prefer-ip-address: true #是否显示IP地址
        leaseRenewalIntervalInSeconds: 10 #eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒 (与下面配置的单位都是秒)
        leaseExpirationDurationInSeconds: 30 #Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒
    
    spring:
      application:
        name: zuul-server #此实例注册到eureka服务端的name
    zuul:
      #  prefix: /api
      ignored-services: "*"
      routes:
        power:
          serviceId: server-power
          path: /power/**
    ````



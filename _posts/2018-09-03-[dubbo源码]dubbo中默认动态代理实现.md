---
title: dubbo默认动态代理实现
categories:
 - Java
tags: 
 - dubbo
---

这里先记录下dubbo通过javassist动态字节码技术来生成代理类的源码。
````java
public Object getPropertyValue(Object o, String n) {
      org.apache.dubbo.demo.DemoService w;
      try {
          w = ((org.apache.dubbo.demo.DemoService) $1);
      } catch (Throwable e) {
          throw new IllegalArgumentException(e);
      }
      throw new org.apache.dubbo.common.bytecode.NoSuchPropertyException("Not found property \"" + $2 + "\" field or setter method in class org.apache.dubbo.demo.DemoService.");
  }

  public void setPropertyValue(Object o, String n, Object v) {
      org.apache.dubbo.demo.DemoService w;
      try {
          w = ((org.apache.dubbo.demo.DemoService) $1);
      } catch (Throwable e) {
          throw new IllegalArgumentException(e);
      }
      throw new org.apache.dubbo.common.bytecode.NoSuchPropertyException("Not found property \"" + $2 + "\" field or setter method in class org.apache.dubbo.demo.DemoService.");
  }

  public Object invokeMethod(Object o, String n, Class[] p, Object[] v) throws java.lang.reflect.InvocationTargetException {
      org.apache.dubbo.demo.DemoService w;
      try {
          //o对应的就是接口实现类，$0指的是this，这里需要注意下
          w = ((org.apache.dubbo.demo.DemoService) $1);
      } catch (Throwable e) {
          throw new IllegalArgumentException(e);
      }
      try {
          //$2即传入的参数n，这里可以推断出就是方法名，$3即方法参数类型，$4即方法的参数
          if ("sayHello".equals($2) && $3.length == 1) {
              return ($w) w.sayHello((java.lang.String) $4[0]);
          }
      } catch (Throwable e) {
          throw new java.lang.reflect.InvocationTargetException(e);
      }
      throw new org.apache.dubbo.common.bytecode.NoSuchMethodException("Not found method \"" + $2 + "\" in class org.apache.dubbo.demo.DemoService.");
  }
````


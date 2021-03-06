---
title: Spring中BeanFactory和FactoryBean的区别
categories:
 - Spring
tags: 
    - Java
    - Spring
---

首先说下面试中常问到的一个问题BeanFactory和FactoryBean的区别。
乍一看名字类似，其实完全不一样，
FactoryBean，本身是一个接口，需要我们的业务bean去实现这个接口。
实现该接口的Bean和其他的Bean有什么区别呢。
看下面一段代码，体会下
````java
@Component("testBean")
public class TestBean implements FactoryBean {

    public void test() {
        System.out.println("testBean");
    }

    @Override
    public Object getObject() throws Exception {
        return new TestDaoImpl();
    }

    @Override
    public Class<?> getObjectType() {
        return TestDaoImpl.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }
}
````
解释一下，实现了FactoryBean接口的bean在spring容器中会生成两个bean，
一个bean是getObject()方法返回的bean，该bean的名称对应的是@Component("testBean")
注解中声名的testBean，而TestBean本身在容器中的名字叫&testBean，可以通过以下方法来测试

````java
BeanFactory beanFactory = new AnnotationConfigApplicationContext(Configure.class);

TestBean testBean = (TestBean) beanFactory.getBean("&testBean");
TestDaoImpl testDaoImpl = (TestDaoImpl) beanFactory.getBean("testBean");
testBean.test();
testDaoImpl.test();
````

仔细一点可以看到这里我从容器中去获取bean的时候用的是BeanFactory对象去接口
new AnnotationConfigApplicationContext(Configure.class);中配置的bean，
从这里就可以看出BeanFactory其实就是spring的bean容器。ok，这两个概念就讲清楚了。

另外介绍下FactoryBean在代码中的应用，非常明显的一个示例就是mybatis为我们提供的SqlSessionFactoryBean，
它就实现了FactoryBean接口，而它返回的对象是SqlSessionFactory，在SqlSessionFactory中通过buildSqlSessionFactory()
方法为bean配置了所有与mybatis相关的配置。

#TODO 具体好处，目前我只能体会，但是还没想好用什么语句或者示例来表达

最后补全我的测试代码
````java
@Configuration
@ComponentScan(basePackages = {"com.clear.spring.beans"})
public class Configure {

}
````

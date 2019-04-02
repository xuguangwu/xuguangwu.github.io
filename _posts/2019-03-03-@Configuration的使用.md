---
title: Configuration的使用
categories:
 - Spring
tags: 
 - java
 - Spring
---

在springboot中我们经常使用@Configuration来配置项目中的配置类的bean。
那么它到底有什么用呢。追根溯源，还是回到spring的源码中来看。
首先演示下不加@Configuration的场景，其实可以发现bean依然可以注入进来，没有任何问题。

上代码：
````java
public class Application {

	public static void main(String[] args) {
		AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(Configure.class);
		TestDaoImpl testDao = annotationConfigApplicationContext.getBean(TestDaoImpl.class);
		testDao.test();
	}
}

@ComponentScan(value = "com.clear")
public class Configure {

	@Bean
	public TestDaoImpl testDaoImpl() {
		return new TestDaoImpl();
	}
}

public class TestDaoImpl {
	public void test() {
        System.out.println("test");
    }
}
````
可以看到我在TestDaoImpl中没有添加@Component/@Service/@Repository样的声明，
只是在Configure类中以@Bean并new一个TestDaoImpl
方式注入到spring容器中，使用没有问题，那么为什么要加@Configuration注解呢，是用来解决什么样的问题呢。

核心实现在ConfigurationClassPostProcessor中，我贴出较为关键的代码分析下

````java
public void enhanceConfigurationClasses(ConfigurableListableBeanFactory beanFactory) {
    Map<String, AbstractBeanDefinition> configBeanDefs = new LinkedHashMap<>();
    for (String beanName : beanFactory.getBeanDefinitionNames()) {
        BeanDefinition beanDef = beanFactory.getBeanDefinition(beanName);
        if (ConfigurationClassUtils.isFullConfigurationClass(beanDef)) {
            if (!(beanDef instanceof AbstractBeanDefinition)) {
                throw new BeanDefinitionStoreException("Cannot enhance @Configuration bean definition '" +
                        beanName + "' since it is not stored in an AbstractBeanDefinition subclass");
            }
            else if (logger.isWarnEnabled() && beanFactory.containsSingleton(beanName)) {
                logger.warn("Cannot enhance @Configuration bean definition '" + beanName +
                        "' since its singleton instance has been created too early. The typical cause " +
                        "is a non-static @Bean method with a BeanDefinitionRegistryPostProcessor " +
                        "return type: Consider declaring such methods as 'static'.");
            }
            configBeanDefs.put(beanName, (AbstractBeanDefinition) beanDef);
        }
    }
    if (configBeanDefs.isEmpty()) {
        // nothing to enhance -> return immediately
        return;
    }

    ConfigurationClassEnhancer enhancer = new ConfigurationClassEnhancer();
    for (Map.Entry<String, AbstractBeanDefinition> entry : configBeanDefs.entrySet()) {
        AbstractBeanDefinition beanDef = entry.getValue();
        // If a @Configuration class gets proxied, always proxy the target class
        beanDef.setAttribute(AutoProxyUtils.PRESERVE_TARGET_CLASS_ATTRIBUTE, Boolean.TRUE);
        try {
            // Set enhanced subclass of the user-specified bean class
            Class<?> configClass = beanDef.resolveBeanClass(this.beanClassLoader);
            if (configClass != null) {
                //完成对全注解类的cglib代理
                Class<?> enhancedClass = enhancer.enhance(configClass, this.beanClassLoader);
                if (configClass != enhancedClass) {
                    if (logger.isDebugEnabled()) {
                        logger.debug(String.format("Replacing bean definition '%s' existing class '%s' with " +
                                "enhanced class '%s'", entry.getKey(), configClass.getName(), enhancedClass.getName()));
                    }
                    beanDef.setBeanClass(enhancedClass);
                }
            }
        }
        catch (Throwable ex) {
            throw new IllegalStateException("Cannot load configuration class: " + beanDef.getBeanClassName(), ex);
        }
    }
}
````
当类添加了@Configuration注解后会由cglib对其实现动态代理，那么来看下newEnhancer方法实现。
````java
private Enhancer newEnhancer(Class<?> configSuperClass, @Nullable ClassLoader classLoader) {
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(configSuperClass);
    //增强接口，主要是需要通过该接口获取到beanFactory
    enhancer.setInterfaces(new Class<?>[] {EnhancedConfiguration.class});
    enhancer.setUseFactory(false);
    enhancer.setNamingPolicy(SpringNamingPolicy.INSTANCE);
    enhancer.setStrategy(new BeanFactoryAwareGeneratorStrategy(classLoader));
    enhancer.setCallbackFilter(CALLBACK_FILTER);
    enhancer.setCallbackTypes(CALLBACK_FILTER.getCallbackTypes());
    return enhancer;
}
````
既然是代理那么就需要重点关注它的回调方法，这里提供的回调方法是一个数组
````java
private static final Callback[] CALLBACKS = new Callback[] {
    //核心逻辑
    new BeanMethodInterceptor(),
    new BeanFactoryAwareMethodInterceptor(),
    NoOp.INSTANCE
};
````
那么继续跟进到BeanMethodInterceptor中去看intercept方法，主要看这段逻辑
````java
if (isCurrentlyInvokedFactoryMethod(beanMethod)) {
    return cglibMethodProxy.invokeSuper(enhancedConfigInstance, beanMethodArgs);
}

return resolveBeanReference(beanMethod, beanMethodArgs, beanFactory, beanName);
````
isCurrentlyInvokedFactoryMethod方法的实现，
````java
private boolean isCurrentlyInvokedFactoryMethod(Method method) {
    Method currentlyInvoked = SimpleInstantiationStrategy.getCurrentlyInvokedFactoryMethod();
    return (currentlyInvoked != null && method.getName().equals(currentlyInvoked.getName()) &&
            Arrays.equals(method.getParameterTypes(), currentlyInvoked.getParameterTypes()));
}
````
这里就是判断@Configuration配置的bean中如果当@Bean中需要引用到另外一个bean的时候是创建一个新的bean还是引用之前创建的bean问题，
也就是为了遵守单例原则，做的一次判断。这里的判断是第二次来创建bean的方法是否与第一次来创建bean的方法一样，
显然，第二次来创建bean的方法是代理方法与第一次不一样，则第二次来创建的bean的时候直接从beanFactory中获取即可。






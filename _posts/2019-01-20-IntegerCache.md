---
title: IntegerCache
categories:
 - Java
tags: 
 - Java
---

Integer中有个静态内部类IntegerCache，里面有个cache[],也就是Integer常量池，常量池的大小为一个字节（-128~127）。

````java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer cache[];

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        cache = new Integer[(high - low) + 1];
        int j = low;
        for(int k = 0; k < cache.length; k++)
            cache[k] = new Integer(j++);

        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
````
例如：Integer a = 10;
调用的是Integer.valueOf()方法，其实现如下：
````java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
````
完成了自动装箱操作。

本地测试代码如下：
````java
public class IntegerCacheTest {

    public static void main(String[] args) {

        //从缓存池中读取
        Integer a1 = 1;
        Integer b1 = 1;
        System.out.println(a1 == b1);

        //超过缓存区间
        Integer a2 = 155;
        Integer b2 = 155;
        System.out.println(a2 == b2);


        //没有走缓存
        Integer a3 = new Integer(-1);
        Integer b3 = new Integer(-1);
        System.out.println(a3 == b3);

        //自动拆箱
        System.out.println("a3 == -1:" + (a3 == -1));

    }
}
````



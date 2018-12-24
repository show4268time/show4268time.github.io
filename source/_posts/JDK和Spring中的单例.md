---
title: JDK和Spring中的单例
date: 2018-07-07 23:57:04
tags:
- Design pattern
- Spring
---
## JDK中的单例模式 ##

光说不练假把式，上一遍博文讲了单例的实现方法，这一篇就讲讲哪里用到单例模式了，分为JDK和Spring，因为这两个是我们平时写代码一种用到的。我们先从JDK开始。

1. **java.lang.Runtime**

在JAVA进程中，时间必须都是统一的，及Runtime的实例应该只有一个。所以应该使用单例来实现。  
话不多说，先贴代码
```java 
public class Runtime {
    private static Runtime currentRuntime = new Runtime();

    public static Runtime getRuntime() {
        return currentRuntime;
    }

    /** Don't let anyone else instantiate this class */
    private Runtime() {}
}
```

以上时JDK中的部分代码，从上面的代码我们可以得知使用的是饿汉单例模式，在Runtime这个类被加载的时候这个类就被创建出来了。

Runtime类封装了Java运行时的环境。每一个java程序实际上都是启动了一个JVM进程，那么每个JVM进程都是对应这一个Runtime实例，此实例是由JVM为其实例化的。每个 Java 应用程序都有一个 Runtime 类实例，使应用程序能够与其运行的环境相连接。

---

## Spring中的单例模式 ##
Spring依赖注入Bean实例默认是单例的，但是Spring中使用的单例模式不是我们上一篇中介绍的那几种模式，而是一种特殊的单例模式--**登记模式**，为什么说他特殊呢？因为登记模式不是一种真正意义上的单例模式，单例模式的单例作用域是整个java环境也就是JVM，但是登记模式不是，Spring中Bean的作用域是应用程序的上下文也就是  **ApplicationContext**，换句话说也就是可以有多个Spring容器，每一个容器内存在唯一Bean实例。
好了，就让我们看看这个注册模式

```java
public class RegisterSingleton {

    //构建采用ConcurrentHashMap,用于充当缓存注册表
    private final static Map<String, Object> singletonObjects = new
    ConcurrentHashMap<>();

    // 静态代码块只加载执行一次
    static {
        // 实例化Bean
        RegisterSingleton registerSingleton = new 
        RegisterSingleton();
        //并注册到注册表中,key为类的完全限定名,value为实例化对象
        singletonObjects.put(RegisterSingleton.getClass().getName(),
        registerSingleton);
    }

    /**
     * 私有化构造方法,避免外部创建本类实例
     */
    private RegisterSingleton() {}


    /**
     *  对外暴露获得该bean的方法,Spring框架一般会返回Object
     * @return
     */
    public static  RegisterSingleton getInstance(String className) {
        if (StringUtils.isEmpty(className)) {
            return null;
        }
        //从注册表获取,如果没有直接创建
        if (singletonObjects.get(className) == null) {
            try {
                //如果为空,通过反射进行实例化
                singletonObjects.put(className, 
                Class.forName(className).newInstance());
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        //从缓存表中回去,如果缓存命中直接返回
        return (RegisterSingleton)singletonObjects.get(className);
    }
    
}
```
---
Spring中就是采用了这种特殊的注册模式来实现单例的模式的。  
```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {

    // 通过 Map 实现单例注册表
    private final Map<String, Object> singletonObjects = new        ConcurrentHashMap<String, Object>(64);

    public Object getSingleton(String beanName, ObjectFactory<?> singletonFactory) {
        Assert.notNull(beanName, "'beanName' must not be null");
        synchronized (this.singletonObjects) {
            // 检查缓存中是否存在实例  
            Object singletonObject = this.singletonObjects.get(beanName);
            if (singletonObject == null) {
                // ...忽略代码
                try {
                    singletonObject = singletonFactory.getObject();
                }
                catch (BeanCreationException ex) {
                    // ...忽略代码
                }
                finally {
                    // ...忽略代码
                }
                // 如果实例对象在不存在，我们注册到单例注册表中。
                addSingleton(beanName, singletonObject);
            }
            return (singletonObject != NULL_OBJECT ? singletonObject : null);
        }
    }

    protected void addSingleton(String beanName, Object singletonObject) {
        synchronized (this.singletonObjects) {
            this.singletonObjects.put(beanName, (singletonObject != null ? singletonObject : NULL_OBJECT));

        }
    }
}
```  

Spring对Bean实例的创建是采用单例注册表的方式进行实现的，而这个注册表的缓存是 ConcurrentHashMap对象。
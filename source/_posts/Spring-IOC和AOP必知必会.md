---
title: Spring IOC和AOP必知必会
date: 2019-04-11 22:03:10
tags:
---

- Spring-IOC
- Spring-AOP
- JDK动态代理
- BeanPostProcessor实现自定义注解

## Spring-IOC

**所谓IoC**，对于spring框架来说，就是由spring来负责控制对象的生命周期和对象间的关系，而不是传统的手动去控制对象的创建。

**从配置上来说**，相当于把代码中的依赖关系转移到xml配置文件中。

**优点**:通过使用IOC，再加上Spring提倡的面向接口编程，能很大程度实现组件之间的解耦，利于功能复用，最终提高了项目整体的灵活性和可维护性。

**DI(依赖注入**)其实就是IOC的另外一种说法，IoC容器动态的向某个对象提供它所需要的其他对象。

## Spring-AOP

>AOP：Aspect Orient Programming，面向切面编程；用来处理具有横切性质的系统级服务，如事务管理、安全检查、缓存、对象池管理等。

使用场景：
- Authentication 权限
- Caching 缓存
- Context passing 内容传递
- Error handling 错误处理
- Lazy loading　懒加载
- Debugging　调试
- logging, tracing, profiling and monitoring　 记录跟踪　优化　校准
- Performance optimization　性能优化
- Persistence　持久化
- Resource pooling　资源池
- Synchronization　同步
- Transactions  事务

AOP分类：
- 静态AOP实现：AspectJ，编译阶段修改
- 动态AOP实现：Spring AOP,底层使用JDK动态代理或cglib

基本概念：
- Aspect -切面
- Joinpoint -连接点
- Advice -增强处理
- Pointcut -切入点
<!--more-->
>Spring AOP不需要在编译时对目标类进行增强，而是在运行时生成目标类的代理类，该代理类要么与目标类实现相同的接口（JDK动态代理处理策略），要么是目标类的子类（cglib处理策略）。**处理策略可以自动切换，也可以强制指定。**

xml配置：
```xml
<aop:aspectj-autoproxy/>

<context:component-scan base-package="com.fish">
    <context:include-filter type="annotation" expression="org.aspectj.lang.annotation.Aspect"/>
</context:component-scan>
```

定义切面和切入点：
```java
@Aspect
public class AuthAspect {

    @Before("execution(* com.fish.handler.*.*(..))")
    public void authority(){
        System.out.println("执行权限检查...");
    }
}
```

Around增强处理通常需要在线程安全的环境下使用。

```java
@Aspect
public class AuthAspect {

    @Around("execution(* com.fish.handler.*.*(..))")
    public void authority(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        // 此处可修改方法入参
        Object[] args = proceedingJoinPoint.getArgs();
        // 此处可修改方法返回值
        Object rvt = proceedingJoinPoint.proceed(args);
        long end = System.currentTimeMillis();

        // 方法名
        String methodName = proceedingJoinPoint.getSignature().getName();
        // 全限定类名
        String methodDeclaringTypeName = proceedingJoinPoint.getSignature().getDeclaringTypeName();
        System.out.println(methodDeclaringTypeName + "." + methodName
                + "()方法执行所用的时间为：" + (end - start)/1000 + "s");
    }
}
```

## JDK动态代理

- Proxy
- InvocationHandler

```java
public class MyInvokationHandler implements InvocationHandler {

    private Object target;
    public MyInvokationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("进行增强处理...");
        // 使用反射以target作为调用者执行方法
        Object result = method.invoke(target,args);
        return result;
    }
}
```

```java
public class Main {

    public static void main(String[] args) {
        HelloIFace iface = new HelloImpl();
        MyInvokationHandler handler = new MyInvokationHandler(iface);

        // 创建代理对象
        HelloIFace proxy = (HelloIFace)Proxy.newProxyInstance(
            iface.getClass().getClassLoader(),
            iface.getClass().getInterfaces(),handler);
        proxy.sayHello();
    }
}
```

## BeanPostProcessor实现自定义注解

- Bean后处理器（BeanPostProcessor）：是IOC的扩展，对Bean进一步增强处理
- 容器后处理器：如属性占位符配置器、重写占位符配置器

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogTest {
}
```

```java
@Component
public class HelloHandler {

    @LogTest
    public void sayHello(){
        System.out.println("Hello!");
    }
}
```

此示例使用cjlib创建代理，也可采用Proxy的方式。

```java
@Component
public class LogAnnotationBeanPostProcessor implements BeanPostProcessor {

    // 初始化之前
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        return o;
    }

    // 初始化之后
    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        Method[] methods = ReflectionUtils.getAllDeclaredMethods(o.getClass());
        if (methods != null) {
            for (Method method : methods) {
                LogTest logTest = AnnotationUtils.findAnnotation(method, LogTest.class);
                if (logTest != null) {
                    Enhancer enhancer = new Enhancer();
                    // 设置父类为被代理的类
                    enhancer.setSuperclass(o.getClass());
                    enhancer.setCallback(new MethodInterceptor() {
                        @Override
                        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
                            System.out.println("Before:"+method);
                            Object object=methodProxy.invokeSuper(o, objects);
                            System.out.println("After:"+method);
                            return object;
                        }
                    });
                    return enhancer.create();
                }
            }
        }
        return o;
    }
}
```

测试：

```java
public static void main(String[] args) {
    ApplicationContext apx = new ClassPathXmlApplicationContext("apx.xml");
    HelloHandler helloHandler = apx.getBean(HelloHandler.class);
    helloHandler.sayHello();
}
```

配置文件：

```xml
<context:component-scan base-package="com.fish"/>
<aop:aspectj-autoproxy/>
```
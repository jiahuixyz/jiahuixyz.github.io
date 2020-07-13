---
title: 使用SpringAOP实现自定义注解之切入点表达式
date: 2020-01-11 20:13:40
tags:
---


使用Spring AOP实现自定义注解时，关键在于切入点PointCut表达式的书写，即通过表达式扫描指定的注解。

以下给出两种写法,这两种写法都可以扫描指定包下的注解。

1、

```java
/**
 * 作用：捕获方法异常，并修改异常返回结果
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExceptionMessageHandler {

}
```

```java
@Aspect
@Component
public class ExceptionAnnotationAspect {
 
 private static final Logger LOG =Logger.getLogger(ExceptionAnnotationAspect.class);

 @Around("execution(@com.fish.webcore.annotation.ExceptionMessageHandler * com.fish.webcore..*.*(..))")
 public Object exceptionHandler(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
  try {
   return proceedingJoinPoint.proceed();
  } catch (Exception e) {
   LOG.error("服务器内部错误!", e);
   return JacksonUtil.getExceptionResult(e, "服务器内部错误!");
  }
 }

}
```

2、

```java
@Around("within(com.fish..*) && @annotation(lrt)")
public void authority(ProceedingJoinPoint proceedingJoinPoint, LogRunTime lrt) throws Throwable
```

Spring.xml配置：
```xml
    <aop:aspectj-autoproxy/>
    <context:component-scan base-package="com.fish"/>
```

参考资料：<br>
[https://blog.csdn.net/qq_36951116/article/details/79172485](https://blog.csdn.net/qq_36951116/article/details/79172485)
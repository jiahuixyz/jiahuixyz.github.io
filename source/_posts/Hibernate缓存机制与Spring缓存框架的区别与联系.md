---
title: Hibernate缓存机制与Spring缓存框架的区别与联系
date: 2019-04-11 16:35:36
tags:
---

- Hibernate一级缓存
- Hibernate二级缓存
- Spring缓存机制
- Hibernate缓存与Spring缓存的比较

## Hibernate一级缓存

一级缓存是Session级别的缓存(即当前Session有效)，默认启用。

一级缓存的key为ID(主键)。

当执行load/get/list/iterator/save/update/saveOrUpdate等方法时会把得到的实体对象放入一级缓存中（只支持实体对象缓存，不支持属性的缓存），之后在同一个session范围内执行load/get/iterator等查询语句（不包括list）查询缓存过的数据时，Hibernate不会去数据库中查询。

当程序保存或修改实体时，不会立即同步到数据库，而是保存在一级缓存中，只有程序显示调用Session的flush()或close()方法才会刷新至数据库，这样可以减少数据库的交互，提高数据库的访问性能。

一级缓存清除方法：
- contains(object obj)
- evict(Object obj)
- clear()
<!--more-->
## Hibernate二级缓存

二级缓存是SessionFactory级别的，是所有Session共享的，全局性的，需要手动开启；开启后查询时，Session会先查找一级缓存，再查找二级缓存，最后再查找数据库。

二级缓存的key为ID(主键)，value为实体对象。一般load（），iterate（）使用到二级缓存，list()需要结合查询缓存使用。

二级缓存需要指定缓存实现，如EhCache，并使用@Cache注解指定哪些实体类启用二级缓存，并指定缓存策略(一般为READ_ONLY)。

Hibernate缓存策略(隔离级别)：
- 只读(READ_ONLY)
- 读/写(READ_WRITE)
- 非严格读/写(NONSTRICT_READ_WRITE)
- 事务(TRANSACTIONAL)

## Spring缓存机制

Spring缓存需要指定缓存实现(如EhCache)。

相关注解：
- @Cacheable
- @CacheEvict

对于一个支持缓存的方法，Spring会在其被调用后将其返回值缓存起来，以保证下次利用同样的参数来执行该方法时可以直接从缓存中获取结果，而不需要再次执行该方法。

缓存的key默认是通过KeyGenerator生成的，其默认生成策略如下：
- 如果方法没有参数，则使用0作为key。
- 如果只有一个参数的话则使用该参数作为key。
- 如果参数多于一个的话则使用所有参数的hashCode作为key。

实际使用中需要自定义key的生成策略，如
```
@Cacheable(key = "#root.target" + "_" + "#root.method" + "_" + "#p0" + "_" + "#p1")
```

参考资料：<br>
[https://www.jianshu.com/p/2a584aaafad3](https://www.jianshu.com/p/2a584aaafad3)<br>
[https://blog.csdn.net/u013378306/article/details/52168628](https://blog.csdn.net/u013378306/article/details/52168628)


## Hibernate二级缓存与Spring缓存的比较

与Hibernate SessionFactory级别的二级缓存相比，Spring缓存的级别更高，Spring缓存可以在控制器组件或业务逻辑组件级别进行缓存，这样应用完全无需重复调用底层的DAO组件（如基于Hibernate实现）的方法。
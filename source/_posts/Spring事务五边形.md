---
title: Spring事务五边形
date: 2020-03-13 21:29:20
tags:
---

**ACID：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）、持久性（Durability）**

### 事务五边形

- 传播行为
- 隔离级别
- 是否只读
- 事务超时时间
- 回滚规则

```
// Spring事务核心接口
public interface TransactionDefinition {
    int getPropagationBehavior(); // 返回事务的传播行为
    int getIsolationLevel(); // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getTimeout();  // 返回事务必须在多少秒内完成
    boolean isReadOnly(); // 事务是否只读，事务管理器能够根据这个返回值进行优化，确保事务是只读的
} 
```

<!--more-->

### 传播行为

```java
//用法示例
@Transactional(propagation=Propagation.REQUIRED)
```

事务行为       | 说明
:- |:-
REQUIRED	   | 支持当前事务，假设当前没有事务。就新建一个事务
SUPPORTS	   | 支持当前事务，假设当前没有事务，就以非事务方式运行
MANDATORY	   | 支持当前事务，假设当前没有事务，就抛出异常
REQUIRES_NEW   | 新建事务，假设当前存在事务。把当前事务挂起
NOT_SUPPORTED  | 以非事务方式运行操作。假设当前存在事务，就把当前事务挂起
NEVER	       | 以非事务方式运行，假设当前存在事务，则抛出异常
NESTED	       | 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作

默认REQUIRED。

参考资料：https://blog.csdn.net/qq_38526573/article/details/87898161

### 数据库事务隔离级别

事务隔离级别|脏读|不可重复读|幻读
:-|:-|:-|:-
读未提交（read-uncommitted）    |是	|是	|是
不可重复读（read-committed）	|否	|是	|是
可重复读（repeatable-read） 	|否	|否	|是
串行化（serializable）	        |否	|否	|否

事务的并发问题

1. 脏读：事务A读取了事务B更新的数据，然后B回滚操作，那么A读取到的数据是脏数据
2. 不可重复读：事务 A 多次读取同一数据，事务 B 在事务A多次读取的过程中，对数据作了更新并提交，导致事务A多次读取同一数据时，结果 不一致。
3. 幻读：系统管理员A将数据库中所有学生的成绩从具体分数改为ABCDE等级，但是系统管理员B就在这个时候插入了一条具体分数的记录，当系统管理员A改结束后发现还有一条记录没有改过来，就好像发生了幻觉一样，这就叫幻读。
> 注：不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表

MySQL默认:REPEATABLE-READ(可重复读)
```
# MySQL查看隔离级别
select @@tx_isolation;
```
Oracle默认:READ COMMITTED(读已提交)

参考资料：[https://blog.csdn.net/trigl/article/details/50968079#t21](https://blog.csdn.net/trigl/article/details/50968079#t21)

### Spring事务的隔离级别

```java
//用法示例
@Transactional(isolation = Isolation.READ_UNCOMMITTED)
```

Spring事务隔离级别比数据库事务隔离级别多一个default，这也是PlatfromTransactionManager默认的隔离级别，表示使用数据库默认的事务隔离级别，Spring另外四个隔离级别同数据库的隔离级别。

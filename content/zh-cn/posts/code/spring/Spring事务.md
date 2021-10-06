---
title: "Spring事务"
date: 2021-08-27T11:15:10+08:00
draft: false
reward: true
categories: [
"编程思想"
]
tags : [
"Spring"
]
series : [
"Manual"
]
images : [
"images/center.png"
]
---

[comment]: <> "# Spring事务"

Spring事务可简单的理解为由`PlatformTransactionManager`,  `TransactionDefinition`,  `TransactionStatus`  构成

### PlatformTransactionManager

其中 `org.springframework.transaction.PlatformTransactionManager` 接口定义如下：

```java
public interface PlatformTransactionManager extends TransactionManager {
    // Return a currently active transaction or create a new one, according to the specified propagation behavior（根据指定的传播行为，返回当前活动的事务或创建一个新事务。）
    TransactionStatus getTransaction(TransactionDefinition definition) throws TransactionException; 
    // Commit the given transaction, with regard to its status（使用事务目前的状态提交事务）
    Void commit(TransactionStatus status) throws TransactionException;  
    // Perform a rollback of the given transaction（对执行的事务进行回滚）
    Void rollback(TransactionStatus status) throws TransactionException;  
}
```

Spring并不直接管理事务，它只提供接口抽象，事务管理器的具体实现由持久化框架自己实现，例如：`DataSourceTransactionManager` (JDBC / MyBatis),

 `JpaTransactionManager`,  `HibernateTransactionManager`,  `JtaTransactionManager`,  `KafkaTransactionManager`,  `RedissonTransactionManager`

### TransactionDefinition

Spring定义了事务管理的五大属性：**隔离级别**、**传播行为**、**是否只读**、**事务超时**、**回滚规则**

除`回滚规则`之外，其他属性都可以通过TransactionDefinition接口获得。

其中`org.springframework.transaction.TransactionDefinition` 接口定义如下：

```java
public interface TransactionDefinition {
    // 返回事务的传播行为
    int getPropagationBehavior(); 
    // 返回事务的隔离级别，事务管理器根据它来控制另外一个事务可以看到本事务内的哪些数据
    int getIsolationLevel(); 
    //返回事务的名字
    String getName()；
    // 返回事务必须在多少秒内完成
    int getTimeout();  
    // 返回是否优化为只读事务。
    boolean isReadOnly();
} 
```

### TransactionStatus

只有hasSavepoint方法不是继承的。

其中 `org.springframework.transaction.TransactionStatus`  接口定义如下：

```java
public interface TransactionStatus extends TransactionExecution, SavepointManager, Flushable {
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
}
```



### 参考

[可能是最漂亮的Spring事务管理详解](https://juejin.cn/post/6844903608224333838)

[spring 事务的传播机制看这篇就够了](https://segmentfault.com/a/1190000020386113)

[Spring 事务管理](https://www.w3cschool.cn/wkspring/by1r1ha2.html)


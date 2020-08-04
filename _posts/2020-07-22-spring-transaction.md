---
layout: post
title: "Spring事务解析"
subtitle: ""
date: 2020-07-22 18:04:29 +0800
author: "YangYongKang"
header-style: text
catalog: true
archive:
    - java
    - spring
    - 事务
tags:
    - java
    - spring
    - 事务
---

### 什么是事务
指作为单个逻辑工作单元执行的一系列操作，要么完全地执行，要么完全地不执行。 简单的说，事务就是并发控制的单位，是用户定义的一个操作序列。事务的概念可以描述为具有以下四个关键属性说成是 ACID：
### 事务的特性

- A：原子性（Atomicity） 事务中的操作要么都不做，要么就全做。
- C：一致性（Consistency） 事务执行的结果必须是从数据库从一个一致性状态转换到另一个一致性状态。
- I：隔离性（Isolation） 一个事务的执行不能被其他事务干扰
- D：持久性（Durability） 一个事务一旦提交，它对数据库中数据的改变就应该是永久性的
### 事务的分类
#### 1. 编程式事务
所谓编程式事务指的是通过编码方式实现事务，允许用户在代码中精确定义事务的边界。即类似于JDBC编程实现事务管理。管理使用TransactionTemplate或者直接使用底层的PlatformTransactionManager。对于编程式事务管理，spring推荐使用TransactionTemplate。
#### 2. 声明式事务
管理建立在AOP之上的。其本质是对方法前后进行拦截，然后在目标方法开始之前创建或者加入一个事务，在执行完目标方法之后根据执行情况提交或者回滚事务。声明式事务最大的优点就是不需要通过编程的方式管理事务，这样就不需要在业务逻辑代码中掺杂事务管理的代码，只需在配置文件中做相关的事务规则声明(或通过基于@Transactional注解的方式)，便可以将事务规则应用到业务逻辑中。
### Spring中事务管理的核心接口
![1120165-20171003112557490-1092711802.png](/img/1593584022094-085a07b9-4b7b-4dec-96ef-15c94645896f.png)
#### 1. PlatformTransactionManager  事务管理器
Spring事务管理器的接口是org.springframework.transaction.PlatformTransactionManager，如上图所示，Spring并不直接管理事务，通过这个接口，Spring为各个平台如JDBC、Hibernate等都提供了对应的事务管理器，也就是将事务管理的职责委托给Hibernate或者JTA等持久化机制所提供的相关平台框架的事务来实现。
具体的方法

- TransactionStatus getTransaction（TransactionDefinition definition）：用于获取事务状态信息。
- void commit（TransactionStatus status）：用于提交事务。
- void rollback（TransactionStatus status）：用于回滚事务。
#### 2. TransactionDefinition  事务定义
TransactionDefinition 接口是事务定义（描述）的对象，它提供了事务相关信息获取的方法，其中包括五个操作，具体如下。

- String getName()：获取事务对象名称。
- int getIsolationLevel()：获取事务的隔离级别。
- int getPropagationBehavior()：获取事务的传播行为。
- int getTimeout()：获取事务的超时时间。
- boolean isReadOnly()：获取事务是否只读。
#### 3. TransactionStatus 事务状态
该接口用来记录事务的状态 该接口定义了一组方法,用来获取或判断事务的相应状态信息.
PlatformTransactionManager.getTransaction(…) 方法返回一个 TransactionStatus 对象。返回的TransactionStatus 对象可能代表一个新的或已经存在的事务（如果在当前调用堆栈有一个符合条件的事务）。
**TransactionStatus接口接口内容如下：**
```java
public interface TransactionStatus{
    boolean isNewTransaction(); // 是否是新的事物
    boolean hasSavepoint(); // 是否有恢复点
    void setRollbackOnly();  // 设置为只回滚
    boolean isRollbackOnly(); // 是否为只回滚
    boolean isCompleted; // 是否已完成
}
```
### 事务并发出现的问题

- **脏读（Dirty read）:** 当一个事务正在访问数据并且对数据进行了修改，而这种修改还没有提交到数据库中，这时另外一个事务也访问了这个数据，然后使用了这个数据。因为这个数据是还没有提交的数据，那么另外一个事务读到的这个数据是“脏数据”，依据“脏数据”所做的操作可能是不正确的。
- **丢失修改（Lost to modify）:** 指在一个事务读取一个数据时，另外一个事务也访问了该数据，那么在第一个事务中修改了这个数据后，第二个事务也修改了这个数据。这样第一个事务内的修改结果就被丢失，因此称为丢失修改。
例如：事务1读取某表中的数据A=20，事务2也读取A=20，事务1修改A=A-1，事务2也修改A=A-1，最终结果A=19，事务1的修改被丢失。
- **不可重复读（Unrepeatableread）:** 指在一个事务内多次读同一数据。在这个事务还没有结束时，另一个事务也访问该数据。那么，在第一个事务中的两次读数据之间，由于第二个事务的修改导致第一个事务两次读取的数据可能不太一样。这就发生了在一个事务内两次读到的数据是不一样的情况，因此称为不可重复读。
- **幻读（Phantom read）:** 幻读与不可重复读类似。它发生在一个事务（T1）读取了几行数据，接着另一个并发事务（T2）插入了一些数据时。在随后的查询中，第一个事务（T1）就会发现多了一些原本不存在的记录，就好像发生了幻觉一样，所以称为幻读。
- **不可重复度和幻读区别：**

     不可重复读的重点是修改，幻读的重点在于新增或者删除。
### 事务的传播行为
* PROPAGATION_REQUIRED  支持当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择，也是Spring默认的事务的传播。 
* PROPAGATION_SUPPORTS  支持当前事务，如果当前没有事务，就以非事务方式执行。 
* PROPAGATION_MANDATORY 支持当前事务，如果当前没有事务，就抛出异常。
* PROPAGATION_REQUIRES_NEW  新建事务，如果当前存在事务，把当前事务挂起。 
* PROPAGATION_NOT_SUPPORTED 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 
* PROPAGATION_NEVER 以非事务方式执行，如果当前存在事务，则抛出异常。 
* PROPAGATION_NESTED 如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则进行与PROPAGATION_REQUIRED类似的操作。 

### 事务的隔离级别
* ISOLATION_DEFAULT 这是一个PlatfromTransactionManager默认的隔离级别，使用数据库默认的事务隔离级别。另外四个与JDBC的隔离级别相对应 
* ISOLATION_READ_UNCOMMITTED 这是事务最低的隔离级别，它充许另外一个事务可以看到这个事务未提交的数据。这种隔离级别会产生脏读，不可重复读和幻读。 
* ISOLATION_READ_COMMITTED 保证一个事务修改的数据提交后才能被另外一个事务读取。另外一个事务不能读取该事务未提交的数据。 
* ISOLATION_REPEATABLE_READ 这种事务隔离级别可以防止脏读，不可重复读。但是可能出现幻读。 
* ISOLATION_SERIALIZABLE 这是花费最高代价但是最可靠的事务隔离级别。事务被处理为顺序执行。除了防止脏读，不可重复读外，还避免了幻读。 

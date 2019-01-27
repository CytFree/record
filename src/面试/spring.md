### 注解
><context:annotation-config>:注解扫描是针对已经在Spring容器里注册过的Bean

><context:component-scan>:不仅具备<context:annotation-config>
的所有功能，还可以在指定的package下面扫描对应的bean

### 事物
>四大特性

 + 原子性
 + 一致性
 + 隔离性
 + 持久性


#### spring事物接口
>事物管理器：PlatformTransactionManager

+ getTransaction(TransactionDefinition definition)
+ commit(TransactionStatus status)
+ rollback(TransactionStatus status)
>事物定义信息；TransactionDefinition

    定义事物的隔离性、传播性、超时时间、是否只读等等
###### 事务的隔离级别从低到高（spring默认隔离级别为数据库的隔离级别）
isolation
+ Read Uncommitted：

        最低的隔离级别，什么都不需要做，
        一个事务可以读到另一个事务未提交的结果。
        所有的并发事务问题都会发生。
+ Read Committed：oracle默认

        只有在事务提交后，其更新结果才会被其他事务看见。
        可以解决脏读问题。
+ Repeated Read：mysql默认

        在一个事务中，对于同一份数据的读取结果总是相同的，
        无论是否有其他事务对这份数据进行操作，
        以及这个事务是否提交。可以解决脏读、不可重复读。
+ Serialization：

        事务串行化执行，隔离级别最高，牺牲了系统的并发性。
        可以解决并发事务的所有问题。

###### 事物为了解决以下问题：
 +  脏读(Drity Read)：

        事务A修改了一个数据，但未提交，
        事务B读到了事务A未提交的更新结果，
        如果事务A提交失败，事务B读到的就是脏数据。
        Read Committed可以解决脏读问题，
        但仍存在以下两种问题。
+ 不可重复读(Non-repeatable read) :

        在同一个事务中，对于同一份数据读取到的结果不一致。
        比如，事务B在事务A提交前读到的结果，和提交后读到
        的结果可能不同。不可重复读出现的原因就是事务
        并发修改记录，要避免这种情况，最简单的方法就是
        对要修改的记录加锁，这导致锁竞争加剧，影响性能。
        （另一种方法是通过MVCC可以在无锁的情况下，
        避免不可重复读。待了解。。）Repeated Read可以解决不可重复读问题和脏读问题
        ，但仍无法解决下面的问题。

+ 幻读(Phantom Read) :

        在同一个事务中，同一个查询多次返回的结果不一致。事务A新增了一条记录，
        事务B在事务A提交前后各执行了一次查询操作，发现后一次比前一次多了一条记录。
        幻读仅指由于并发事务增加记录导致的问题，这个不能像不可重复读通过记录加锁解决，
        因为对于新增的记录根本无法加锁。需要将事务串行化，才能避免幻读。
        Serialization解决了以上所有问题，但是性能效率较低。
　　　通常来说，事务隔离级别越低，所需持有锁的时间也就越短，并发性能也就越好。

###### 事物的传播性
1. 什么是事务传播行为？

        事务传播行为用来描述由某一个事务传播行为修饰的方法被嵌套进
        另一个方法的时事务如何传播。
+ PROPAGATION_REQUIRED

        如果当前没有事务，则新建一个事务。如果已经存在事务，则加入当前事务。
+ PROPAGATION_SUPPORTS

        如果当前没有事务，则以无事务方式执行。如果已经存在事务，则加入当前事务。
+ PROPAGATION_MANDATORY

        如果当前没有事务，则抛异常。支持当前事务。

+ PROPAGATION_REQUIRES_NEW

        如果当前没有事务，则新建事务。如果有，则当前事务挂起

+ PROPAGATION_NOT_SUPPORTED

        以非事务方式执行。如果存在事务，当前事务挂起

+ PROPAGATION_NEVER

        以非事务方式执行。如果存在事务，抛异常

+ PROPAGATION_NESTED

        如果存在事务，则嵌套事务内执行。如果没有，则新建一个事务


>事物运行状态：TransactionStatus

##### spring事务配置

1、编程式事务管理

    注入TransactionTemplate类

2、声明式事务管理
+ TransactionProxyFactoryBean方式
    ```
     <bean id="" class="TransactionProxyFactoryBean">
            <property name="target" ref="切面类"/>
            <property name="transactionManager" ref="事务管理器"/>
            <property name="transactionAttributes">
                 <props>
                     <prop key="方法">传播级别,隔离级别,readOnly,-回滚异常,+不回滚的异常</prop>
                 </props>
            </property>
     </bean>
    ```
- 基于AspectJ的xml方式
    ```
        1、定义aop增强 advice
        <tx:advice id="" transaction-manager="">
           <tx:attrbutrs>
                <tx:method name="方法" 传播级别,隔离级别,readOnly/>
           </tx:attrbutrs>
        </tx:advice>
        2、定义aop 配置 config
            pointcut:切点
            advisor：一个切面  aspect：多个切点的切面和advise
            <aop:config>
                <aop:pointcut expression="" id="">
                <aop:advisor advice-ref="" pointcut-ref=""/>
            </aop:config>
    ```
+ 基于注解的方式
    ```
        <tx:annatation-driver transaction-manager=""/>
        注解：@Transactional
    ```
- springboot开启注解的方式
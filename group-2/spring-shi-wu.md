# Spring 事务

Spring 事务支持两种使用方式，分别是：

* **编程式事务（代码方式）**
* **声明式事务（注解方式）**

## **Spring 事务中的隔离级别**

在 TransactionDefinition 接口中定义了五个表示隔离级别的常量：

* <mark style="color:blue;">**ISOLATION\_DEFAULT**</mark>：使用后端数据库默认的隔离级别，Mysql 默认采用的 REPEATABLE\_READ 隔离级别；Oracle 默认采用的 READ\_COMMITTED 隔离级别。
* <mark style="color:blue;">**ISOLATION\_READ\_UNCOMMITTED**</mark>：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。
* <mark style="color:blue;">**ISOLATION\_READ\_COMMITTED**</mark>：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生。
* <mark style="color:blue;">**ISOLATION\_REPEATABLE\_READ**</mark>：对同一字段的多次读取结果都是一致的，除非数据是被当前事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。
* <mark style="color:blue;">**ISOLATION\_SERIALIZABLE**</mark>：所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## **Spring 事务中有哪几种事务传播行为？**

在 TransactionDefinition 接口中定义了 7 个表示事务传播行为的常量：

*   <mark style="color:blue;">**PROPAGATION\_REQUIRED**</mark>：

    * 如果当前存在事务，则加入该事务；
    * **如果当前没有事务，则创建一个新的事务。**

    > 这是最常见的选择，也是 Spring 默认的事务的传播。
* <mark style="color:blue;">**PROPAGATION\_SUPPORTS**</mark>：&#x20;
  * 如果当前存在事务，则加入该事务；
  * **如果当前没有事务，则以非事务的方式继续运行。**
* <mark style="color:blue;">**PROPAGATION\_MANDATORY**</mark>：&#x20;
  * 如果当前存在事务，则加入该事务；
  * **如果当前没有事务，则抛出异常。**
*   <mark style="color:blue;">**PROPAGATION\_REQUIRES\_NEW**</mark>：

    * **创建一个新的事务；**
    * 如果当前存在事务，则把当前事务挂起。

    > <mark style="color:orange;">**新建的事务将和被挂起的事务没有任何关系，是两个独立的事务，外层事务失败回滚之后，不能回滚内层事务执行的结果，内层事务失败抛出异常，外层事务捕获，也可以不处理回滚操作**</mark>
* <mark style="color:blue;">**PROPAGATION\_NOT\_SUPPORTED**</mark>：&#x20;
  * **以非事务方式运行；**
  * 如果当前存在事务，则把当前事务挂起。
* <mark style="color:blue;">**PROPAGATION\_NEVER**</mark>：
  * &#x20;以非事务方式运行；
  * **如果当前存在事务，则抛出异常。**
*   <mark style="color:blue;">**PROPAGATION\_NESTED**</mark>：

    * 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；
    * 如果当前没有事务，则按照 PROPAGATION\_REQUIRED 执行。

    > 它使用了一个单独的事务，这个事务拥有多个可以回滚的保存点。
    >
    > 内部事务的回滚不会对外部事务造成影响。
    >
    > 它只对 DataSourceTransactionManager 事务管理器起效。

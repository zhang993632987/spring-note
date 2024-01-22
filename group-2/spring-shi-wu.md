# Spring 事务

## **Spring事务管理的方式有几种？**

1.编程式事务：在代码中硬编码（不推荐使用）。

2.声明式事务：在配置文件中配置（推荐使用），分为基于XML的声明式事务和基于注解的声明式事务。

## **Spring事务中的隔离级别有哪几种？**

在TransactionDefinition接口中定义了五个表示隔离级别的常量：

ISOLATION\_DEFAULT：使用后端数据库默认的隔离级别，Mysql默认采用的REPEATABLE\_READ隔离级别；Oracle默认采用的READ\_COMMITTED隔离级别。

ISOLATION\_READ\_UNCOMMITTED：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

ISOLATION\_READ\_COMMITTED：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

ISOLATION\_REPEATABLE\_READ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

ISOLATION\_SERIALIZABLE：最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。

## **Spring事务中有哪几种事务传播行为？**

在TransactionDefinition接口中定义了7个表示事务传播行为的常量。

支持当前事务的情况：

PROPAGATION\_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

PROPAGATION\_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

PROPAGATION\_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）。

不支持当前事务的情况：

PROPAGATION\_REQUIRES\_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。

PROPAGATION\_NOT\_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。

PROPAGATION\_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。

其他情况：

PROPAGATION\_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于PROPAGATION\_REQUIRED。

***

著作权归@pdai所有 原文链接：https://pdai.tech/md/interview/x-interview-2.html

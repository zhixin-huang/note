# Spring事务

## Spring事务的隔离级别
DEFAULT：默认隔离级别，由DBA设置来决定默认隔离级别。  
READ_UNCOMMITTED：允许读取尚未提交的更改。会出现脏读、不可重复读、幻读。  
READ_COMMITTED：允许从已提交的并发事务中读取数据。可以避免脏读，会出现不可重复读和幻读。  
REPEATABLE_READ：对相同字段的多次读取的结果是一致的，除非数据被当前事务本身改变。可以避免脏读、不可重复读，会出现幻读。  
SERIALIZABLE：串行化执行，完全服从ACID的隔离级别。通过完全锁定当前事务所涉及的数据表来完成操作。  

脏读：A事务读取到B事务未提交的数据value，此时B事务回滚，A事务读取的数据value就有问题。  
不可重复读：A事务读取数据后还未提交数据，此时B事务修改该数据并完成提交，此时A事务再次读取的时候，发现数据发生了改变。
幻读：A事务查询范围内的数据还未提交数据，此时B事务新增了一条该范围内的数据并已提交，此时A事务再次读取次范围内的数据，发现数据多了一条。


## Spring事务的传播行为
PROPAGATION_REQUIRED：如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。  
PROPAGATION_SUPPORTS：如果存在一个事务，则支持当前事务。如果当前没有事务，则以非事务方式执行。  
PROPAGATION_MANDATORY：如果存在一个事务，则支持当前事务。如果当前没有事务，则抛出异常。  
PROPAGATION_REQUIRES_NEW：总是开启一个新事务。如果当前事务存在，则将当前事务挂起。  
PROPAGATION_NOT_SUPPORTS：总是非事务的执行，并挂起任何存在的事务。  
PROPAGATION_NEVER：总是非事务的执行，如果存在一个活动事务，则抛出异常。  
PROPAGATION_NESTED：如果存在一个活动事务，则运行一个事务嵌套在活动事务中。如果当前没有存在一个活动事务，则按照PROPAGATION_REQUIRED方式。  

PROPAGATION_REQUIRED：spring默认的传播行为。

## 开发中注意的点
事务不生效？  
1、mysql的选择，InnoDB才支持事务，MyISAM不支持事务操作。  
2、类没有被spring管理，例如没有加@Service等注解。  
3、方法不是public。  
4、自身调用的问题。本身调内部方法，没有经过spring代理类。  
5、异常被吃了。  
6、抛出的异常类型错误，例如回滚异常为RuntimeException，抛出的是Exception。  

4和6是经常忽视的，在写事务的时候需要注意。
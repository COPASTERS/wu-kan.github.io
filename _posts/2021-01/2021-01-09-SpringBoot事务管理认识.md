---
title: SpringBoot事务管理认识
categories: SpringBoot 事务管理
tags:  SpringBoot 事务管理
date: 2021-01-09 11:31:00
---

## 背景

springboot的事务使用总感觉有点理解不透彻，查了点资料加上自己的理解

## 一：事务认识

大家所了解的事务Transaction，它是一些列严密操作动作，要么都操作完成，要么都回滚撤销。Spring事务管理基于底层数据库本身的事务处理机制。数据库事务的基础，是掌握Spring事务管理的基础。这篇总结下Spring事务。

事务具备ACID四种特性，ACID是Atomic（原子性）、Consistency（一致性）、Isolation（隔离性）和Durability（持久性）的英文缩写。　

（1）原子性（Atomicity）

事务最基本的操作单元，要么全部成功，要么全部失败，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚到事务开始前的状态，就像这个事务从来没有执行过一样。

（2）一致性（Consistency）

事务的一致性指的是在一个事务执行之前和执行之后数据库都必须处于一致性状态。如果事务成功地完成，那么系统中所有变化将正确地应用，系统处于有效状态。如果在事务中出现错误，那么系统中的所有变化将自动地回滚，系统返回到原始状态。

（3）隔离性（Isolation）

指的是在并发环境中，当不同的事务同时操纵相同的数据时，每个事务都有各自的完整数据空间。由并发事务所做的修改必须与任何其他并发事务所做的修改隔离。事务查看数据更新时，数据所处的状态要么是另一事务修改它之前的状态，要么是另一事务修改它之后的状态，事务不会查看到中间状态的数据。
　　
（4）持久性（Durability）

指的是只要事务成功结束，它对数据库所做的更新就必须永久保存下来。即使发生系统崩溃，重新启动数据库系统后，数据库还能恢复到事务成功结束时的状态。 

## 二：事务的传播机制

1.什么是事务：

　　事务是程序中一系列严密的操作，所有操作执行必须成功完成，否则在每个操作所做的更改将会被撤销，这也是事务的原子性（要么成功，要么失败）。

　　数据库向用户提供保存当前程序状态的方法，叫事务提交（commit）；当事务执行过程中，使数据库忽略当前的状态并回到前面保存的状态的方法叫事务回滚（rollback）

2.事务的传播机制

　　以spring的事务传播机制为例子：

　　Spring事务机制主要包括声明式事务和编程式事务，此处侧重讲解声明式事务，编程式事务在实际开发中得不到广泛使用，仅供学习参考。

　　Spring声明式事务让我们从复杂的事务处理中得到解脱。使得我们再也无需要去处理获得连接、关闭连接、事务提交和回滚等这些操作。再也无需要我们在与事务相关的方法中处理大量的try…catch…finally代码。我们在使用Spring声明式事务时，有一个非常重要的概念就是事务属性。事务属性通常由事务的传播行为，事务的隔离级别，事务的超时值和事务只读标志组成。我们在进行事务划分时，需要进行事务定义，也就是配置事务的属性。

spring在TransactionDefinition接口中定义了七个事务传播行为：

- propagation_requierd：如果当前没有事务，就新建一个事务，如果已存在一个事务中，加入到这个事务中，这是最常见的选择。

- propagation_supports：支持当前事务，如果没有当前事务，就以非事务方法执行。

- propagation_mandatory：使用当前事务，如果没有当前事务，就抛出异常。

- propagation_required_new：新建事务，如果当前存在事务，把当前事务挂起。

- propagation_not_supported：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

- propagation_never：以非事务方式执行操作，如果当前事务存在则抛出异常。

- propagation_nested：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与propagation_required类似的操作

### （1）PROPAGATION_REQUIRED  如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务。

Java代码：

```java
//事务属性 PROPAGATION_REQUIRED 
methodA{ 
　　…… 
　　methodB(); 
　　…… 
}
 
//事务属性 PROPAGATION_REQUIRED 
methodB{ 
   …… 
}
```

使用spring声明式事务，spring使用AOP来支持声明式事务，会根据事务属性，自动在方法调用之前决定是否开启一个事务，并在方法执行之后决定事务提交或回滚事务。

单独调用methodB方法：

Java代码:
```java
main{
　　metodB(); 
}
```

相当于Java代码

```java
Main{ 
    Connection con=null; 
    try{ 
        con = getConnection(); 
        con.setAutoCommit(false); 
        //方法调用
        methodB(); 
        //提交事务
        con.commit(); 
    } 
    Catch(RuntimeException ex){ 
        //回滚事务
        con.rollback();   
    } 
    finally{ 
    //释放资源
    closeCon(); 
    } 
}
```

Spring保证在methodB方法中所有的调用都获得到一个相同的连接。在调用methodB时，没有一个存在的事务，所以获得一个新的连接，开启了一个新的事务。

单独调用MethodA时，在MethodA内又会调用MethodB.

执行效果相当于：

Java代码

```java
main{ 
     Connection con = null; 
    try{ 
         con = getConnection(); 
        methodA(); 
        con.commit(); 
    } 
    catch(RuntimeException ex){ 
        con.rollback(); 
    } 
    finally{ 
       closeCon(); 
    }  
}
```

调用MethodA时，环境中没有事务，所以开启一个新的事务.当在MethodA中调用MethodB时，环境中已经有了一个事务，所以methodB就加入当前事务。

### （2）PROPAGATION_SUPPORTS 如果存在一个事务，支持当前事务。如果没有事务，则非事务的执行。但是对于事务同步的事务管理器，PROPAGATION_SUPPORTS与不使用事务有少许不同。

Java代码：

```java
//事务属性 PROPAGATION_REQUIRED 
methodA(){ 
  methodB(); 
}
 
//事务属性 PROPAGATION_SUPPORTS 
methodB(){ 
  …… 
}
```

单纯的调用methodB时，methodB方法是非事务的执行的。当调用methdA时,methodB则加入了methodA的事务中,事务地执行。

### （3）PROPAGATION_MANDATORY 如果已经存在一个事务，支持当前事务。如果没有一个活动的事务，则抛出异常。

Java代码：

```java
//事务属性 PROPAGATION_REQUIRED 
methodA(){ 
    methodB(); 
}
 
//事务属性 PROPAGATION_MANDATORY 
methodB(){ 
    …… 
}
```

当单独调用methodB时，因为当前没有一个活动的事务，则会抛出异常throw new IllegalTransactionStateException(“Transaction propagation ‘mandatory’ but no existing transaction found”);当调用methodA时，methodB则加入到methodA的事务中，事务地执行。


### （4）PROPAGATION_REQUIRES_NEW 总是开启一个新的事务。如果一个事务已经存在，则将这个存在的事务挂起。

```java
//事务属性 PROPAGATION_REQUIRED 
methodA(){ 
   doSomeThingA(); 
   methodB(); 
   doSomeThingB(); 
}
 
//事务属性 PROPAGATION_REQUIRES_NEW 
methodB(){ 
   …… 
}
```

Java代码：

```java
main(){ 
  methodA(); 
}
```

相当于

Java代码：

```java
main(){ 
  TransactionManager tm = null; 
try{ 
  //获得一个JTA事务管理器 
    tm = getTransactionManager(); 
    tm.begin();//开启一个新的事务 
    Transaction ts1 = tm.getTransaction(); 
    doSomeThing(); 
    tm.suspend();//挂起当前事务 
    try{ 
      tm.begin();//重新开启第二个事务 
      Transaction ts2 = tm.getTransaction(); 
      methodB(); 
      ts2.commit();//提交第二个事务 
   } 
  Catch(RunTimeException ex){ 
      ts2.rollback();//回滚第二个事务 
  } 
  finally{ 
     //释放资源 
   } 
    //methodB执行完后，复恢第一个事务 
    tm.resume(ts1); 
doSomeThingB(); 
    ts1.commit();//提交第一个事务 
} 
catch(RunTimeException ex){ 
   ts1.rollback();//回滚第一个事务 
} 
finally{ 
   //释放资源 
} 
}
```

在这里，我把ts1称为外层事务，ts2称为内层事务。从上面的代码可以看出，ts2与ts1是两个独立的事务，互不相干。Ts2是否成功并不依赖于ts1。如果methodA方法在调用methodB方法后的doSomeThingB方法失败了，而methodB方法所做的结果依然被提交。而除了methodB之外的其它代码导致的结果却被回滚了。使用PROPAGATION_REQUIRES_NEW,需要使用JtaTransactionManager作为事务管理器。

### （5）PROPAGATION_NOT_SUPPORTED  总是非事务地执行，并挂起任何存在的事务。使用PROPAGATION_NOT_SUPPORTED,也需要使用JtaTransactionManager作为事务管理器。（代码示例同上，可同理推出）

### （6）PROPAGATION_NEVER 总是非事务地执行，如果存在一个活动事务，则抛出异常；

### （7）PROPAGATION_NESTED如果一个活动的事务存在，则运行在一个嵌套的事务中. 如果没有活动事务, 则按TransactionDefinition.PROPAGATION_REQUIRED 属性执行。这是一个嵌套事务,使用JDBC 3.0驱动时,仅仅支持DataSourceTransactionManager作为事务管理器。需要JDBC 驱动的java.sql.Savepoint类。有一些JTA的事务管理器实现可能也提供了同样的功能。使用PROPAGATION_NESTED，还需要把PlatformTransactionManager的nestedTransactionAllowed属性设为true;而nestedTransactionAllowed属性值默认为false;

Java代码：

```java
//事务属性 PROPAGATION_REQUIRED 
methodA(){ 
   doSomeThingA(); 
   methodB(); 
   doSomeThingB(); 
}
 
//事务属性 PROPAGATION_NESTED 
methodB(){ 
  …… 
}
```

如果单独调用methodB方法，则按REQUIRED属性执行。如果调用methodA方法，相当于下面的效果：

Java代码：

```java
main(){ 
Connection con = null; 
Savepoint savepoint = null; 
try{ 
   con = getConnection(); 
   con.setAutoCommit(false); 
   doSomeThingA(); 
   savepoint = con2.setSavepoint(); 
   try{ 
       methodB(); 
   }catch(RuntimeException ex){ 
      con.rollback(savepoint); 
   } 
   finally{ 
     //释放资源 
  }
 
   doSomeThingB(); 
   con.commit(); 
} 
catch(RuntimeException ex){ 
  con.rollback(); 
} 
finally{ 
   //释放资源 
} 
}
```

当methodB方法调用之前，调用setSavepoint方法，保存当前的状态到savepoint。如果methodB方法调用失败，则恢复到之前保存的状态。但是需要注意的是，这时的事务并没有进行提交，如果后续的代码(doSomeThingB()方法)调用失败，则回滚包括methodB方法的所有操作。


嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚。

Spring 默认的事务传播行为是 PROPAGATION_REQUIRED，它适合于绝大多数的情况。假设 ServiveX#methodX() 都工作在事务环境下（即都被 Spring 事务增强了），假设程序中存在如下的调用链：Service1#method1()->Service2#method2()->Service3#method3()，那么这 3 个服务类的 3 个方法通过 Spring 的事务传播机制都工作在同一个事务中。

## 三：数据库的4种隔离级别

数据库事务的隔离级别有4种，由低到高分别为Read uncommitted 、Read committed 、Repeatable read 、Serializable 。而且，在事务的并发操作中可能会出现脏读，不可重复读，幻读。下面通过事例一一阐述它们的概念与联系。

脏读、不可重复读、幻象读概念说明：　　

- 脏读：指当一个事务正在访问数据，并且对数据进行了修改，而这种数据还没有提交到数据库中，这时，另外一个事务也访问这个数据，然后使用了这个数据。因为这个数据还没有提交那么另外一个事务读取到的这个数据我们称之为脏数据。依据脏数据所做的操作肯能是不正确的。

- 不可重复读：指在一个事务内，多次读同一数据。在这个事务还没有执行结束，另外一个事务也访问该同一数据，那么在第一个事务中的两次读取数据之间，由于第二个事务的修改第一个事务两次读到的数据可能是不一样的，这样就发生了在一个事物内两次连续读到的数据是不一样的，这种情况被称为是不可重复读。

- 幻象读：一个事务先后读取一个范围的记录，但两次读取的纪录数不同，我们称之为幻象读（两次执行同一条 select 语句会出现不同的结果，第二次读会增加一数据行，并没有说这两次执行是在同一个事务中）

### 读未提交 （Read uncommitted）

读未提交，顾名思义，就是一个事务可以读取另一个未提交事务的数据。

事例：老板要给程序员发工资，程序员的工资是3.6万/月。但是发工资时老板不小心按错了数字，按成3.9万/月，该钱已经打到程序员的户口，但是事务还没有提交，就在这时，程序员去查看自己这个月的工资，发现比往常多了3千元，以为涨工资了非常高兴。但是老板及时发现了不对，马上回滚差点就提交了的事务，将数字改成3.6万再提交。

分析：实际程序员这个月的工资还是3.6万，但是程序员看到的是3.9万。他看到的是老板还没提交事务时的数据。这就是脏读。因此，在这种隔离级别下，查询是不会加锁的，也由于查询的不加锁，所以这种隔离级别的一致性是最差的，可能会产生“脏读”、“不可重复读”、“幻读”。如无特殊情况，基本是不会使用这种隔离级别的。

那怎么解决脏读呢？Read committed！读提交，能解决脏读问题。

### 读提交（Read Committed）

读提交，顾名思义，就是只能读到已经提交了的内容

事例：程序员拿着信用卡去享受生活（卡里当然是只有3.6万），当他埋单时（程序员事务开启），收费系统事先检测到他的卡里有3.6万，就在这个时候！！程序员的妻子要把钱全部转出充当家用，并提交。当收费系统准备扣款时，再检测卡里的金额，发现已经没钱了（第二次检测金额当然要等待妻子转出金额事务提交完）。程序员就会很郁闷，明明卡里是有钱的…

分析：这就是读提交，若有事务对数据进行更新（UPDATE）操作时，读操作事务要等待这个更新操作事务提交后才能读取数据，可以解决脏读问题。但在这个事例中，出现了一个事务范围内两个相同的查询却返回了不同数据，这就是不可重复读。

这是各种系统中最常用的一种隔离级别，也是SQL Server和Oracle的默认隔离级别。这种隔离级别能够有效的避免脏读，但除非在查询中显示的加锁，如：

```sql
select * from T where ID=2 lock in share mode;

select * from T where ID=2 for update;
```

不然，普通的查询是不会加锁的。

那为什么“读提交”同“读未提交”一样，都没有查询加锁，但是却能够避免脏读呢？

这就要说道另一个机制“快照（snapshot）”，而这种既能保证一致性又不加锁的读也被称为“快照读（Snapshot Read）”

假设没有“快照读”，那么当一个更新的事务没有提交时，另一个对更新数据进行查询的事务会因为无法查询而被阻塞（因为上了X锁，即写锁，所以不能得到S锁，即读锁），这种情况下，并发能力就相当的差。而“快照读”就可以完成高并发的查询，不过，“读提交”只能避免“脏读”，并不能避免“不可重复读”和“幻读”。

那怎么解决可能的不可重复读问题？Repeatable read ！

### 可重复读(Repeated Read)

　　可重复读，顾名思义，就是专门针对“不可重复读”这种情况而制定的隔离级别，自然，它就可以有效的避免“不可重复读”。而它也是MySql的默认隔离级别。

　　事例：程序员拿着信用卡去享受生活（卡里当然是只有3.6万），当他埋单时（事务开启，不允许其他事务的UPDATE修改操作），收费系统事先检测到他的卡里有3.6万。这个时候他的妻子不能转出金额了。接下来收费系统就可以扣款了。

　　分析：重复读可以解决不可重复读问题。写到这里，应该明白的一点就是，不可重复读对应的是修改，即UPDATE操作。但是可能还会有幻读问题。因为幻读问题对应的是插入INSERT操作，而不是UPDATE操作。

什么时候会出现幻读？

　　事例：程序员某一天去消费，花了2千元，然后他的妻子去查看他今天的消费记录（全表扫描FTS，妻子事务开启），看到确实是花了2千元，就在这个时候，程序员花了1万买了一部电脑，即新增INSERT了一条消费记录，并提交。当妻子打印程序员的消费记录清单时（妻子事务提交），发现花了1.2万元，似乎出现了幻觉，这就是幻读。

　　在这个级别下，普通的查询同样是使用的“快照读”，但是，和“读提交”不同的是，当事务启动时，就不允许进行“修改操作（Update）”了，而“不可重复读”恰恰是因为两次读取之间进行了数据的修改，因此，“可重复读”能够有效的避免“不可重复读”，但却避免不了“幻读”，因为幻读是由于“插入或者删除操作（Insert or Delete）”而产生的。

　　那怎么解决幻读问题？Serializable！

### 序列化 Serializable

　　这是数据库最高的隔离级别，这种级别下，事务“串行化顺序执行”，也就是一个一个排队执行。这种级别下，“脏读”、“不可重复读”、“幻读”都可以被避免，但是执行效率奇差，性能开销也最大，所以基本没人会用。


***值得一提的是：大多数数据库默认的事务隔离级别是Read committed，比如Sql Server , Oracle。Mysql的默认隔离级别是Repeatable read。***

MySql如何解决幻读问题？

（1）什么是幻读

前提条件：InnoDB引擎，可重复读隔离级别，使用当前读时。

在一次事务里面，多次查询之后，结果集的个数不一致的情况叫做幻读。而多出来或者少的哪一行被叫做 幻行

表现：一个事务(同一个read view)在前后两次查询同一范围的时候，后一次查询看到了前一次查询没有看到的行。两点需要说明：

1、在可重复读隔离级别下，普通查询是快照读，是不会看到别的事务插入的数据的，幻读只在当前读下才会出现。

2、幻读专指新插入的行，读到原本存在行的更新结果不算。因为当前读的作用就是能读到所有已经提交记录的最新值。
 
（2）幻读产生的原因

行锁只能锁住行，即使把所有的行记录都上锁，也阻止不了新插入的记录。 

（3）如何解决幻读？

在快照读读情况下，mysql通过mvcc来避免幻读。

在当前读读情况下，mysql通过next-key来避免幻读

备注：什么是快照读、当前读

- 快照读, 读取专门的快照 (对于RC，快照（ReadView）会在每个语句中创建。对于RR，快照是在事务启动时创建的)

```sql
简单的select操作即可(不需要加锁,如: select ... lock in share mode, select ... for update)
```

针对的也是select操作

- 当前读, 读取最新版本的记录, 没有快照。 在InnoDB中，当前读取根本不会创建任何快照。

```sql
select ... lock in share mode

select ... for update

insert

update

delete

针对如下操作, 会让如下操作阻塞: 
   
```sql
insert
update
delete
```

- 在RR(可重复读)级别下, 快照读是通过MVVC(多版本控制)和undo log来实现的,

当前读是通过手动加record lock(记录锁)和gap lock(间隙锁)来实现的。所以从上面的显示来看，如果需要实时显示数据，还是需要通过加锁来实现。这个时候会使用next-key技术来实现。所以说

总结一下：

- 为什么会出现“脏读”？因为没有“select”操作没有规矩。
- 为什么会出现“不可重复读”？因为“update”操作没有规矩。
- 为什么会出现“幻读”？因为“insert”和“delete”操作没有规矩。
- “读未提（Read Uncommitted）”能预防啥？啥都预防不了。
- “读提交（Read Committed）”能预防啥？使用“快照读（Snapshot Read）”，避免“脏读”，但是可能出现“不可重复读”和“幻读”。
- “可重复读（Repeated Red）”能预防啥？使用“快照读（Snapshot Read）”，锁住被读取记录，避免出现“脏读”、“不可重复读”，但是可能出现“幻读”。
- “串行化（Serializable）”能预防啥？排排坐，吃果果，有效避免“脏读”、“不可重复读”、“幻读”，不过效果谁用谁知道。

提示：

- spring的事务传播行为是：REQUIRED

- 　　mysql的默认的事务处理级别是：REPEATABLE-READ, 也就是可重复读

- 　　Oracle的默认的事务处理级别是：READ COMMITTED, 也就是读已提交

可以通过更改数据库的默认事务处理级别


## 四：事务几种实现方式　(编程式事务管理、编程式事务管理)　

- 　　（1）编程式事务管理对基于 POJO 的应用来说是唯一选择。我们需要在代码中调用beginTransaction()、commit()、rollback()等事务管理相关的方法，这就是编程式事务管理。
- 　　（2）基于 TransactionProxyFactoryBean的声明式事务管理
- 　　（3）基于 @Transactional 的声明式事务管理
- 　　（4）基于Aspectj AOP配置事务

　注：此处侧重讲解声明式事务，编程式事务在实际开发中得不到广泛使用，仅供学习参考。

[参考网址](https://www.cnblogs.com/myseries/p/10834172.html)


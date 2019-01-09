---
title: java里面的事务解析
date: 2019-01-08 16:54:00 
tags: [java] 
categories: 甘明阳
description: 记录事务的使用
---

##### 概念

事务，一般写作Transcation，其实是一组操作，里面包含许多单一的逻辑，只要有一个逻辑没有执行成功，那么都算失败，数据会回滚。

##### 为什么要有事务

为了确保逻辑的成功，例如：银行的转账，A向B转账1000块钱，逻辑是A的账户先减少1000块钱，然后B的账户多了1000块钱，这两个逻辑要同时成立转账才算成功，假如B的账户增加1000的时候出了异常，那么交易就算失败，数据回滚到交易前的状态。

mysql默认自动提交事务，我们可以手动关闭

```
set autocommit = off;
```

查看事务状态

 ```bash
show variables like '%commit%';
 ```

##### 使用命令行方式演示事务

开启事务

```bash
start transaction;
```

提交或者回滚事务

```bash
commit; 提交事务， 数据将会写到磁盘上的数据库

rollback ;  数据回滚，回到最初的状态

```

从start transcation 到commit或者rollback叫一个事务的结束

### 使用代码方式演示事务

> 代码里面的事务，主要是针对连接来的。 
>
> 1. 通过conn.setAutoCommit（false ）来关闭自动提交的设置。

> 1. 提交事务  conn.commit();

> 1. 回滚事务 conn.rollback();

```java
@Test
public void testTransaction(){
	
	Connection conn = null;
	PreparedStatement ps = null;
	ResultSet rs = null;
	try {
		conn = JDBCUtil.getConn();
		
		//连接，事务默认就是自动提交的。 关闭自动提交。
		conn.setAutoCommit(false);
		
		String sql = "update account set money = money - ? where id = ?";
		ps = conn.prepareStatement(sql);
		
		//扣钱， 扣ID为1 的100块钱
		ps.setInt(1, 100);
		ps.setInt(2, 1);
		ps.executeUpdate();
		
		
		int a = 10 /0 ;
		
		//加钱， 给ID为2 加100块钱
		ps.setInt(1, -100);
		ps.setInt(2, 2);
		ps.executeUpdate();
		
		//成功： 提交事务。
		conn.commit();
		
	} catch (SQLException e) {
		try {
			//事变： 回滚事务
			conn.rollback();
		} catch (SQLException e1) {
			e1.printStackTrace();
		}
		e.printStackTrace();
		
	}finally {
		JDBCUtil.release(conn, ps, rs);
	}
}
```

事务的特性ACID

* 原子性

  > 指的是事务中包含的逻辑，不可分割

* 一致性

  > 数据的完整性保持一致

* 隔离性

  > 事务在执行期间不应该受到其他事务影响

* 持久性

  > 指的是 事务执行成功，那么数据应该持久保存到磁盘上。

##### 事务的安全隐患

不考虑隔离级别设置，那么会出现以下的问题

脏读 不可重复读 幻读

* 脏读

   一个事务读到另外一个事务还未提交的数据，解决这个问题可以设置隔离级别为 读已提交 

  ```bash
set session transcation isolation level read committed;
  ```

* 不可重复读 

一个事务读到了另外一个事务提交的数据 ，造成了前后两次查询结果不一致。可以设置隔离级别为 可重复读

mysql默认隔离级别是  可重复读，因为一个事务是不应该被另一个事务影响的

```bash
set session transcation isolation level repeatable read;
```

* 幻读

  一个事务读到另一个事务已提交的插入的数据，导致多次查询结果不一致。设置隔离级别为可串行化可以解决

  ```bash
  set session transcation isolation level serializable;
  ```

  这个隔离级别设置当前只能一个事务执行，别的事务要等待这个事务完结后才能对数据进行操作，谁先打开事务就有执行事务的权利，其他的事务要等这个事务提交或回滚才能执行，效率比较低

  隔离级别的效率，从高到低

  读未提交 > 读已提交 > 可重复读 > 可串行化

  安全程度

  可串行化 > 可重复读 > 读已提交 > 读未提交

查询隔离级别

 `select @@tx_isolation;`

##### 丢失更新

多个事务对一份数据修改造成数据更新丢失的情况，例如：

一份数据为name 李四  money 1000 ，假如A事务修改姓名为张三，为提交的时候B事务开启，然后A事务提交，然后B事务修改money为800，B事务提交，这个时候最终数据为李四 800，A事务修改的数据丢失了，这是因为事务之间是不应该相互影响的。

解决方法有两种：悲观锁和乐观锁

##### 悲观锁

悲观锁认为数据是一定会被其他事务修改的，在查询的时候，后面加上for update，这是利用了数据库的一种锁机制，叫排他锁，这样B事务查询的时候界面就卡住了，有点像隔离级别为可串行化的意思。

##### 乐观锁

乐观锁认为数据一定不会被修改，要求程序员自己控制，程序员添加一个字段为version，值为0，A事务提交后改为1，这个时候B事务查询的时候，对比数据库和自己的version，不一样，不允许提交。要重新查询数据再修改。

##### 连接池

###### 为什么要连接池？

数据库的连接对象创建工作，比较消耗性能。

###### 连接池的原理

一开始现在内存中开辟一块空间（集合） ， 一开先往池子里面放置 多个连接对象。  后面需要连接的话，直接从池子里面去。不要去自己创建连接了。  使用完毕， 要记得归还连接。确保连接对象能循环利用。




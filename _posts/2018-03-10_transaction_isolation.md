---
title: 事务的隔离级别
date: 2018-03-10 12:13:59
tags: mysql
permalink: transaction-isolation
keywords: 事务, 隔离级别
rId: MB-18031001
---

## 事务的ACID特性

1. 原子性 
2. 一致性（事务处理前后从一个一致性状态变换到另一个一致性状态） 
3. 隔离性（多个并发事务间的隔离） 
4. 持久性 

## 无隔离处理下的几种问题

### 1. 脏读

并发的其中一个事务读取另一个事务未提交的内容，当另一个事务回滚后发现读的是无效数据 

### 2. 不可重复读

并发的其中一个事务在一开始读取了一次数据，并在另一个事务提交后又读取了一次数据，发现两次读取的数据不一致 

### 3. 幻读（虚读）

并发的其中一条事务对表内的所有行执行字段a的值从1变为2，这时在另一条事务中在表内添加了另一条a的值为1的数据，前一条事务会发现执行完后，表内仍有a字段值为1的记录 



## 事务隔离级别

针对上述问题，设置了四种事务的隔离级别，由用户根据业务选择使用哪种隔离级别 ，在MySQL中默认是Repeatable Read级别，Oracle中默认是Read Committed级别。

### 1. 读未提交(READ UNCOMMITTED)

可以读到其他事务未提交的数据。 

### 2. 读已提交(READ COMMITTED)

只能读到其他事务已提交的数据。 

### 3. 可重复读(REPEATABLE READ)

在读已提交的基础上，增加一点，本条事务内，读同一份数据，以该事务第一次读取的数据的备份为标准（即使在此期间其他事务提交了该数据的修改，读到的仍是原数据）。 

### 4. 串行(SERIALIZABLE)

串行执行，该事务不能与其他事务同时操作同一个数据，必须等待操作数据的事务或串行事务提交才能继续运行。Serializable的事务先查询或修改了该条记录，则其他事务对该记录的操作均会被阻塞。 


事务隔离会在写数据的时候给数据加上排他锁，而串行事务下读数据的时候也会给数据加上排他锁。 



## 其他

查看当前事务隔离级别  `SELECT @@TX_ISOLATION`

设置当前事务隔离级别  `SET SESSION TRANSACTION ISOLATION LEVEL read uncommitted / read committed / repeatable read / serializable`

Spring中注解设置 `@Transactional(isolation = Isolation.REPEATABLE_READ)`
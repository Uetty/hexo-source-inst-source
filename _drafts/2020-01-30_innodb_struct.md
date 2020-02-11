---
title: InnoDB体系架构
date: 2020-01-30 13:38:42
tags: mysql
permalink: innodb-struct
keywords: innodb, buffer
rId: MB-20013001
---

InnoDB存储引擎具有行锁设计、支持事务、支持外键、支持MVCC、支持插入缓冲、支持自适应哈希索引等特点，其整体体系架构主要由后台线程、内存池、文件系统三部分组成，如下图所示：

![图1](../static/MB20013001-1.png)

接下来会针对后台线程和内存池展开介绍。

## 内存池





## 后台线程

### 1. Master线程

负责任务

1. 负责将缓冲池中的数据异步刷新到磁盘
2. 合并插入缓冲

### 2. IO线程

分为4种线程

1. Read线程（默认4条线程，由`innodb_read_io_thread`参数控制）
2. Write线程（默认4条线程，由`innodb_write_io_thread`参数控制）
3. Insert Buffer线程（1条线程）
4. Log线程（1条线程）



### 3. Purge线程

1. UNDO页回收

### 4. Cleaner线程

1. 将脏页刷新







##  概念

**1. 索引页**



**2. 数据页**



**3. undo页**



**4. 插入缓冲**



**5. 自适应哈希索引**



**6. 锁信息**



**7. 数据字典**



**8. 脏页**






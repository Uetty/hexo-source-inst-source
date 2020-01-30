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

## 后台线程


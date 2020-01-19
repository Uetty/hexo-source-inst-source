---
title: Linux限制进程CPU上限
date: 2019-11-25 19:20:11
tags: linux
permalink: linux-cgroup-cpu
keywords: linux, 限制cpu使用率
rId: MB-19112501
---

要实现linux限制进程上限的功能，需要用到Cgroups技术，它的全程Linux Control Group，用于限制一个进程组能够使用的资源（CPU、内存、磁盘、网络带宽）上限，还能够对进程进行优先级设置，以及进行将进程挂起恢复的操作。

Cgroups给用户暴露出来的操作接口是文件系统，以目录和文件的方式组织在`/sys/fs/cgroup`路径下。

执行`ls /sys/fs/cgroup`命令可以看到如下文件列表

```
root@ubuntu:$ ls /sys/fs/cgroup
blkio  cpuacct      cpuset   freezer  memory   net_cls,net_prio  perf_event  rdma     unified
cpu    cpu,cpuacct  devices  hugetlb  net_cls  net_prio          pids        systemd
```

代表了可操作的各种资源

以cpu资源为例，查看`cpu`目录，执行`ls /sys/fs/cgroup/cpu`可以看到如下列表

```
root@ubuntu:$ ls /sys/fs/cgroup/cpu
cgroup.clone_children  cpuacct.usage_all          cpuacct.usage_user  notify_on_release  user.slice
cgroup.procs           cpuacct.usage_percpu       cpu.cfs_period_us   release_agent
cgroup.sane_behavior   cpuacct.usage_percpu_sys   cpu.cfs_quota_us    system.slice
cpuacct.stat           cpuacct.usage_percpu_user  cpu.shares          tasks
cpuacct.usage          cpuacct.usage_sys          cpu.stat
```

其中`tasks`文件列出了受限制的进程pid列表，`cpu.cfs_period_us`和`cpu.cfs_quota_us`文件搭配使用可以达到限制cpu使用率的目的，打开两个文件分别看到内容如下

`cpu.cfs_period_us`

```100000
100000
```

`cpu.cfs_quota_us`

```
-1
```

表示100000us时间内不限制分配的cpu时间长度（-1表示不限制，如果将-1换成30000则表示允许分配30000us即30ms的cpu时间）



## 实践一下

cgroup是树型结构的，可以在当前树下新建子节点，根目录`/sys/fs/cgroup/cpu`一般表示整个计算机。所以我们不是直接操作`/sys/fs/cgroup/cpu`下面的文件，而是在目录树下创建一个名为`test`的子目录，查看目录可以发现操作系统会在新建的`test`目录下自动生成了该系统对应的资源限制文件

```
root@ubuntu:$ cd /sys/fs/cgroup/cpu
root@ubuntu:/sys/fs/cgroup/cpu$ mkdir test
root@ubuntu:/sys/fs/cgroup/cpu$ cd test
root@ubuntu:/sys/fs/cgroup/cpu$ 
root@ubuntu:/sys/fs/cgroup/cpu/test$ ls
cgroup.clone_children  cpuacct.usage         cpuacct.usage_percpu_sys   cpuacct.usage_user  cpu.shares         tasks
cgroup.procs           cpuacct.usage_all     cpuacct.usage_percpu_user  cpu.cfs_period_us   cpu.stat
cpuacct.stat           cpuacct.usage_percpu  cpuacct.usage_sys          cpu.cfs_quota_us    notify_on_release
```

接下来，先启动两个用于测试的死循环进程，进程号分别为15007和15952

```
root@ubuntu:/sys/fs/cgroup/cpu/test$ while : ; do : ; done &
[1] 15007
root@ubuntu:/sys/fs/cgroup/cpu/test$ while : ; do : ; done &
[2] 15952
```

查看此时的cpu使用情况，cpu使用率达到了93.8%

```
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
15952 root      20   0   22776   2960   1212 R 50.0  0.1  18:18.90 bash
15007 root      20   0   22776   1684      0 R 43.8  0.1  24:04.92 bash
```

修改`cpu.cfs_quota_us`文件和`tasks`文件

```
root@ubuntu:/sys/fs/cgroup/cpu/test$ echo "40000" > cpu.cfs_quota_us
root@ubuntu:/sys/fs/cgroup/cpu/test$ echo -e "15007\n15952" > tasks
```

查看此时的cpu使用情况，发现cpu使用率成功降下来了

```
  PID USER      PR  NI    VIRT    RES    SHR S %CPU %MEM     TIME+ COMMAND
15952 root      20   0   22776   2960   1212 R 20.3  0.1  20:04.56 bash
15007 root      20   0   22776   1684      0 R 19.9  0.1  25:50.57 bash
```

### 添加限制的进程

```
root@ubuntu:/sys/fs/cgroup/cpu/test$ echo -e "1234" > tasks
```

虽然上面写文件使用的符号是单大于号`>`，但实际上并不会覆盖原本文件，实际效果是添加一个pid为1234的进程到该cgroup组中

### 移除进程限制

在cgroup数中，一个进程必须属于一个cgroup，所以不能从一个cgroup凭空删除一个进程，只能将进程移动到其他cgroup节点，所以删除操作也便成为了将进程移动到cgroup树的根节点

```
root@ubuntu:/sys/fs/cgroup/cpu/test$ echo "15007" > ../tasks
```

### 删除cgroup组

删除cgroup文件夹即可，但这里使用`rm -rf test`是行不通的，需要使用`rmdir`命令。（删除时需确保cgroup组中没有进程存在）

```
root@ubuntu:/sys/fs/cgroup/cpu$ rmdir test
```


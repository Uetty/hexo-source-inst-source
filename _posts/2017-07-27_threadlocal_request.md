---
title: SpringMVC 在controller层中注入request（不会产生线程安全问题）
date: 2017-07-27 19:20:10
tags: spring
permalink: spring-threadlocal
keywords: request, 线程安全, spring
rId: MB-17072701
---

之前做项目的时候，在controller中多个方法需要用到request和session获取用户相关值，为了方便写了个BaseController所有controller基础它，在BaseController中Autowired注解request和httpsession，这样子，不需要在各个接口单独加上request入参。最近偶然看到一篇博客称这种方式会有县城安全问题，所以重新复核了一遍。

阅读了spring的部分源码之后，发现spring是通过threadlocal方式存储的reqeust信息，而spring包下的HttpServletRequest是spring为了达到这个目的而对ServletReqeust进行的一层壳封装，通过这层封装达到对ThreadLocal中的request的调用。因此，直接将HttpServletRequest注入到成员变量上不会产生线程安全问题。

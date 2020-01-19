---
title: SpringMVC自定义配置自动注入对象
date: 2018-02-25 15:20:10
tags: spring
permalink: spring-autowire-resolver
keywords: spring, 注入对象, 自定义
rId: MB-18022501
---

1. 继承org.springframework.web.method.support.HandlerMethodArgumentResolver的类test.UserArgumentResolver
2. 配置文件配置参数

```
<mvc:annotation-driven>
    <mvc:argument-resolvers>
    	<bean class="com.xxx.UserArgumentResolver"/>
    </mvc:argument-resolvers>
</mvc:annotation-driven>
```


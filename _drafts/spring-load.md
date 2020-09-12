## spring 加载

传入参数

1. 从currentThread获取AppClassloader，作为项目classloader

2. 通过反射推断WebApplication类型。如果存在`reactive.DispatcherHandler`且不存在两种`servlet.DispatcherServlet`，则为REACTIVE类型；如果同时存在`javax.servlet.Servlet`和`context.ConfigurableWebApplicationContext`，则为SERVLET类型；否则为NONE类型。

3. 通过classloader加载各jar包`META-INF/spring.factories`目录下的Factory类，设置键名为`ApplicationContextInitializer`和ApplicationListener的类（多个类按照Order排序）
4. 通过Java调用栈推断并构造main函数所在的Java类实例
5. 完成`SpringApplication`类实例化，运行SpringApplication实例run方法
6. 通过classloader加载jar包META-INF/spring.factories目录下的Factory类
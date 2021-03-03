# SpringBoot 框架解析系列一（出入参）





## 入参处理

org.springframework.web.method.support.HandlerMethodArgumentResolverComposite#getArgumentResolver

org.springframework.web.method.support.HandlerMethodArgumentResolver#supportsParameter



org.springframework.data.web.ProxyingHandlerMethodArgumentResolver

org.springframework.web.method.annotation.RequestParamMethodArgumentResolver

org.springframework.web.method.annotation.RequestParamMapMethodArgumentResolver

org.springframework.web.servlet.mvc.method.annotation.PathVariableMethodArgumentResolver

org.springframework.web.servlet.mvc.method.annotation.PathVariableMapMethodArgumentResolver

org.springframework.web.servlet.mvc.method.annotation.MatrixVariableMethodArgumentResolver

org.springframework.web.servlet.mvc.method.annotation.MatrixVariableMapMethodArgumentResolver

org.springframework.web.method.annotation.ModelAttributeMethodProcessor

org.springframework.web.servlet.mvc.method.annotation.RequestResponseBodyMethodProcessor







方法调用

org.springframework.web.servlet.handler.AbstractHandlerMapping#getHandlerExecutionChain

org.springframework.web.servlet.handler.AbstractHandlerMapping#adaptedInterceptors

org.springframework.web.servlet.DispatcherServlet#getHandlerAdapter



org.springframework.web.servlet.DispatcherServlet#doDispatch  -> ha.handle

org.springframework.aop.framework.ReflectiveMethodInvocation#interceptorsAndDynamicMethodMatchers

org.springframework.security.access.intercept.AbstractSecurityInterceptor#beforeInvocation

accessDecisionManager.decide

org.springframework.security.access.expression.method.ExpressionBasedPreInvocationAdvice






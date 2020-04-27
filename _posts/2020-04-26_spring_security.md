---
title: SpringBoot下使用Security踩坑
date: 2020-04-26 19:22:22
tags: java
permalink: spring-security
keywords: spring-security, 配置
rId: MB-20042601
---

该文基于SpringBoot版本2.1.8.RELEASE，案例仓库以后有空时整理后补上。

## （一）配置文件简单示例

继承WebSecurityConfigureAdapter类，加上@EnableWebSecurity注解，并实现configure方法（注意这个方法有三种入参形式，下面只是其中一种，后面的篇幅中会看到另外一种），下面只是一个简单的配置文件，仅配置了一些简单的URL路径相关的权限。

**SecurityConfigure.class**

```
@EnableWebSecurity
public class SecurityConfigure extends WebSecurityConfigurerAdapter {
	@Override
    protected void configure(HttpSecurity http) throws Exception {
		http.authorizeRequests()
				// 允许这些路径不用验证
				.antMatchers("/test", "/static").permitAll()
				// 这些路径需要USER角色的权限
				.antMatchers("/user/**").hasRole("USER")
				// 其余的路径都需要登录
				.anyRequest().authenticated();
				// 实际还会有其他设置，这里不列出
	}
}
```



## （二）设置登录接口

spring security已经帮你完成了登录验证的代码，无需再在controller中进行编码。

该部分前两个案例意义不大，建议可以直接跳（Ⅲ）案例。

### （Ⅰ）最简陋且无意义的登录配置案例

该案例与上面一样，配置在实现`WebSecurityConfigureAdapter`接口的类中，如下所示：

**SecurityConfigure.class**

```
@EnableWebSecurity
public class SecurityConfigure extends WebSecurityConfigurerAdapter {
	@Override
    protected void configure(HttpSecurity http) throws Exception {
    	// 这里为了测试方便暂时先禁用csrf
        http.csrf().disable();
		http
			.authorizeRequests()
			// 拥有其中一项权限即可，"USER"权限与UserDetails的Authorities中"USER"匹配
			// .antMatchers("/api/user/**").hasAnyAuthority("USER", "ADMIN")
			// 其余的路径都需要登录
			.anyRequest().authenticated();
			// 上面是权限相关的配置，后面第五部分详细介绍
			
			.and()
            .formLogin()
            // 登录页面地址（也即判断请求未登录验证时，重定向到的页面）
            .loginPage("/login")
            // 登录接口路径
            .loginProcessingUrl("/api/guard/login")
            // 登录成功后重定向到的页面地址
            .defaultSuccessUrl("/index")
            // 登录失败时重定向到的页面
            .failureUrl("/login-error")
            // 登录接口传递用户名的参数名
            .usernameParameter("username")
            // 登录接口传递密码的参数名
            .passwordParameter("password")
            ;
	}
}
```

**在这种配置下，既没有指定自定义登录验证逻辑，又没有指定用户名密码，spring security会自动生成密码**，会在启动时在控制台打印如下所示（可以看到是INFO级别log，看不到的话除了可能是配置问题还要考虑日志级别问题）

```
2020-04-15 01:14:18,430 [main] o.s.b.a.s.s.UserDetailsServiceAutoConfiguration.getOrDeducePassword:83 [INFO] - 

Using generated security password: f31ad0b3-5247-4a2a-864c-b6a3e4a0b936
```

前端代码是从spring官网拿的文件，是前后端不分离的thymeleaf文件（前端案例之所以是不分离，是因为security还提供了CSRF功能，这个会通过不分离的页面自动将token写入到页面，CSRF后面会说，涉及到安全的，这里不用关心），具体前端代码这里就不贴了。为了方便讲解，下面直接上POST MAN的请求解说。

登录测试，是POST请求，在POST MAN中大概就是这样

```
http://localhost:9090/api/guard/login
form表单参数如下
username: user
password: f31ad0b3-5247-4a2a-864c-b6a3e4a0b936
```

登录成功，返回的是index页面的内容。

上面的配置中登录权限控制，也是没问题可以用的，就不展示验证的篇幅了。

**这个案例并不实用，仅有的一点意义是展示一下security跳转地址的配置以及登录参数名的配置。**

### （Ⅱ）增加一点配置的仍旧无意义登录配置案例

在这个案例中，将自动生成的密码改成了代码中写死的密码。配置如下：

**SecurityConfigure.class**

```
@EnableWebSecurity
public class SecurityConfigure extends WebSecurityConfigurerAdapter {
	@Override
    protected void configure(HttpSecurity http) throws Exception {
    	// 这里跟第(Ⅰ)个案例的配置一样，略了，从上面拷下来就行
        ...
	}
	
	@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    	// 注意这里跟上面方法名相同，参数不同，不要混了
    	// 这个会将密码加密后再存在内存中
    	UserDetails build = User.withDefaultPasswordEncoder()
                .username("vince")
                .password("123456")
                .roles("USER")
                .build();
        auth.inMemoryAuthentication()
        .withUser("vince").roles("USER").password(build.getPassword());
    }
}
```

登录测试还是跟上面一个案例一样，不再赘述。

**这个案例同样意义不大，仅有的意义大概就是展示了一下Spring内存对密码是有加密的特性**

### （Ⅲ）真正有意义的登录案例

这个案例，是我们真正常用的，直接看配置吧。

**SecurityConfigure.class**

```
@EnableWebSecurity
public class SecurityConfigure extends WebSecurityConfigurerAdapter {

	@Autowire
	UserDetailsService detailsService;
	
	@Override
    protected void configure(HttpSecurity http) throws Exception {
    	// 这里跟第(Ⅰ)个案例的配置一样，略了，从上面拷下来就行
        ...
	}
	
	/**
	 * 这里为security指定一个加密方式（这个加密目的仅是内存安全，数据库里最好还是要单独进行md5加密）
	 */
	@Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
	
	@Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    	// 注意这里跟上面方法名相同，参数不同，不要混了
    	// 设置自定义获取用户信息的业务类
        auth.userDetailsService(detailsService);
    }
}
```

**UserDetailsServiceImpl.class**

```
@Component
public class UserDetailsServiceImpl implements UserDetailsService {

    @Autowired
    UserDao userDao;
    @Autowired
    PasswordEncoder passwordEncoder;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {

        User user = userDao.getByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("username[" + username + "] not found");
        }
        ArrayList<GrantedAuthority> authorities = Lists.newArrayList();
        return new org.springframework.security.core.userdetails.User(username,
                passwordEncoder.encode(user.getPassword()), authorities);
    }
}
```

上面的代码，创建了实现`UserDetailsService`接口的类，并在`SecurityConfigure`类中增加用于设置`UserDetailsService`的configure方法。

当每次有登录请求（前面配置的登录接口是`/api/guard/login`）进来的时候，Spring Security框架就会自动调用`UserDetailsService`的`loadUserByUsername`方法，获取正确用户名密码，并自动执行验证逻辑。登录测试案例与前面一样，就不在贴了。

这里需要注意的点是：

> 需要用@Bean注解申明密码加密类`BCryptPasswordEncoder`，以便在`UserDetailsService`类中注入。 
> 特别要注意的是不要在`UserDetailsService`中通过new直接生成`BCryptPasswordEncoder`类的实例，因为Security内部默认是使用`DelegatingPasswordEncoder`类进行密码加密的，这样会导致与`UserDetailsService`中的加密方式不一致。通过注解申明了`BCryptPasswordEncoder`之后，Security内部也就会换成该加密方式。
> 而且，即使在已经注解声明了`BCryptPasswordEncoder`的情况下，`UserDetailsService`中也不要使用new的方式使用，因为`BCryptPasswordEncoder`内部会默认随机生成加盐字符串，同一个类的不同实例，加盐字符串不一样，加密出来的结果自然也会是不一样的。



## （三）自定义登录接口的响应返回值

有时候我们需要判断是接口请求还是页面请求来定义返回json还是页面，就需要自定义响应内容。自定义登录响应有两种方法，一种就是像第二节案例(Ⅰ)一样，将成功的url和失败的url指向controller接口处理。

第二种是指定自定义处理器的方式，如下所示（下面代码另外包含无权限情况下的响应处理）：

修改如下配置

**SecurityConfigure.class**

```
@EnableWebSecurity
public class SecurityConfigure extends WebSecurityConfigurerAdapter {

	// 这几行是新增的注解，其他与第二节的(Ⅱ)案例一样
	@Autowired
    SecurityExceptionHandler exceptionHandler;
    @Autowired
    AuthenticationHandler authenticationHandler;
	
	@Override
    protected void configure(HttpSecurity http) throws Exception {
    	// 这里为了测试方便暂时先禁用csrf
        http.csrf().disable();
		http.formLogin()
                // 登录页面地址
                .loginPage("/login")
                // 登录接口路径
                .loginProcessingUrl("/api/guard/login2")
                // 登录接口入参名称
                .usernameParameter("username")
                .passwordParameter("password")
                // 下面几行为新增的
                // 登录成功和登录失败的处理
                .failureHandler(authenticationHandler)
                .successHandler(authenticationHandler)
                .and()
                // 无权限的处理
                .exceptionHandling()
                .accessDeniedHandler(exceptionHandler)
                .authenticationEntryPoint(exceptionHandler);
	}
	
	// 下面其他部分与第二节的(Ⅲ)案例一样
	...
}
```

新增几个类分别实现`AuthenticationFailureHandler`、`AuthenticationSuccessHandler`、`AuthenticationEntryPoint`、`AccessDeniedHandler`接口的类，如下所示：

**AccessDeniedHandlerImpl.class**

```
/**
 * 无权限情况下的返回值处理
 */
@Component
public class AccessDeniedHandlerImpl implements AuthenticationEntryPoint, AccessDeniedHandler {

	/**
     * 没有登录的情况下访问需要登录的接口
     */
    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response,
                         AuthenticationException authenticationException) throws IOException, ServletException {

        handleAccessFailed(request, response, authenticationException);
    }

    /**
     * 已登录但没有权限的情况下访问需要角色权限的接口
     */
    @Override
    public void handle(HttpServletRequest request, HttpServletResponse response,
                       AccessDeniedException accessDeniedException) throws IOException, ServletException {

        handleAccessFailed(request, response, accessDeniedException);
    }

    public void handleAccessFailed(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {

        String requestURI = httpServletRequest.getRequestURI();
        if (StringUtil.startsWith(requestURI, "/api")) { // 接口请求
            httpServletResponse.setCharacterEncoding("utf-8");
            httpServletResponse.setContentType("application/json;charset=UTF-8");
            httpServletResponse.getWriter().print("{\"msg\": \"user has no access\",\"status\": 100002}");

            httpServletResponse.flushBuffer();

        } else { // 页面请求
            // 重定向页面
            httpServletResponse.sendRedirect("/login");
        }
    }
}
```

**AuthenticationHandler.class**

```
/**
 * 登录成功/失败时的响应处理
 */
@Component
public class AuthenticationHandler implements AuthenticationFailureHandler, AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationFailure(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, AuthenticationException e) throws IOException, ServletException {
        String requestURI = httpServletRequest.getRequestURI();
        httpServletResponse.setCharacterEncoding("utf-8");
        httpServletResponse.setContentType("application/json;charset=UTF-8");
        httpServletResponse.getWriter().print("{\"msg\": \"login failed\",\"status\": 100001}");
        httpServletResponse.flushBuffer();
    }

    @Override
    public void onAuthenticationSuccess(HttpServletRequest httpServletRequest, HttpServletResponse httpServletResponse, Authentication authentication) throws IOException, ServletException {
        String requestURI = httpServletRequest.getRequestURI();
        httpServletResponse.setCharacterEncoding("utf-8");
        httpServletResponse.setContentType("application/json;charset=UTF-8");
        httpServletResponse.getWriter().print("{\"msg\": \"success\",\"status\": 1}");
        httpServletResponse.flushBuffer();
    }
}
```

`AuthenticationEntryPoint`接口对应访问需要登录的接口被拒绝的情况下的处理，`AccessDeniedHandler`接口对应访问需要角色权限的接口被拒绝的情况下的处理，在这里我根据请求的路径判断是接口请求还是页面请求，接口请求返回json，页面请求重定向到登录页面。`AuthenticationFailureHandler`和`AuthenticationSuccessHandler`则分别对应登录失败和登录成功的状况，因为登录只有接口，所以返回的直接是json。

这种配置的情况下，有一个好处是`AuthenticationFailureHandler`可以根据前面抛出的异常来判定失败原因，具体决定给前端的响应状态码。

## （四）如何做验证码校验

做验证码校验，一种方法是在`UserDetailsServiceImpl`中做验证码的校验，如果验证码错误直接抛出异常，到登录失败处理器中处理该异常，这种方法的不好之处在于，Security会打印非`UsernameNotFoundException`异常，造成日志文件中异常信息过多，但这种方法也很方便。

第二种方式，是在`UsernamePasswordAuthenticationFilter`前增加一个自定义的过滤器。

代码配置如下：

**SecurityConfigure.class**

```
@EnableWebSecurity
public class SecurityConfigure extends WebSecurityConfigurerAdapter {
	// 增加两行过滤器有关的代码，其他不变
	@Autowired
    PreLoginFilter preLoginFilter;
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
    	...
    	// 增加这一行，其他不变
    	http.addFilterBefore(preLoginFilter, UsernamePasswordAuthenticationFilter.class);
    	
    	...
    }
    ...
}
```

**PreLoginFilter.class**

```
/**
 * 登录前置过滤器，可以拿来做验证码校验
 */
@Component
public class PreLoginFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        boolean valid = false;
        // 具体校验逻辑
		...
		
        if (valid) {
            // 校验通过放行
            filterChain.doFilter(request, response);
        } else {
            response.getWriter().write("{\"msg\": \"varifiation code error\",\"status\": 100003}");
            response.flushBuffer();
        }
    }
}
```
## （五）权限认证的配置及自定义

### （Ⅰ）权限模型

Security授权主要由访问决策管理器来决定是否获得权限。Security默认提供的访问决策管理器中可能有多个投票器，如`org.springframework.security.access.vote.AbstractAccessDecisionManager`中代码所示：

**org.springframework.security.access.vote.AbstractAccessDecisionManager.class**

```
public abstract class AbstractAccessDecisionManager implements AccessDecisionManager,
		InitializingBean, MessageSourceAware {
	protected final Log logger = LogFactory.getLog(getClass());

	private List<AccessDecisionVoter<? extends Object>> decisionVoters;
	
	// 其他代码略
	... 
}
```

投票器可以投赞成票、反对票和弃权票，分别对应数值1、-1、0。多个投票器会有多种不同的投票结果，最终的结果是授权还是拒绝，由访问决策管理器决定。

### （Ⅱ）访问决策管理器

Security提供了三种访问决策管理器：

1. 基于肯定的访问决策管理器(`AffirmativeBased`) —— 一个及以上赞成票视为投票通过，Security默认使用的就是这个。
2. 基于一致肯定的访问决策管理器(`UnanimousBased`) —— 没有拒绝票且有赞成的情况下通过
3. 基于共识的访问决策管理器(`ConsensusBased`) —— 赞成票数大于拒绝票数的情况下通过

这三个投票管理器，均有一个布尔类型的属性可以设置是否允许全部弃权，如果允许，在全部弃权的情况下最终视为投票通过。特殊的，对于`ConsensusBased`投票管理器，还有一个额外的布尔值用于设定是否允许通过票与拒绝票相同，如果允许，那么两边票数相同的情况也视为投票通过。

一般情况下，这三种投票管理器已经能够满足我们的需求了，不需要重新再写，我们只需要选择一个使用即可。如有需要，自定义的决策管理器需要实现`org.springframework.security.access.AccessDecisionManager`接口，也可以继承抽象类`org.springframework.security.access.vote.AbstractAccessDecisionManager`，该抽象类实现了决策管理器接口，前面权限模型中说到的投票器概念就是在该抽象类中引入的。在配置类中设置决策管理器的配置方法如下：

```
// 该配置位于上面提到的SecurityConfigure.class文件中
http.authorizeRequests()
	.accessDecisionManager(accessDecisionManager);
```

### （Ⅲ）投票器

Security默认提供了几种投票器：`RoleVoter`、`PreInvocationAuthorizationAdviceVoter`、`AuthenticatedVoter`、`Jsr250Voter`、`WebExpressionVoter`。Springboot中Security默认使用的是`WebExpressionVoter`投票器（对于大部分应用只需要关心这种配置方式即可，原因见后一段）。



有一个比较麻烦且一直没找到解决方案的问题：在Springboot的Java方式配置中，不使用`WebExpressionVoter`情况下，使用其他投票器，暂时没找到简单有效的Java代码配置方式。在使用XML方式配置的Spring项目中，不使用`WebExpressionVoter`是可以做到的，只需要额外指定`<http use-expressions="false">`即可。

在`org.springframework.security.config.http.HttpConfigurationBuilder`类中可以看到对XML文件中`use-expressions`解析的源码，有需要且有足够时间研究的前提下，可以参考源码研究Java配置方式（我有稍微看了一点，大概不是很容易，因为XML方式配置和Java方式配置上有很大的不同）。



在保留使用`WebExpressionVoter`情况下，增加其他投票器，还是可以做到配置的，但实际上意义是不大的。因为，**`WebExpressionVoter`本身就能够做到覆盖其他投票器的功能**。保留`WebExpressionVoter`情况下，增加其他投票器的配置方式如下：

**SecurityConfigure.class**

```
@EnableWebSecurity
public class SecurityConfigure extends WebSecurityConfigurerAdapter {
	... 其他配置略
	
    public AffirmativeBased affirmativeBased() {
        List<AccessDecisionVoter<?>> collect = new ArrayList<>();
        // 这里出于方便以增加RoleVoter投票器为例，实际使用中替换为自定义投票器
        RoleVoter roleVoter = new RoleVoter(); 
        collect.add(roleVoter);
        
        WebExpressionVoter webExpressionVoter = new WebExpressionVoter();
        collect.add(webExpressionVoter);
        return new AffirmativeBased(collect);
    }
    
    @Override
    protected void configure(HttpSecurity http) throws Exception {
    	... 其他配置略
    	http.authorizeRequests()
    			// Security中没法单独添加投票器，所以需要通过将决策管理器替换的方式
                .accessDecisionManager(affirmativeBased())
    	... 其他配置略
    }
	... 其他配置略
}
```

#### ① RoleVoter

基于角色的投票器，在`UserDetailsService`中通过`loadUserByUsername`方法我们获取了用户信息，同时也会返回`GrantedAuthority`列表，`RoleVoter`的权限控制便是基于`GrantedAuthority`列表进行判定的。

#### ② WebExpressionVoter

该投票器可以通过表达式进行权限验证，它可以使用内置的表达式，也可以自定义解析代码。

它有多种配置代码，像这样：

```
http.authorizeRequests()
	// 对/api/**下所有路径放行
	.antMatchers("/api/login").permitAll();
```

像这样：

```
http.authorizeRequests()
    .antMatchers("/api/login","/api/forgetPassword").permitAll()
    // 需要USER角色或ADMIN角色，分别与GrantedAuthority的"ROLE_USER"、"ROLE_ADMIN"匹配
    .antMatchers("/api/**").hasAnyRole("USER","ADMIN")
	// 剩余路径，需要登陆才能访问
	.anyRequest().authenticated();
```

像这样：

```
http.authorizeRequests()
	// 需要ADMIN权限，与GrantedAuthority中的"ADMIN"匹配
	.antMatchers("/api/admin/**").hasAuthority("ADMIN")
	// 需要 STAFF角色 且 IP地址段满足给定值
	.antMatchers("/api/staff/**").access("hasRole('STAFF') and hasIpAddress('192.168.1.0/24')");
```

像这样限定权限

```
http.authorizeRequests()
	// 需要读用户的权限
	.antMatchers("/api/user/list").access("hasPermission('user', 'read')")
	// 需要读特定文件的权限
	.antMatchers("/api/file/{fileId}").access("hasPermission(#fileId, 'file', 'read')");
```

在上面的表达式中，`hasPermission`需要自己定义，如下所示，有需要定义的方法有两个，分别对应上面调用的两个方法，可以发现表达式中都默认省略的第一个参数，因为它是必定会传递的

```
@Component
public class MyPermissionEvaluator implements PermissionEvaluator {
    @Override
    public boolean hasPermission(Authentication authentication, Object targetDomainObject, Object permission) {
        // 自定义逻辑
        return true;
    }
    @Override
    public boolean hasPermission(Authentication authentication, Serializable targetId, String targetType, Object permission) {
        // 自定义逻辑
        return true;
    }
}
```

还可以有更丰富的自定义表达式计算器

```
http.authorizeRequests()
    // 使用自定义表达式计算
    .antMatchers("/api/**")
    .access("@myExpressionInterceptor.eval1(authentication,request)")
	// 使用自定义表达式计算，且提取路径中的值，并作为参数传给计算函数
    .antMatchers("/api/user/{userId}")
    .access("@myExpressionInterceptor.eval2(authentication,request,#userId)");
```

对于自定义的表达式计算器，只需要按需要定义表达式计算类

**MyExpressionInterceptor.class**

```
@Component
public class MyExpressionInterceptor {

    public boolean eval1(Authentication authentication, HttpServletRequest request) {
        // 自定义逻辑
        return true;
    }

    public boolean eval2(Authentication authentication, HttpServletRequest request, String id) {
        // 自定义逻辑
        return true;
    }
}
```

`WebExpressionVoter`强大的功能，加上可以自定义表达式计算，已经基本上可以解决大部分的场景了，一般情况下是没有新增其他的投票器的必要了。

### （Ⅳ）使用注解

在Spring Security中，是可以使用注解的，只需要开启注解即可

```
@EnableWebSecurity
// 允许使用注解
@EnableGlobalMethodSecurity(securedEnabled = true, prePostEnabled = true)
public class SecurityConfigure extends WebSecurityConfigurerAdapter {

...
}
```

这样，就可以在方法或者类中使用`@Secured`和`@PreAuthorize`，如下所示

```
public interface BankService {

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account readAccount(Long id);

@Secured("IS_AUTHENTICATED_ANONYMOUSLY")
public Account[] findAccounts();

@Secured("ROLE_TELLER")
public Account post(Account account, double amount);
}
```

```
public interface BankService {

@PreAuthorize("isAnonymous()")
public Account readAccount(Long id);

@PreAuthorize("isAnonymous()")
public Account[] findAccounts();

@PreAuthorize("hasAuthority('ROLE_TELLER')")
public Account post(Account account, double amount);
}
```




## （六） Spring Security拥有的其他特性

### （Ⅰ）CSRF安全

#### 什么是CSRF攻击

浏览器对于向同一个网站的请求，会保持请求头封装的Cookie字段内容的一致性。服务器端权限认证大都基于此（即使后端采用的是Session——Session实际上就是请求头Cookie字段内的jsession字段），如果当前Cookie所代表的用户已经登录则不再需要进行登录，否则会被要求进行登录。浏览器对Cookie的处理，大大方便了前端的开发。正常的登录业务下，这是没有什么问题的，但如果涉及到转账等其他敏感业务，没有进行其他的安全策略包护，就会存在安全性问题。

假设存在这样的一个情境，网站A是恶意网站，网站B是一个支付网站，且支付所需的校验仅仅只是通过session判断用户是否登录。某个用户U登录了网站B，并在登录期间又浏览了恶意网站A，恶意网站A上有一个按钮，封装了向网站A的账户转账的请求参数。用户U点击了这个按钮，浏览器页面会从恶意网站A跳转到网站B执行转账的业务，由于网站B已经登录，浏览器发送给网站B服务器的Session被检测为已登录，所以网站B执行了这个转账操作。在这个案例中，转账显然不是用户的意愿，而是被恶意网站劫持所导致，更要命的是，恶意网站A实际上可以连这个按钮都不需要，只要在后台运行一段JavaScript脚本就能达到目的。

上面的案例清楚的解释了什么是CSRF攻击，这个攻击主要依赖的便是流量器对Cookie的自动封装，试图获取用户正在浏览的信息恶意攻击者无法利用它，但一些不关心用户信息只关心操作结果的恶意攻击，确是可以利用它的。

#### 对于CSRF攻击的防范

防范CSRF攻击的关键就在于，对于安全级别高的操作，后端不能仅依赖浏览器自动封装的字段进行校验。可以生成token令牌发送给前端，前端提交表单时必须包含给定的csrf token令牌。

Spring Security默认是开启csrf token保护的，对于依赖模板引擎的前后端不分离的页面，Spring Security会在给前端的登录页面表单上自动设置包含名为_csrf的input标签，也可以在模板引擎中手动设置像下面input标签这样的存储csrf token参数的位置。

```
<input type="hidden" name="${_csrf.parameterName}" value="${_csrf.token}"/>
```

在前端页面head的meta位置设置csrf字段也是可行的，浏览器会在请求时自动将meta中的字段和值封装到http head中，与Cookie不一样的是，meta是必须当前页面发起的请求才会有的，所以不会有Cookie那样的问题。

```
<html>
<head>
    <meta name="_csrf" content="${_csrf.token}"/>
    <!-- default header name is X-CSRF-TOKEN -->
    <meta name="_csrf_header" content="${_csrf.headerName}"/>
    <!-- ... -->
</head>
<!-- ... -->
```

实际上真正需要csrf验证的接口并不多，而且在所有页面开启CSRF的情况下，类似于从其他网站跳转到该网站的跨平台合作性业务受限。如果不需要Security的CSRF实现，可以在configure中通过如下代码关闭csrf，如第二节案例（Ⅰ）所示。类似做验证码校验一样，自定义csrf的实现，实际上是很简单的。

```
http.csrf().disable();
```



### （Ⅱ）OAUTH

后面补上


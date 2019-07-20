---
layout: post
title: Spring Security 那些事
categories: [Tech]
tags: [Spring]
description: 来看看 Spring Security 的认证和授权都是怎么玩的。
---

> Spring Security is a framework that focuses on providing both authentication and authorization to Java applications. 

根据官方文档定义，Spring Security 是一个专注于解决 Java 应用程序中认证和授权问题的框架。

#### 认证 VS 授权

在开始介绍 Spring Security 之前，让我们先理清一下认证(Authentication) 与授权(Authorization)的区别。

认证主要为了解决我是谁的问题，通过提供证据证明你是你说的那个人。比如说你到酒店入住，你说你是张三，那前台怎么确认你确实是张三而不是冒名顶替的张三呢？请出示一下身份证，前台确认证件上的名字确实是张三，在公安系统里验证身份证真伪并进行人脸识别，如果通过验证就认为你确实是张三。其实这就是一个现实中的认证过程。通过提供身份证这种很重要，通常只有当事人持有的证件来达到证明自己的目的。在互联网上，姓名和身份证通常被抽象为用户名和密码。用户通过提供用户名对应的正确的密码来证明自己就是该用户。

当然，这是基于密码只有用户本人知道的假设前提。一旦发生密码泄露，比如用户 B 的密码被用户 A 知道，就可能发生用户 A 冒充用户 B 登陆进入系统的情况。针对这种情况，可以通过配合手机动态码或邮箱的方式来进行双因素验证。不过这些都是具体的安全认证方案了，我们这里不做过多深入。

授权主要是为了解决我能干什么的问题。其是在认证之后，已经知道了你是谁，根据你的角色（通常会在认证结束时一并返回）和你所拥有的资源，赋予你能做某些事情的权限。在授权时，根据其赋予权限时使用的依据，又可将授权分为基于角色的权限管理和基于数据的权限管理。

#### 基于角色的权限管理和基于数据的权限管理

顾名思义，基于用户角色的权限管理就是根据不同的用户角色，赋予不同的操作权限。比如对于一个购物商城应用，管理员能查看和修改商品的详情，购物者只能浏览商品详情。这种按角色为维度进行划分，纵深的对每种角色进行权限管理的方式，我们也称其为垂直权限管理。

现在考虑另外一个场景，购物者具有查看自己地址的权限，用户 A 和用户 B都是购物者，如果只有基于用户角色的权限管理，那么用户 A 也能看到用户 B 的地址，这显然是不合理的。这种希望基于角色管理的基础上，增加对自己可见资源的权限约束的细化管理我们称之为水平权限管理。由于这部分细化管理通常是涉及不同用户所拥有数据或资源的管理，我们也称其为基于数据的权限管理。

目前 Spring Security 提供了两种授权管理方式： 基于 URL 的访问控制和基于 method 的访问控制，但其本质都是基于角色的权限管理。基于数据的权限管理，由于其与业务紧耦合，需要具体情况具体对待，故目前市面上还没有主流的框架自然支持基于数据的权限管理。

#### Spring Security 的使用

在深入 Spring Security 实现细节之前，我们先看看 Spring Security 可以怎么用。

####最简单的 Spring Security 样例 

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {
    
	@Override
	protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication()
				.withUser("zhangsan") // 创建用户 zhangsan, 密码为 zhangsan, 角色为 ADMIN
				.password("zhangsan")
				.roles("ADMIN")
				.and()
				.passwordEncoder(NoOpPasswordEncoder.getInstance()); // 密码为明文存储
	}

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
		  .authorizeRequests()
		  .antMatchers("/login").permitAll()  // 设置任何人都可以访问 /login url
          .antMatchers("/hello").hasRole("ADMIN") // 只有拥有 ADMIN 权限的用户能访问 /hello url
		  .anyRequest().authenticated() // 其余所有路径都要求用户登陆
		  .and()
		  .httpBasic();  // 使用内置的 Basic 认证方式
	}
}
```

以上是一个非常简单的 Spring Security 配置。其中有一个内存用户 zhangsan, 拥有角色 ADMIN，密码为明文存储的 zhangsan。对于该系统，认证时采用 Basic 认证的方式。任何人都能访问 /login 路径，只有拥有 ADMIN 权限的用户能访问 /hello 路径，其余所有路径都要求用户登陆后才能访问，否则会抛出403 forbidden 错误。

一旦 Spring Security 配置成功，就可以在业务代码中获取到当前登陆到用户，根据用户信息进行相关的业务处理。下面的例子简单的获取到当前登陆到用户并输出其姓名和第一个角色。

```java
@GetMapping("/hello")
public String sayHello() {
	final UserDetails user = (UserDetails) SecurityContextHolder.getContext().getAuthentication().getPrincipal();
	return "hello!" + user.getUsername() + ".You are a " + user.getAuthorities().iterator().next();
}
```

看了这个简单的例子，下面我们来看看 Spring Security 提供的两种访问控制方式要怎么使用。

##### 基于 URL 的访问控制

通过配合 WebSecurityConfigurerAdapter 和 @EnableWebSecurity 实现基于 URL 的访问控制。

上面的简单例子就是使用的基于 URL 的访问控制，配置中重写了两个 configure方法。 WebSecurityConfigurerAdapter 一共有三个 configure 方法，用于配置认证和 url 授权相关的内容。其方法签名如下：

```java
protected void configure(HttpSecurity http) throws Exception;
protected void configure(WebSecurity web) throws Exception;
protected void configure(AuthenticationManagerBuilder auth) throws Exception
```

* void configure(HttpSecurity http)  ：主要用于配置基于 http 请求的安全检查，基于 URL 的访问控制规则通常在这里定义。

  默认配置为

  ```java
  http.authorizeRequests().anyRequest().authenticated().and().formLogin().and().httpBasic();
  ```

  如果没有重写该方法，系统会默认使用 basic 认证的方式，提供一个 /login 的登陆界面，任何除 /login 的路径外的请求都要求登陆后访问。

  

  对具体 url 进行访问控制可以通过如下代码实现，主要是通过组合 url 和角色设置访问规则。

  ```java
  http.authorizeRequests().antMatchers("/hello", "/user").hasRole("ADMIN");
  http.authorizeRequests().antMatchers("/hello", "/user").hasAuthority("ROLE_ADMIN");
  ```

  

  指定 url, 主要使用 antMatchers。

  ```java
  antMatchers("/hello")    // 支持一次传入一个路径
  antMatchers("/hello", "/user")  // 支持一次传入多个路径
  antMatchers(HttpMethod.GET, "/hello")  // 支持指定 HttpMethod
  ```

  antMatcher 也支持模糊匹配，规则为：

  ​	?  匹配一个字符

  ​	\*  匹配 0 或多个字符  
  ​	** 匹配 0 或多层路径

  

  指定角色，可以使用 hasRole, hasAnyRole, hasAuthority, hasAnyAuthority 等方法。

  hasRole/hasAnyRole vs hasAuthority/hasAnyAuthority

  使用 hasRole 或 hasAnyRole 时，如果传入的 String 不是以 ROLE_ 前缀开头，会自动在前面添加 ROLE_ 前缀，hasAuthority 或 hasAnyAuthority 则不会对传入的字符串做处理。

  eg. 用户角色为 ROLE_ADMIN， 则可以使用 hasRole("ADMIN") 或 hasAuthority("ROLE_ADMIN") 匹配角色 ROLE_ADMIN。

  Tips: 为了保证 hasRole/hasAuthority 都能使用，以及基于 method 的检查中 role 相关的方法能正常工作，建议定义用户角色的时候总是以 ROLE_ 前缀开始。

* void configure(WebSecurity web)  ：主要用于配置基于网站全局的安全检查，如设置自定义的防火墙，设置 debug 模式，忽略资源文件等。

* void configure(AuthenticationManagerBuilder auth) ： 主要用于用户认证阶段的配置，通过自定义 UserDetailService 及 AuthenticationProvider 等自定义认证机制。

  UserDetailService 和 AuthenticationProvider 主要承担的职责会在接下来的部分具体介绍。

  上面的例子中定义系统中存储密码时不进行编码，使用 NoOpPasswordEncoder， 这是非常不安全的。NoOpPasswordEncoder 已经被标记为  deprecated, 平时主要用于测试中。实际使用时，官方推荐使用 BCryptPasswordEncoder。Spring Security 内置了 bcrypt, ldap, md5, sha256 等常用的加密方式，完整的支持方式可在 PasswordEncoderFactories 中找到。

  重写该 configure 方法定义的 AuthenticationManager 只对当前 adapter 生效。也可以通过下面方法定义一个全局的 AuthenticationManager。

  ```java
  	@Autowired
  	public void globalConfig(AuthenticationManagerBuilder auth) throws Exception {
  		auth.userDetailsService(userDetailsService)
  				.passwordEncoder(new BCryptPasswordEncoder());
  	}
  
  	@Bean
  	@Override
  	public AuthenticationManager authenticationManagerBean() throws Exception {
  		return super.authenticationManagerBean();
  	}
  ```

##### 基本 method 的访问控制

使用 GlobalMethodSecurityConfiguration 配合 @EnableGlobalMethodSecurity 实现

 ```java
@EnableGlobalMethodSecurity(
		prePostEnabled = true,
		securedEnabled = true,
		jsr250Enabled = true)
public class MethodSecurityConfig extends GlobalMethodSecurityConfiguration {
}
 ```

由于基类已经包含 @Configuration 注释，继承该类后会自动作为配置加入的 Spring 容器中。

有了以上的配置，就可以使用基于 method 的访问控制注解了。

```java
@Secured
@RolesAllowed
@PreAuthorize
@PostAuthorize
@PreFilter
@PostFilter
```

EnableGlobalMethodSecurity 中的三个变量控制了上述 6 个方法是否可用，默认都为 false 的不可用状态。securedEnabled 控制 @Secured， jsr250Enabled 控制 @RolesAllowed，余下 4 个方法均由 prePostEnabled 控制。

* @secured

  传入值为权限的字面量。直接加在需要做角色校验的方法前即可。

  ```java
  @GetMapping("/hello-user")
  @Secured("ROLE_USER")
  // @Secured({"ROLE_USER", "ROLE_ADMIN"})  // 支持传入多个角色，只要有一个匹配即可
  public String helloUser() {
      return "hello! user";
  }
  ```

* @RolesAllowedf

  传入的字符串不是以ROLE\_开头，会加上 ROLE\_ 前缀，如果以 ROLE_ 开头则使用传入值。

* @PreAuthorize & @PostAuthorize 

  从 Spring Security 3.0 开始支持 SpEL(Spring Expression Language)。SpEL 支持在运行时查询和操作对象，其有一些内置的方法，如 hasRole, hasPermission 等。

  ```java
  @GetMapping("/hello-user")
  @PreAuthorize("hasRole('USER')")
  // @PreAuthorize(“hasRole('USER')  or hasRole('ADMIN')“) // 多个角色可用用 or 连接
  public String helloUser() {
  	return "hello! user";
  }
  
  ```

  同时可将方法参数作为表达式的一部分，使用 # 加参数名引用参数。

  ```java
  @PreAuthorize("#username == authentication.principal.username")
  public String getMyRoles(String username) {
  	//...
  }
  ```

* @PreFilter & @PostFilter 

  对输入/输出集合参数进行过滤。要求集合必须要有 remove 方法，如参数为数组类型或运行时集合为空时都会报错。

  ```java
  @PreFilter(value = "filterObject != authentication.principal.username",
    filterTarget = "usernames")   // 有多个参数时需指定参数名
  public String joinUsernamesAndRoles(List<String> usernames, List<String> roles) {
      return usernames.stream().collect(Collectors.joining(";")) 
        + ":" + roles.stream().collect(Collectors.joining(";"));
  }
  ```

### Spring Security 的核心元素

UserDetails: 用户对象的抽象接口，包含用户的基本信息。接口如下：

```java
public interface UserDetails extends Serializable {
	Collection<? extends GrantedAuthority> getAuthorities();  
	String getPassword();
	String getUsername();
	boolean isAccountNonExpired();
	boolean isAccountNonLocked();
	boolean isCredentialsNonExpired();
	boolean isEnabled();
}
```

UserDetailService：用于获取 UserDetails 对象，类似于 UserDetails 的 repository。

GrantedAuthority： 被授予权限的抽象接口，通常用字符串表示。

Authentication：认证信息的载体。主要有两种状态，存储认证前的认证请求和认证后的认证结果。最常用的实现类是 UsernamePasswordAuthenticationToken。

AuthenticationManager：执行具体认证逻辑的抽象接口类，其具体实现类为 ProviderManager。ProviderManager 包含一系列的 AuthenticationProvider，认证时会遍历这些 provider, 找到能处理传入的 Authentication 的 provider，调用其 authentication 方法做认证。

SecurityContextHolder: 用户持有安全上下文，比如当前登陆的用户信息等。

AuthenticationProvider： 处理具体 Authentication 认证对象的抽象类。通常一个 provider 对应一种认证方式。ProviderManager 中可以包含多个 AuthenticationProvider 表示系统可以支持多种认证方式。

### Spring Security 认证流程

![security authentication](https://git-page.oss-cn-chengdu.aliyuncs.com/spring-security/authentication.png)

用户想要登陆时，首先需要输入用户名和密码。这些信息被封装为一个 Authentication 对象, 通常是一个UsernamePasswordAuthenticationToken。 这个 Authentication 对象会被提交给负责认证的 AuthenticationManager。AuthenticationManager 有一个 authenticate 方法，方法签名如下：

```java
Authentication authenticate(Authentication authentication) throws AuthenticationException; 
```

这里传入的是认证前表示请求的 Authentication 对象，返回的是认证成功后包含用户信息且擦除了用户密码等敏感信息的 Authentication。

如果认证成功，返回的 Authentication 对象会被放入 SecurityContextHolder 中。 SecurityContextHolder是 ThreadLocal 的，后续可以通过
SecurityContextHolder.getContext().getAuthentication() 的方式获得放入的 Authentication 对对象。如果认证失败，会交给 AuthenticationFailureHandler 处理。

AuthenticationManager 在执行 authenticate 方法时，会遍历 AuthenticationProvider，找到一个能处理当前传入 Authentication 的  provider 并使用其进行处理，如找不到则抛出 AuthenticationException。

provider 做认证时，会首先调用  UserDetailService, 根据用户名获得 UserDetails 对象，这个 UserDetails 对象包含了其拥有的权限和加密过的密码。随后 provider 会比较 UserDetails.getPassword() 与 Authentication.getCridentials()。这里 UserDetail 的 password  是存储在系统中的用户的正确密码，Authentication 的 credential 是用户输入的密码。当然，在比较密码前后，provider 还会做一些常规校验，比如校验用户是否是 enable的，账户是否被锁定，账号是否过期等，详情请参见 AbstractUserDetailsAuthenticationProvider 的 authentication 方法。如果密码正确，则认为认证成功，会将 UserDetails 放入 Authentication 中作为 principal 返回。

认证成功后，此时登陆用户信息已经放入 SecurityContextHolder 中，后续会交由 AbstractSecurityInterceptor 进行处理。访问控制，也就是授权，主要由 AccessDecisionManager 的 decide 方法来判断。所有基于 url  和 method 的访问规则都会被转换成 ConfigAttribute, decide 方法会根据这些 ConfigAttribute, 结合用户的权限，判断其是否能访问该方法。

### Spring Security Test

添加了访问控制后，如何绕过用户登陆对应用作测试呢？ Spring Security 提供了对测试的支持。

如果需要基于 Spring Security 做测试，需要使用 SpringRunner 并添加 @ContextConfiguration。

```java
@RunWith(SpringRunner.class)
@ContextConfiguration
public class HelloControllerTest {

}
```

Spring Security Test 提供了三种注入用户的方法。

1. @WithMockUser

   ```java
   @Test
   @WithMockUser(username = "zhangsan", roles = "ADMIN")
   public void testUser() {
       // …
   }
   ```

   其中的 roles 在比较时会自动添加 ROLE_ 前缀。使用 @WithMockUser， 用户可以不必存在在数据库中。

2. @WithUserDetails

    ```java
   @Test
   @WithUserDetails(value = "zhangsan")
   public void testUser() {
       // …
   }
   ```

   使用该注释，只需要给用户名就行。用户必须存在在数据库中, 需要与 @SpringBootTest 注解配套使用，因为该方法需要注入 UserDetailService, 实际调用 UserDetailService 来获得用户。

3. @WithAnonymousUser

   ```java
   @Test
   @WithAnonymousUser
   public void testUser() {
       // …
   }
   ```

   放入 SecurityContextHolder 中的 Authentication 对象是一个 annoymous 字符串。

此外，为了避免在多个方法上 mock 同一个用户，可以使用 meta annotation。这样方便对 mock 的用户进行统一管理。

```java
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(value="zhangsan",roles="ADMIN")
public @interface WithMockAdmin { }
```

#### 小结

Spring Security 提供了完整的认证及基于角色的访问控制，同时支持很多第三方的认证方式。本文只简单介绍了最基本的使用方法及认证时的一个大概流程。更多有趣的内容参见官方文档和源码。
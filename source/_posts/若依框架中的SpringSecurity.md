---
title: 若依框架中的SpringSecurity
date: 2023/9/25 13:17
tags: Spring
categories: Spring
top_img: ./img/41.jpg
cover: ./img/66.jpg
---

# 若依框架中的SpringSecurity



## 1.pom文件中新增依赖

```xml
<!-- spring security 安全认证 -->
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```



## 2.SpringSecurity基本功能

Spring Security 是一个强大且灵活的身份验证和访问控制框架，用于Java应用程序的安全性处理。它提供了对身份验证、授权、攻击防护等方面的支持。

1. **身份验证（Authentication）**：
   - 提供用户身份验证的机制，包括基本认证、表单认证、OAuth认证等。
   - 支持用户自定义身份验证逻辑。

2. **授权（Authorization）**：
   - 定义和控制用户对应用程序资源的访问权限。
   - 支持基于角色、权限、表达式等的访问控制。

3. **攻击防护**：
   - 防止常见的安全攻击，如CSRF（跨站请求伪造）、XSS（跨站脚本攻击）、Session Fixation等。

4. **用户管理**：
   - 支持用户的注册、登录、密码重置等操作。



## 3.ruoyi框架中的SpringSecurity配置类

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.crypto.password.NoOpPasswordEncoder;
import org.springframework.security.crypto.password.PasswordEncoder;

@EnableGlobalMethodSecurity(prePostEnabled = true,securedEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
	
    //自定义用户认证逻辑
    @Autowired
    private UserDetailsService userDetailsService;
    
    //认证失败处理类
    @Autowired
    private AuthenticationEntryPointImpl unauthorizedHandler;
    
    //退出处理类
    @Autowired
    private LogoutSuccessHandlerImpl logoutSuccessHandler;
    
    //token认证过滤器
    @Autowired
    private JwtAuthenticationTokenFilter authenticationTokenFilter;
    
    //跨域过滤器
    @Autowired
    private CorsFilter corsFilter;
    
    //解决无法直接注入 AuthenticationManager
    @Bean
    @Override
    public AuthenticationManager authenticationManagerBean() throws Exception{
        return super.authenticationManagerBean();
    }
    
    /**
    * anyRequest        |   匹配所有请求路径
    * access            |   SpringEl表达式结果为true时可以访问
    * anonymous         |   匿名可以访问
    * denyAll           |   用户不能访问
    * fullyAuthenticated|   用户完全认证可以访问（非remember-me下自动登录）
    * hasAnyAuthority   |   如果有参数，参数表示权限，则其中任何一个权限可以访问
    * hasAnyRole        |   如果有参数, 参数表示角色，则其中任何一个角色可以访问
    * hasAuthority      |   如果有参数，参数标识权限，则其权限可以访问
    * hasIpAddress      |   如果有参数，参数标识IP地址，如果用户IP和参数匹配，则可以访问
    * hasRole           |   如果有参数，参数标识角色，则其角色可以访问
    * permitAll         |   用户可以任意访问
    * rememberMe        |   允许通过remember-me登录的用户访问
    *authenticated      |   用户登录后可以访问
    */
    @Override
    protected void configure(HttpSecurity httpSecurity) throws Exception{
    	httpSecurity
    		// CSRF禁用，因为不使用session
            .csrf().disable()
            // 认证失败处理类
            .exceptionHandling().authenticationEntryPoint(unauthorizedHandler).and()
            //基于token，所以不需要session
            .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS).and()
            //过滤请求
            .authorizeRequests()
            //对于登录login 验证码captchaImage允许匿名访问
            .antMatchers("/login","/capthaImage")
            .antMatchers(HttpMethod.GET,"/*.html","/**/*.html","/**/*.css","/**/*.js").permitAll()
            .antMatchers("/profile/**").anonymous()
            .antMatchers("/common/download**").anonymous()
            .antMatchers("/common/download/resource**").anonymous()
            .antMatchers("/swagger-ui.html").anonymous()
            .antMatchers("/swagger-resources/**").anonymous()
            .antMatchers("/webjars/**").anonymous()
            .antMatchers("/*/api-docs").anonymous()
            .antMatchers("/druid/**").anonymous()
            //除上面外的所有请求全部需要鉴权认证
            .anyRequest().authenticated()
            .and()
            .headers().frameOptions().disable();
        httpSecurity.logout().logoutUrl("/logout").logoutSuccessHandler(logoutSuccessHandler);
        // 添加JWT filter
        httpSecurity.addFilterBefore(authenticationTokenFilter,UsernamePasswordAuthenticationFilter.class);
        // CORS filter
        httpSecurity.addFilterBefore(corsFilter,JwtAuthenticationTokenFilter.class);
        httpSecurity.addFilterBefore(corsFilter,LogoutFilter.class);
    	
    }

    //强散列哈希加密实现
    @Bean
    public BCryptPasswordEncoder bCryptPasswordEncoder(){
        return new BCryptPasswordEncoder();
    }
    
    //身份认证接口
    @Overide
    protected void configure(AuthenticationManagerBuilder auth) throws Exception{
        auth.userDetailsService(userDetailsService).passwordEncoder(bCryptPasswordEncoder());
    }
}
```

### 分析代码：



#### 该配置类继承WebSecurityConfigurerAdapter，有两个核心覆写



#### 1.身份认证接口，configure(AuthenticationManagerBuilder auth)



#### 2.授权接口，configure(HttpSecurity httpSecurity)



#### 3.攻击防护， .csrf().disable()

##### 禁用了CSRF，因为不使用session

CSRF（Cross-Site Request Forgery，跨站请求伪造）和Session（会话）是Web应用程序安全的两个不同方面，但它们可以在某些情况下互相关联。

##### CSRF（跨站请求伪造）：
CSRF攻击是一种利用受信任用户的身份在用户不知情的情况下执行非预期操作的攻击方式。攻击者诱导用户点击特定链接或提交恶意请求，以执行可能对应用程序有害的操作。CSRF攻击可以通过伪装用户请求的来源来完成。

##### Session（会话）：
Session用于在服务器端跟踪用户的状态，通常通过cookie或URL重写来标识用户。会话通常用于保持用户登录状态和存储与用户相关的信息。

##### 关系和禁用CSRF的原因：
1. **关系**：在防止CSRF攻击时，常用的一种机制是将CSRF令牌（CSRF token）包含在表单中。这个令牌通常存储在用户的会话中，并与每个表单一起发送。这样，服务器可以验证请求是否合法，从而防止CSRF攻击。

2. **禁用CSRF的原因**：如果不使用Session来存储CSRF令牌，可以选择在每次请求时都生成新的CSRF令牌。这种情况下，禁用CSRF保护可能会使应用程序更易受到CSRF攻击，因为没有一种方法来验证请求的合法性，即使请求不来自受信任的来源。

禁用CSRF保护时，通常需要确保其他安全措施足够强大，如使用适当的权限和身份验证机制，以确保应用程序不容易受到其他攻击，如未经授权的访问。然而，禁用CSRF保护不是推荐的做法，除非在特定情况下有严格的安全控制措施。

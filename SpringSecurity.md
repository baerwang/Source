# Spring Security

## 简介

Spring Security 是针对Spring项目的安全框架，也是Spring Boot底层安全模块默认的技术选型，他可以实现强大的Web安全控制，对于安全控制，我们仅需要引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理！

记住几个类：

```java
WebSecurityConfigurerAdapter:自定义Security策略
AuthenticationManagerBuilder:自定义认证策略
@EnableWebSecurity:开启WebSecurity模式
```

Spring Security的两个目标是"认证"和"授权"(访问控制)

"认证"(Authentication)	"授权"(Authorization)

这个概念是通用的，而不是只在Spring Security中存在

## 配置 Spring Security

@EnableWebSecurity注解是个组合注解，他的注解中，又使用了@EnableGlobalAuthentication注解

@EnableWebSecurity这个作用激活了WebSecurityConfiguration配置类，在这个配置类中，注入了一个非常重要的Bean，Bean的name为:SpringSecurityFilterChain，这是Spring Security的核心过滤器，这是请求的认证入口。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({WebSecurityConfiguration.class, SpringWebMvcImportSelector.class, OAuth2ImportSelector.class})
@EnableGlobalAuthentication
@Configuration
public @interface EnableWebSecurity {
    boolean debug() default false;
}
```



@EnableGlobalAuthentication这个作用是激活了AuthenticationConfiguration配置类，这个类是来配置认证相关的核心类，这个类的主要作用是，向Spring容器注入AuthenticationManagerBuilder，AuthentiationManagerBuilder其实是使用了建造者模式，他能建造AuthenticationManager，是身份认证的入口。

```java
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({AuthenticationConfiguration.class})
@Configuration
public @interface EnableGlobalAuthentication {
}
```

@EnableWebSecurity注解有两个作用

1. 加载了WebSecurityConfiguration配置类，配置安全认证策略。
2. 加载了AuthenticationConfiguration，配置了认证信息

新建个Security配置类

```java
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    /**
     * 授权
     *
     * @param http
     * @throws Exception
     */
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests()
                /*首页所有人可以访问*/
                .antMatchers("/").permitAll()
                /*功能页只有应对权限的人才能访问*/
                .antMatchers("/level1/**").hasRole("vip1")
                .antMatchers("/level2/**").hasRole("vip2")
                .antMatchers("/level3/**").hasRole("vip3");

        /*没有权限默认会到登录页面,默认Login(/login)
        http.formLogin();*/

        /*loginPage("/toLogin")定制登录页面
        http.formLogin().loginPage("/toLogin").loginProcessingUrl("/usr/login");*/

        /*loginProcessingUrl定义登录action路径
        http.formLogin().loginPage("/toLogin").loginProcessingUrl("/usr/login");*/

        /*usernameParameter:设置html,输入框input name的属性，默认为username,
        passwordParameter:输入框name的属性,默认为password*/
    	http.formLogin().loginPage("/toLogin").usernameParameter("name")
       		 .passwordParameter("pwd").loginProcessingUrl("/usr/login");

        /*防止网站工具，get,post,关闭csrf功能，登录失败存在的原因*/
        //http.csrf().disable();

        /*注销,注销后重定向到指定的页面(默认为/login)*/
        http.logout().logoutSuccessUrl("/");

        /*开启记住我功能，cookie默认保存两周，自动显示checkbox复选框
        http.rememberMe();*/

        /*自定义接收前端的参数,开启记住我*/
        http.rememberMe().rememberMeParameter("remember");
    }

    /**
     * 认证
     *
     * @param auth
     * @throws Exception
     */
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
        /*passwordEncoder设置密码编码，用户当前有vip1权限*/
        auth.inMemoryAuthentication().passwordEncoder(new BCryptPasswordEncoder()).
                /*设置user,password和role*/
       withUser("admin").password(new BCryptPasswordEncoder().encode("admin")).roles("vip1")
                /*用户当前有vip1,2,3权限*/
      .and().withUser("root").password(new BCryptPasswordEncoder().encode("666")).roles("vip1", "vip2", "vip3");
    }
}
```

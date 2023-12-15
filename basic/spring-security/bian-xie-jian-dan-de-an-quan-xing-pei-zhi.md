# 编写简单的安全性配置

如下的程序清单展现了 Spring Security 最简单的 Java 配置：

<details>

<summary><mark style="color:purple;">启用 Web 安全性功能的最简单配置</mark></summary>

{% code overflow="wrap" lineNumbers="true" %}
```java
package spitter.config;

import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.servlet.configuration.EnableWebSecurity;

@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
}
```
{% endcode %}

</details>

<mark style="color:blue;">**@EnableWebSecurity**</mark> 注解将会启用 Web 安全功能。

但它本身并没有什么用处，**Spring Security 必须配置在一个实现了 **<mark style="color:blue;">**WebSecurityConfigurer**</mark>** 的 bean 中，或者（简单起见）扩展 **<mark style="color:blue;">**WebSecurityConfigurerAdapter**</mark>**。**

可以通过重载 WebSecurityConfigurerAdapter 的三个 configure() 方法来配置 Web 安全性:

* <mark style="color:blue;">**configure(WebSecurity)**</mark>**：**配置 Spring Security 的 Filter 链
* <mark style="color:blue;">**configure(HttpSecurity)**</mark>：配置如何通过拦截器保护请求
* <mark style="color:blue;">**configure(AuthenticationManagerBuilder)**</mark>：配置 user-detail 服务

> 默认的 configure(HttpSecurity) 等同于：
>
> ```java
> protected void configure(HttpSecurity http) throws Exception {
>   http
>     .authorizeRequests()
>     .anyRequest().authorized()
>     .and()
>     .formLogin()
>     .and()
>     .httpBasic();
> }
> ```
>
> * 这个简单的默认配置指定了该如何保护 HTTP 请求，以及客户端认证用户的方案。通过调用 **authorizeRequests**() 和 **anyRequest**().**authenticated**() 要求所有进入应用的 HTTP 请求都要进行认证。
> * 它也配置 Spring Security 支持基于表单的登录以及 HTTP Basic 方式的认证。

> 因为没有重载 **configure(AuthenticationManagerBuilder)** 方法，所以没有用户存储支撑认证过程。
>
> 没有用户存储，实际上就等于没有用户。所以，在这里所有的请求都需要认证，但是没有人能够登录成功。

{% hint style="info" %}
## @EnableWebMvcSecurity

**@EnableWebSecurity 可以启用任意 Web 应用的安全性功能，不过，**<mark style="color:blue;">**如果你的应用是使用 Spring MVC 开发的，那么就应该考虑使用 @EnableWebMvcSecurity 替代它**</mark>

* <mark style="color:blue;">**@EnableWebMvcSecurity 注解配置了一个 Spring MVC 参数解析解析器（argument resolver），处理器方法能够通过带有 @AuthenticationPrincipal 注解的参数获得认证用户的 principal（或username）。**</mark>
* 它同时还配置了一个 bean， 在使用 Spring 表单绑定标签库来定义表单时，这个 bean 会自动添加一个隐藏的跨站请求伪造（cross-site request forgery，CSRF）token 输入域。
{% endhint %}

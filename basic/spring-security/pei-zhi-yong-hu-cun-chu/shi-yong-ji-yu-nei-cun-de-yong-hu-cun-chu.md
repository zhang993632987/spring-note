# 使用基于内存的用户存储

因为我们的安全配置类扩展了 **WebSecurityConfigurerAdapter**，因此**配置用户存储的最简单方式就是重载 configure() 方法，并以 AuthenticationManagerBuilder 作为传入参数。**

AuthenticationManagerBuilder 有多个方法可以用来配置 Spring Security 对认证的支持。通过 inMemoryAuthentication() 方法，我们可以启用、配置并任意填充基于内存的用户存储。

<details>

<summary><mark style="color:purple;">配置 Spring Security 使用内存用户存储</mark></summary>

{% code overflow="wrap" lineNumbers="true" %}
```java
package spittr.config;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.authentication.builders.AuthenticationManagerBuilder;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
import org.springframework.security.config.annotation.web.servlet.configuration.EnableWebMvcSecurity;

@Configuration
@EnableWebMvcSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
  
  @Override
  protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth
      .inMemoryAuthentication()
      .withUser("user").password("password").roles("USER");
      .withUser("admin").password("password").roles("USER", "ADMIN");
  }
}
```
{% endcode %}

</details>

在上述程序清单中，SecurityConfig 重载了 configure() 方法，并使用两个用户来配置内存用户存储。

> 需要注意的是：
>
> * roles() 方法是 authorities() 方法的简写形式。
> * **roles() 方法所给定的值都会添加一个“ROLE\_”前缀**，并将其作为权限授予给用户。

配置用户详细信息的方法如下表：

<table><thead><tr><th width="277">方法</th><th>描述</th></tr></thead><tbody><tr><td>accountExpired(boolean)</td><td>定义账号是否已经过期</td></tr><tr><td>accountLocked(boolean)</td><td>定义账号是否已经锁定</td></tr><tr><td>and()</td><td>用来连接配置</td></tr><tr><td>authorities(GrantedAuthority...)</td><td>授予某个用户一项或多项权限</td></tr><tr><td>authorities(List&#x3C;? extends GrantedAuthority>)</td><td>授予某个用户一项或多项权限</td></tr><tr><td>authorities(String...)</td><td>授予某个用户一项或多项权限</td></tr><tr><td>credentialsExpired(boolean)</td><td>定义凭证是否已经过期</td></tr><tr><td>disabled(boolean)</td><td>定义账号是否已被禁用</td></tr><tr><td>password(String)</td><td>定义用户的密码</td></tr><tr><td>roles(String...)</td><td>授予某个用户一项或多项角色</td></tr></tbody></table>

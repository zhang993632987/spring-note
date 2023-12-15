# 拦截请求

<mark style="color:blue;">**对每个请求进行细粒度安全性控制的关键在于重载 configure(HttpSecurity) 方法。**</mark>如下的代码片段展现了重载的 configure(HttpSecurity) 方法，它为不同的 URL 路径有选择地应用安全性：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
    .antMatchers("/spitters/me").authenticated()
    .antMatchers(HttpMethod.POST, "/spittles").authenticated()
    .anyRequest().permitAll();
}
```

在这里，我们**首先调用 authorizeRequests()，然后调用该方法所返回的对象的方法来配置请求级别的安全性细节**。其中，

* 第一次调用 antMatchers() 指定了对 /spitters/me 路径的请求需要进行认证。
* 第二次调用 antMatchers() 更为具体，说明对 /spittles 路径的 HTTP POST 请求必须要经过认证。
* 最后对 anyRequests() 的调用中，说明其他所有的请求都是允许的，不需要认证和任何的权限。

> antMatchers() 方法中设定的路径支持 Ant 风格的通配符。如下所示：
>
> ```java
> .antMatchers("/spitter/**").authenticated();
> ```
>
> 也可以在一个对 antMatchers() 方法的调用中指定多个路径：
>
> ```java
> .antMatchers("/spitter/**", "/spittles/mine").authenticated();
> ```

{% hint style="warning" %}
## <mark style="color:orange;">注意：</mark>

可以将任意数量的 antMatchers()、regexMatchers() 和 anyRequest() 连接起来，以满足 Web 应用安全规则的需要。**这些规则会按照给定的顺序发挥作用**。

所以，<mark style="color:orange;">**很重要的一点就是将最为具体的请求路径放在前面，而最不具体的路径（如 anyRequest()）放在最后面。如果不这样做的话，那不具体的路径配置将会覆盖掉更为具体的路径配置。**</mark>
{% endhint %}

**用来定义如何保护路径的配置方法：**

| 方法                         | 能够做什么                                    |
| -------------------------- | ---------------------------------------- |
| access(String)             | 如果给定的 SpEL 表达式计算结果为 true，就允许访问           |
| anonymous()                | 允许匿名用户访问 authenticated() 允许认证过的用户访问      |
| denyAll()                  | 无条件拒绝所有访问                                |
| fullyAuthenticated()       | 如果用户是完整认证的话（不是通过Remember-me 功能认证的），就允许访问 |
| hasAnyAuthority(String...) | 如果用户具备给定权限中的某一个的话，就允许访问                  |
| hasAnyRole(String...)      | 如果用户具备给定角色中的某一个的话，就允许访问                  |
| hasAuthority(String)       | 如果用户具备给定权限的话，就允许访问                       |
| hasIpAddress(String)       | 如果请求来自给定 IP 地址的话，就允许访问                   |
| hasRole(String)            | 如果用户具备给定角色的话，就允许访问                       |
|  not()                     | 对其他访问方法的结果求反                             |
| permitAll()                | 无条件允许访问                                  |
| rememberMe()               | 如果用户是通过 Remember-me 功能认证的，就允许访问          |

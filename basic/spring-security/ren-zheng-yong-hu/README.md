# 认证用户

如果你使用 [#qi-yong-web-an-quan-xing-gong-neng-de-zui-jian-dan-pei-zhi](../bian-xie-jian-dan-de-an-quan-xing-pei-zhi.md#qi-yong-web-an-quan-xing-gong-neng-de-zui-jian-dan-pei-zhi "mention")中最简单的 Spring Security 配置的话，那么就能无偿地得到一个登录页。实际上，**在重写 configure(HttpSecurity) 之前，我们都能使用一个简单却功能完备的登录页**。但是，**一旦重写了 configure(HttpSecurity) 方 法，就失去了这个简单的登录页面**。

把这个功能找回来所需要做的就是**在 configure(HttpSecurity) 方法中，调用 formLogin()**，如下面的程序清单所示。

如果我们访问应用的 “/login” 链接或者导航到需要认证的页面，那么将会在浏览器中展现登录页面。

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .formLogin()
    .and()
    .authorizeRequests()
    .antMatchers("/spitter/me").hasRole("SPITTER")
    .antMatchers(HttpMethod.POST, "/spittles").hasRole("SPITTER")
    .anyRequest().permitAll()
    .and()
    .requeresChannel()
    .antMatchers("/spitter/form").requiresSecure();
}
```

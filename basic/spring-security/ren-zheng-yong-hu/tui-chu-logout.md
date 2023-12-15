# 退出（logout）

<mark style="color:blue;">**退出功能是通过 Servlet 容器中的Filter实现的（默认情况下），这个 Filter 会拦截针对 “/logout” 的请求**</mark>。因此，为应用添加退出功能只需添加如下的链接即可：

```html
<a th:href="@{/logout}">Logout</a>
```

1. 当用户点击这个链接的时候，会发起对 “/logout” 的请求，这个请求会被 Spring Security 的 LogoutFilter 所处理。
2. 用户会退出应用，所有的 Remember-me token 都会被清除掉。
3. 在退出完成后，用户浏览器将会 重定向到 “/login?logout”，从而允许用户进行再次登录。

如果希望**用户被重定向到其他的页面**，如应用的首页，那么可以在 configure() 中进行如下的配置：

```java
@override
protected void configure(HttpSecurity http) throws Exception {
  http
    .formLogin()
    .loginPage("/login");
    .and()
    .logout()
    .logoutSuccessUrl("/")
    ...
}
```

在本例中，调用 **logoutSuccessUrl()** 表明**在退出成功之后，浏览器需要重定向到 “/”**。

除了 logoutSuccessUrl() 方法以外，还可以**重写默认的 LogoutFilter 拦截路径，**可以通过调用 **logoutUrl()** 方法实现这一功能：

```java
.logout()
.logoutSuccessUrl("/")
.logoutUrl("/signout");
```

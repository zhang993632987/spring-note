# 强制通道的安全性（https）

<mark style="color:blue;">**传递到 configure() 方法中的 HttpSecurity 对象有一个 requiresChannel() 方法，借助这个方法能够为各种 URL 模式声明所要求的通道**</mark><mark style="color:blue;">。</mark>

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
  http
    .authorizeRequests()
    .antMatchers("/spitter/me").hasRole("SPITTER")
    .antMatchers(HttpMethod.POST, "/spittles").hasRole("SPITTER")
    .anyRequest().permitAll()
    .and()
    .requeresChannel()
    .antMatchers("/spitter/form").requiresSecure();
}
```

不论何时，只要是对 /spitter/form 的请求，Spring Security 都视为需要安全通道（通过调用 requiresChannel() 确定的）并<mark style="color:orange;">**自动将请求重定向到 HTTPS 上**</mark>。

与之相反，有些页面并不需要通过 HTTPS 传送。例如，首页不包含任何敏感信息，因此并不需要通过 HTTPS 传送。我们可以使用 **requiresInsecure()** 代替 requiresSecure() 方法，将首页声明为始终通过 HTTP 传送：

```java
.antMatchers("/").requiresInecure();
```

如果通过 HTTPS 发送了对 “/” 的请求，Spring Security 将会把请求<mark style="color:orange;">**重定向**</mark>到不安全的 HTTP 通道上。

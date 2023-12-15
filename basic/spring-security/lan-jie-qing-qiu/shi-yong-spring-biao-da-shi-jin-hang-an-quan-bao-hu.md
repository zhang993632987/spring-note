# 使用 Spring 表达式进行安全保护

<mark style="color:blue;">**借助 access() 方法，可以将 SpEL 作为声明访问限制的一种方式**</mark>。例如，如下就是使用 SpEL 表达式来声明具有 ROLE\_SPITTER 角色才能访问 /spitter/me URL：

```java
.antMatchers("/spitter/me").access("hasRole('ROLE_SPITTER')");
```

如果当前用户被授予了给定角色的话，那 hasRole() 表达式的计算结果就为 true。

如果你想限制 /spitter/me URL 的访问，不仅需要 ROLE\_SPITTER，还需要来自指定的 IP 地址，那么我们可以按照如下的方式调用 access() 方法：

```java
.antMatchers("/spitter/me")
.access("hasRole('ROLE_SPITTER') and hasIpAddress('192.168.1.2')");
```

**下表列出了 Spring Security 支持的所有 SpEL 表达式：**

| 安全表达式                     | 计算结果                                               |
| ------------------------- | -------------------------------------------------- |
| authentication            | 用户的认证对象                                            |
| denyAll                   | 结果始终为 false                                        |
| hasAnyRole(list of roles) | 如果用户被授予了列表中任意的指定角色，结果为 true                        |
| hasRole(role)             | 如果用户被授予了指定的角色，结果为 true                             |
| hasIpAddress(IP Address)  | 如果请求来自指定 IP 的话，结果为 true                            |
| isAnonymous()             | 如果当前用户为匿名用户，结果为 true                               |
| isAuthenticated()         | 如果当前用户进行了认证的话，结果为 true                             |
| isFullyAuthenticated()    | 如果当前用户进行了完整认证的话（不是通过 Remember-me 功能进行的认证），结果为 true |
| isRememberMe()            | 如果当前用户是通过 Remember-me 自动认证的，结果为 true               |
| permitAll                 | 结果始终为true                                          |
| principal                 | 用户的principal对象                                     |

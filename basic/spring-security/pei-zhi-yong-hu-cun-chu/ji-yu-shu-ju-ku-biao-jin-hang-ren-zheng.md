# 基于数据库表进行认证

用户数据通常会存储在关系型数据库中，并通过 JDBC 进行访问。为了<mark style="color:blue;">**配置 Spring Security 使用以 JDBC 为支撑的用户存储，我们可以使用 jdbcAuthentication() 方法**</mark>，所需的最少配置如下所示：

```java
@Autowired
DataSource dataSource;

@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
    .dataSource(dataSource);
}
```

我们**必须要配置的只是一个 DataSource**，这样的话，就能访问关系型数据库了。在这里，DataSource 是通过自动装配的技巧得到的。

## **重写默认的用户查询功能**

尽管默认的最少配置能够让一切运转起来，但是它对我们的数据库模式有一些要求。**下面的代码片段来源于 Spring Security 内部，这块代码展现了当查找用户信息时所执行的 SQL 查询语句：**

<pre class="language-sql"><code class="lang-sql">public static final String DEF_USERS_BY_USERNAME_QUERY =
  "select username, password, enabled " +
<strong>  "from users " +
</strong>  "where username = ?";
public static final String DEF_AUTHORITIES_BY_USERNAME_QUERY =
  "select username, authority " +
  "from authorities " +
  "where username = ?";
public static final String DEF_GROUP_AUTHORITIES_BY_USERNAME_QUERY =
  "select g.id, g.group_name, ga.authority " +
  "from groups g, group_members gm, group_authorities ga " +
  "where gm.username = ? " +
  "and g.id = ga.group_id " +
  "and g.id = gm.group_id";
</code></pre>

* 在第一个查询中，我们获取了用户的用户名、密码以及是否启用的信息，这些信息会用来进行用户认证。
* 接下来的查询查找了用户所授予的权限，用来进行鉴权。
* 最后一个查询中，查找了用户作为群组的成员所授予的权限。

可以按照如下的方式配置自己的查询：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
    .dataSource(dataSource)
    .usersByUsernameQuery(
      "select username, password, true " +
      "from Spitter where username=?")
    .authoritiesByUsernameQuery(
      "select username, 'ROLE_USER' from Spitter where username=?");
}
```

在本例中，我们只重写了认证和基本权限的查询语句，但是通过调用 **groupAuthoritiesByUsername**() 方法，我们也能够**将群组权限重写为自定义的查询语句**。

{% hint style="info" %}
**将默认的 SQL 查询替换为自定义的设计时，很重要的一点就是要遵循查询的基本协议：**

* **所有查询都将用户名作为唯一的参数。**
* **认证查询会选取用户名、密码以及启用状态信息。**
* **权限查询会选取零行或多行包含该用户名及其权限信息的数据。**
* **群组权限查询会选取零行或多行数据，每行数据中都会包含群组 ID、群组名称以及权限。**
{% endhint %}

## 使用编码后的密码

上面的认证查询，会预期用户密码存储在了数据库之中。这里唯一的问题在于如果密码明文存储的话，会很容易受到黑客的窃取。但是，<mark style="color:orange;">**如果数据库中的密码进行了编码的话，那么认证就会失败，因为它与用户提交的明文密码并不匹配。**</mark>

为了解决这个问题，我们需要借助 <mark style="color:blue;">**passwordEncoder() 方法指定一个密码编码器（encoder）**</mark>：

```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
  auth
    .jdbcAuthentication()
    .dataSource(dataSource)
    .usersByUsernameQuery(
      "select username, password, true " +
      "from Spitter where username=?")
    .authoritiesByUsernameQuery(
      "select username, 'ROLE_USER' from Spitter where username=?")
    .passwordEncoder(new StandardPasswordEnconder("123456"));
}
```

**passwordEncoder() 方法可以接受 Spring Security 中 PasswordEncoder 接口的任意实现。**

{% hint style="warning" %}
## <mark style="color:orange;">注意：数据库中的密码是永远不会解码的。</mark>

<mark style="color:blue;">**用户在登录时输入的密码会按照相同的算法进行编码，然后再与数据库中已经编码过的密码进行对比。**</mark>这个对比是在 PasswordEncoder 的 matches() 方法中进行的。
{% endhint %}

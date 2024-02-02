# 内置的过滤器工厂

内置的过滤器工厂允许我们在代码中注入策略执行点，并以一致的方式对所有服务调用执行大量操作。换句话说，这些**过滤器让我们修改传入和传出的HTTP请求和响应**。

下表包含了Spring Cloud Gateway中所有内置过滤器的列表：

| 拦截器                  | 描述                                                                                                                                                                                                  | 示例                                         |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------ |
| AddRequestHeader     | 添加一个带有名称和值参数的HTTP请求头。                                                                                                                                                                               | AddRequestHeader=X-Organization-ID, F39s2  |
| AddResponseHeader    | 添加一个带有名称和值参数的HTTP响应头。                                                                                                                                                                               | AddResponseHeader=X-Organization-ID, F39s2 |
| AddRequestParameter  | 添加一个带有名称和值参数的HTTP查询参数。                                                                                                                                                                              | AddRequestParameter=Organizationid, F39s2  |
| PrefixPath           | 在HTTP请求路径上添加前缀。                                                                                                                                                                                     | PrefixPath=/api                            |
| RequestRateLimiter   | <p>接收三个参数：</p><ul><li><strong>replenishRate</strong>：表示每秒允许用户发出的请求数量；</li><li><strong>capacity</strong>：定义允许的突发容量有多大；</li><li><strong>keyResolverName</strong>：定义实现KeyResolver接口的bean的名称。</li></ul> |                                            |
| RedirectTo           | 接受两个参数，一个状态码和一个URL。状态码应该是300重定向的HTTP代码。                                                                                                                                                             | RedirectTo=302, http://localhost:8072      |
| RemoveNonProxy       | 删除一些头部，例如Keep-Alive、Proxy-Authenticate或Proxy-Authorization。                                                                                                                                         | NA                                         |
| RemoveRequestHeader  | 从HTTP请求中删除与接收的名称匹配的头部。                                                                                                                                                                              | RemoveRequestHeader=X-Request-Foo          |
| RemoveResponseHeader | 从HTTP响应中删除与接收的名称匹配的头部。                                                                                                                                                                              | RemoveResponseHeader=X-Organization-ID     |
| **RewritePath**      | 接受路径正则表达式参数和替换参数。                                                                                                                                                                                   | RewritePath=/organization/(?.\*), /${path} |
| SecureHeaders        | 向响应添加安全头，并接收一个路径模板参数，该参数更改请求路径。                                                                                                                                                                     | NA                                         |
| SetPath              | 接收路径模板作为参数。通过允许路径上的模板段，它操纵请求路径。使用了Spring框架的URI模板。允许多个匹配段。                                                                                                                                           | SetPath=/{organization}                    |
| SetStatus            | 接收有效的HTTP状态码并更改HTTP响应的状态。                                                                                                                                                                           | SetStatus=500                              |
| SetResponseHeader    | 接受名称和值参数以在HTTP响应中设置头部。                                                                                                                                                                              | SetResponseHeader=X-Response-ID,123        |
# 内置的谓词工厂

内置的<mark style="color:blue;">**谓词**</mark>是一组对象，**允许我们在执行或处理请求之前检查请求是否满足一组条件**。**对于每个路由，我们可以设置多个谓词工厂，通过逻辑AND使用和组合它们。**

下表列出了Spring Cloud Gateway中所有内置的谓词工厂：

| 谓词         | 描述                                                              | 示例                                    |
| ---------- | --------------------------------------------------------------- | ------------------------------------- |
| Before     | 接收日期时间参数并匹配所有发生在其之前的请求。                                         | Before=2020-03-11T...                 |
| After      | 接收日期时间参数并匹配所有发生在其之后的请求。                                         | After=2020-03-11T...                  |
| Between    | 接收两个日期时间参数并匹配它们之间的所有请求。第一个日期时间是包含的，第二个是排除的。                     | Between=2020-03-11T...,2020-04-11T... |
| Header     | 接收两个参数，头的名称和一个正则表达式，然后使用提供的正则表达式匹配其值。                           | Header=X-Request-Id, \d+              |
| Host       | 接收一个Ant样式的模式，使用“.”分隔的主机名模式作为参数。然后，它将Host头与给定的模式匹配。              | Host=\*\*.example.com                 |
| Method     | 接收要匹配的HTTP方法。                                                   | Method=GET                            |
| Path       | 接收Spring的PathMatcher。                                           | Path=/organization/{id}               |
| Query      | 接收两个参数，一个必需参数和一个可选的正则表达式，然后将其与查询参数匹配。                           | Query=id, 1                           |
| Cookie     | 接收两个参数，一个cookie的名称和一个正则表达式，并查找HTTP请求头中的cookie，然后使用提供的正则表达式匹配其值。 | Cookie=SessionID, abc                 |
| RemoteAddr | 接收IP地址列表并将其与请求的远程地址匹配。                                          | RemoteAddr=192.168.3.5/24             |

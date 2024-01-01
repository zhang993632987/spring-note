# 过滤 Web 请求

Spring Security 借助一系列 Servlet Filter 来提供各种安全性功能。<mark style="color:blue;">**DelegatingFilterProxy**</mark> 是一个特殊的 Servlet Filter，它本身所做的工作并不多。只是<mark style="color:blue;">**将工作委托给一个 javax.servlet.Filter 实现类，这个实现类作为一个Bean注册在 Spring 应用的上下文中**</mark>， 如图所示：

<figure><img src="../../.gitbook/assets/image (2) (1) (1).png" alt=""><figcaption></figcaption></figure>

> 如果你喜欢在传统的 web.xml 中配置 Servlet 和 Filter 的话，可以使用 \<filter> 元素，如下所示：
>
> ```markup
> <filter>
>   <filter-name>springSecurityFilterChain</filter-name>
>   <filter-class>
>     org.springframework.web.filter.DelegatingFilteProxy
>   </filter-class>
> </filter>
> ```
>
> 在这里，**最重要的是 \<filter-name> 设置成了 springSecurityFilterChain。**这是因为我们**马上就会将 Spring Security 配置在 Web 安全性之中，这里会有一个名为 springSecurityFilterChain 的 Filter bean，DelegatingFilterProxy 会将过滤逻辑委托给它**。

如果你希望借助 **WebApplicationInitializer** 以 Java 的方式来配置 **DelegatingFilterProxy** 的话，那么我们所需要做的就是创建一个扩展的新类：

{% code overflow="wrap" %}
```java
package spitter.config;
import org.springframwork.security.web.context.AbstractSecurityWebApplicationInitializer;

public class SecurityWebInitializer extends AbstractSecurityWebApplicationInitializer{
}
```
{% endcode %}

**AbstractSecurityWebApplicationInitializer 实现了 WebApplicationInitializer，因此 Spring 会发现它，并用它在 Web 容器中注册 DelegatingFilterProxy**。尽管我们可以重载它的 **appendFilters**() 或 **insertFilters**() 方法来注册自己选择的 Filter，但是要注册 DelegatingFilterProxy 的话，我们并不需要重载任何方法。

**不管我们通过 web.xml 还是通过 AbstractSecurityWebApplicationInitializer 的子类来配置 DelegatingFilterProxy，它都会拦截发往应用中的请求，并将请求委托给 **<mark style="color:blue;">**ID 为 springSecurityFilterChain 的bean**</mark>**。**

* **springSecurityFilterChain 本身是另一个特殊的 Filter，它也被称为FilterChainProxy**。
* **它可以链接任意一个或多个其他的 Filter**。
* **Spring Security 依赖一系列 Servlet Filter 来提供不同的安全特性**。

# 添加其他的 Servlet 和 Filter

基于 Java 的初始化器（initializer）的一个好处就在于我们<mark style="color:blue;">**可以定义任意数量的初始化器类**</mark>。因此，如果我们想往 Web 容器中注册其他组件的话，只需创建一个新的初始化器就可以了。最简单的方式就是实现 Spring 的 WebApplicationInitializer 接口。

例如，如下的程序清单展现了如何创建 WebApplicationInitializer 实现并注册一个 Servlet。

```java
package com.study.spring.config;

import org.springframework.web.WebApplicationInitializer;

import javax.servlet.*;
import java.io.IOException;

/**
 * @author Zhang B H
 * @create 2023-10-03 18:09
 */
public class MyServletInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext) {
        ServletRegistration.Dynamic myServlet = servletContext
                                .addServlet("myServlet", new MyServlet());
        myServlet.addMapping("/myServlet/**");
    }

    public static class MyServlet extends GenericServlet {

        @Override
        public void service(ServletRequest request, ServletResponse response) 
                throws IOException {
            response.getWriter().println("MyServlet");
        }
    }
}
```

类似地，我们还可以创建新的 WebApplicationInitializer 实现来注册 <mark style="color:blue;">**Listener**</mark> 和 <mark style="color:blue;">**Filter**</mark>。

{% hint style="info" %}
## <mark style="color:blue;">提示</mark>

**如果只是注册 Filter， 并且该 Filter 只会映射到 DispatcherServlet 上的话，所需要做的仅仅是重载 AbstractAnnotationConfigDispatcherServletInitializer 的 getServletFilters() 方法，getServletFilters() 方法返回的所有 Filter 都会映射到 DispatcherServlet 上。**
{% endhint %}

# Spring MVC 高级配置

在 AbstractAnnotationConfigDispatcherServletInitializer 将 DispatcherServlet 注册到 Servlet 容器中之后，就会调用 <mark style="color:blue;">**customizeRegistration()**</mark>，并将 Servlet 注册后得到的 <mark style="color:blue;">**Registration.Dynamic**</mark> 传递进来。

> <mark style="color:blue;">**通过重载 customizeRegistration() 方法，我们可以对 DispatcherServlet 进行额外的配置。**</mark>

借助 customizeRegistration() 方法中的 ServletRegistration.Dynamic，我们能够完成多项任务，包括：

* 通过调用 <mark style="color:blue;">**setLoadOnStartup()**</mark> 设置 <mark style="color:blue;">**load-on-startup**</mark> 优先级
* 通过 <mark style="color:blue;">**setInitParameter()**</mark> 设置<mark style="color:blue;">**初始化参数**</mark>
* 通过调用 <mark style="color:blue;">**setMultipartConfig()**</mark> 配置 Servlet 3.0 对 <mark style="color:blue;">**multipart**</mark> 的支持。

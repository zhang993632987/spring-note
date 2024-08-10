# Spring MVC请求的处理流程

<figure><img src="../../.gitbook/assets/Spring MVC请求.jpg" alt=""><figcaption></figcaption></figure>

如上图所示，Spring MVC 请求的处理流程如下：

1. 在请求<mark style="color:blue;">**离开浏览器**</mark>时，会带有用户所请求的内容信息，至少会包含请求的 URL。同时还可能带有其他的信息，例如用户提交的表单信息。
2.  请求旅程的第一站是 Spring 的 <mark style="color:blue;">**DispatcherServlet**</mark>。Spring MVC 的所有请求都会通过一个<mark style="color:blue;">**前端控制器**</mark>（front controller）Servlet。

    > 前端控制器是常用的 Web 应用程序模式，在 Spring MVC 中，DispatcherServlet 就是前端控制器。
3. DispatcherServlet 的任务是将请求发送给 Spring MVC 控制器（controller），控制器是一个用于处理请求的 Spring 组件。
   * DispatcherServlet 需要知道应该将请求发送给哪个控制器，所以 DispatcherServlet 会查询一个或多个<mark style="color:blue;">**处理器映射（handler mapping）**</mark>来确定请求的下一站在哪里。
   * 处理器映射会根据请求所携带的 URL 信息来进行决策。
4. 一旦选择了合适的控制器，DispatcherServlet 会将请求发送给选中的<mark style="color:blue;">**控制器**</mark>。到了控制器，请求会卸下其负载（用户提交的信息）并耐心地等待控制器处理这些信息。
5. 控制器在完成处理逻辑后，通常会产生一些信息，这些信息需要返回给用户并在浏览器上显示。这些信息被称为<mark style="color:blue;">**模型（Model）**</mark>。
6. 控制器所做的最后一件事就是将模型数据打包，并且标示出用于渲染输出的视图名。它接下来会将请求连同模型和视图名发送回 DispatcherServlet。
7. DispatcherServlet 将会使用<mark style="color:blue;">**视图解析器（view resolver）**</mark>来将**逻辑视图名**匹配为一个特定的**视图实现**。
8. DispatcherServlet 的最后一站是视图的实现，在这里它交付模型数据，请求的任务就完成了。视图将使用模型数据渲染输出，这个输出会通过响应对象传递给客户端。

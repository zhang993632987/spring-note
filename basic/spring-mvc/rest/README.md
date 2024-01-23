# 5 REST

<mark style="color:blue;">**在 REST 中，资源通过 URL 进行识别和定位**</mark>。至于 RESTful URL 的结构并没有严格的规则，但是 URL 应该能够识别资源，而不是简单的发一条命令到服务器上。

REST 中会有行为，它们是通过 HTTP 方法来定义的。具体来讲，也就是 GET、POST、PUT、DELETE、PATCH 以及其他的 HTTP 方法构成了 REST 中的动作。这些 HTTP 方法通常会匹配为如下的 CRUD 动作：

* **Create**：<mark style="color:blue;">**POST**</mark>
* **Read**：<mark style="color:blue;">**GET**</mark>
* **Update**：<mark style="color:blue;">**PUT**</mark> 或 <mark style="color:blue;">**PATCH**</mark>
* **Delete**：<mark style="color:blue;">**DELETE**</mark>

Spring 支持以下方式来创建 REST 资源：

* **控制器可以处理所有的 HTTP 方法，包含四个主要的 REST 方法：GET、PUT、DELETE 以及 POST。Spring 3.2 及以上版本还支持 PATCH 方法；**
* **借助 **<mark style="color:blue;">**@PathVariable**</mark>** 注解，控制器能够处理参数化的 URL（将变量输入作为 URL 的一部分）；**
* 借助 Spring 的视图和视图解析器，资源能够以多种方式进行表述，包括将模型数据渲染为 XML、JSON、Atom 以及 RSS 的 View 实现；
* 可以使用 ContentNegotiatingViewResolver 来选择最适合客户端的表述；
* **借助 **<mark style="color:blue;">**@ResponseBody**</mark>** 注解和各种 HttpMethodConverter 实现，能够替换基于视图的渲染方式；**
* 类似地，<mark style="color:blue;">**@RequestBody**</mark>** 注解以及 HttpMethodConverter 实现可以将传入的 HTTP 数据转化为传入控制器处理方法的 Java 对象；**
* 借助 RestTemplate，Spring 应用能够方便地使用 REST 资源。

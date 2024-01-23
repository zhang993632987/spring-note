# ContentNegotiatingViewResolver

ContentNegotiatingViewResolver 最大的优势在于，它在 Spring MVC 之上构建了 REST 资源表述层，控制器代码无需修改。相同的一套控制器方法能够为面向人类的用户产生 HTML 内容，也能针对不是人类的客户端产生 JSON 或 XML。

如果面向人类用户的接口与面向非人类客户端的接口之间有很多重叠的话，那么内容协商是一种很便利的方案。在实践中，面向人类用户的视图与 REST API 在细节上很少能够处于相同的级别。<mark style="color:blue;">**如果面向人类用户的接口与面向非人类客户端的接口之间没有太多重叠的话，那么 ContentNegotiatingViewResolver 的优势就体现不出来了。**</mark>

ContentNegotiatingViewResolver 还有一个严重的限制。作为 ViewResolver 的实现，它只能决定资源该如何渲染到客户端，并没有涉及到客户端要发送什么样的表述给控制器使用。如果客户端发送 JSON 或 XML 的话，那么 ContentNegotiatingViewResolver 就无法提供帮助了。

> <mark style="color:red;">**因为存在上述这些限制，通常建议不要使用 ContentNegotiatingViewResolver。**</mark>

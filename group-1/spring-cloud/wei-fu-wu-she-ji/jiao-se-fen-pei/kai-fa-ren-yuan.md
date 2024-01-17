# 开发人员

* 遵循REST
* 使用HTTP/HTTPS作为服务的调用协议
* 将服务的行为映射到标准HTTP动词
* 使用JSON作为进出服务的所有数据的序列化格式
* 使用HTTP状态码来传达服务调用的状态

> **为什么JSON用于微服务？**
>
> * 与其他协议（如基于XML的SOAP（Simple Object Access Protocol，简单对象访问协议））相比，JSON是非常<mark style="color:blue;">**轻量级**</mark>的。
> * JSON<mark style="color:blue;">**易于人类阅读**</mark>和使用。
> * JSON是<mark style="color:blue;">**JavaScript中使用的默认序列化协议**</mark>。
>
> 但是，其他机制和协议能够比JSON更有效地在服务之间进行通信。
>
> * **Apache Thrift**框架允许你构建使用二进制协议相互通信的多语言服务。
> * **Apache Avro**协议是一种数据序列化协议，可在客户端和服务器调用之间将数据来回转换为二进制格式。
>
> 如果你需要最小化通过线路发送的数据的大小，建议你查看这些协议。但是根据经验，在你的微服务中使用直接的JSON就可以有效地工作，并且不用在你的服务消费者和服务客户端之间再插入一层通信用来调试。

{% hint style="info" %}
## <mark style="color:blue;">提示</mark>

<mark style="color:orange;">**在深入编写微服务之前，要确保你（以及你组织中的其他可能的团队）为想要公开的端点建立了标准。**</mark>应该使用微服务的URL（Uniform Resource Locator，统一资源定位器）来明确传达服务的**意图**、服务管理的**资源**以及服务内管理的资源之间存在的**关系**。

* <mark style="color:blue;">**使用明确的URL名称来确立服务所代表的资源**</mark>。使用规范的格式定义URL将有助于API更直观，更易于使用，并且在你的命名约定中保持一致。
*   <mark style="color:blue;">**使用URL来确立资源之间的关系**</mark>。通常，微服务中的资源之间会存在一种父子关系，其中子项不会存在于父项的上下文之外。因此，你可能没有针对该子项的单独的微服务。要使用URL来表达这些关系。

    > 如果你发现你的URL很长而且有嵌套，可能意味着你的微服务做的事情太多了。
* <mark style="color:blue;">**尽早建立URL的版本控制方案**</mark>。URL 及其对应的端点代表了服务的所有者和服务的消费者之间的契约。一种常见的模式是<mark style="color:blue;">**使用版本号作为前缀添加到所有端点上**</mark>。要尽早建立版本控制方案，并且坚持下去。在若干消费者使用它们之后，再想将URL改造成版本控制（例如，在URL映射中使用/v1/）是非常困难的。
{% endhint %}

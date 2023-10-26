# 1.3.6 Spring Cloud Sleuth

<mark style="color:blue;">**Spring Cloud Sleuth**</mark>允许**将唯一跟踪标识符集成到应用程序所使用的HTTP调用和消息通道（RabbitMQ、Apache Kafka）中**。这些跟踪号码（有时称为关联ID或跟踪ID）能够让开发人员在事务流经应用程序中的不同服务时跟踪事务。**有了Spring Cloud Sleuth，跟踪ID会被自动添加到微服务生成的任何日志记录语句中。**

Spring Cloud Sleuth与**日志聚合技术工具**（如ELK技术栈）和**跟踪工具**（如Zipkin）结合时，能够展现出真正的威力。

* **Open Zipkin**获取Spring Cloud Sleuth生成的数据，让你可以可视化单个事务所涉及的服务调用流程。
* **ELK技术栈**是3个开源项目（Elasticsearch、Logstash和Kibana）名称的首字母缩写。
  * **Elasticsearch**是搜索和分析引擎。
  * **Logstash** 是服务器端数据处理管道，它消费数据并转换数据，以便将数据发送到“秘密存储点（stash）”。
  * **Kibana**是一个客户端用户界面，允许用户查询并可视化整个技术栈的数据。

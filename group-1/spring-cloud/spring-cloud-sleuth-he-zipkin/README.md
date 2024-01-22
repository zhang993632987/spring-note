# 10.Spring Cloud Sleuth 和 Zipkin

* Spring Cloud Sleuth 是一个用于处理我们传入的HTTP请求的Spring Cloud Sleuth项目，通过添加过滤器和与其他Spring组件的交互，为这些请求生成追踪ID（也称为关联ID）。这样做的目的是确保生成的关联ID能够通过所有的系统调用传递。
* Zipkin 是一个开源的数据可视化工具，用于展示事务在多个服务间的流程。Zipkin允许我们将一个事务分解为各个组成部分，并以可视化的方式识别潜在的性能热点。
* ELK Stack 结合了三个开源工具——Elasticsearch、Logstash和Kibana，使我们能够实时分析、搜索和可视化日志。&#x20;
  * Elasticsearch 是一种分布式分析引擎，适用于各种类型的数据（结构化和非结构化、数值、基于文本等）。
  * Logstash是一个位于服务器端的数据处理管道，允许我们同时从多个源添加和摄取数据，并在将其索引到Elasticsearch之前进行转换。&#x20;
  * Kibana是用于Elasticsearch的可视化和数据管理工具。它提供图表、地图和实时直方图。

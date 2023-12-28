# 10.Spring Cloud Sleuth and Zipkin

* Spring Cloud Sleuth (https://cloud.spring.io/spring-cloud-sleuth/reference/html/)—The Spring Cloud Sleuth project instruments our incoming HTTP requests with trace IDs (aka correlation IDs). It does this by adding filters and interacting with other Spring components to let the generated correlation IDs pass through to all the system calls.
* Zipkin (https://zipkin.io/)—Zipkin is an open source data-visualization tool that shows the flow of a transaction across multiple services. Zipkin allows us to break a transaction down into its component pieces and visually identify where there might be performance hotspots.
* ELK stack (https://www.elastic.co/what-is/elk-stack)—The ELK stack combines three open source tools—Elasticsearch, Logstash, and Kibana—that allow us to analyze, search, and visualize logs in real time.
  * Elasticsearch is a distributed analytics engine for all types of data (structured and non-structured, numeric, text-based, and so on).
  * Logstash is a server-side data processing pipeline that allows us to add and ingest data from multiple sources simultaneously and transform it before it is indexed in Elasticsearch.
  * Kibana is the visualization and data management tool for Elasticsearch. It provides charts, maps, and real-time histograms.

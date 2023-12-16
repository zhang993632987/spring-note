# 9. Spring Cloud Stream

Using asynchronous messages to communicate between applications isn’t new. What’s new is the concept of using messages to communicate events representing changes in state. This concept is called event-driven architecture (EDA). It’s also known as message-driven architecture (MDA). What an EDA-based approach allows us to do is to build highly decoupled systems that can react to changes without being tightly coupled to specific libraries or services. When combined with microservices, EDA allows us to quickly add new functionality to our application by merely having the service listen to the stream of events (messages) being emitted by our application.

> we can use Apache Avro(https://avro.apache.org/) if we need that. Avro is a binary protocol that has versioning built into it. Spring Cloud Stream supports Apache Avro as a messaging protocol.

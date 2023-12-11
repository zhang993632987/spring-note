# What is a service gateway?

A service gateway acts as an intermediary between the service client and an invoked service. The service client talks only to a single URL managed by the service gateway.The service gateway pulls apart the path coming in from the service client call and determines what service the service client is trying to invoke.

With a service gateway in place, our service clients never directly call the URL of an individual service, but instead place all calls to the service gateway.

Because a service gateway sits between all calls from the client to the individual services, it also acts as a central PEP for service calls. The use of a centralized PEP means that cross-cutting service concerns can be carried out in a single place without the individual development teams having to implement those concerns. Examples of cross-cutting concerns that can be implemented in a service gateway include these:

Static routing—A service gateway places all service calls behind a single URL and API route. This simplifies development as we only have to know about one service endpoint for all of our services.

Dynamic routing—A service gateway can inspect incoming service requests and,based on the data from the incoming request, perform intelligent routing for the service caller. For instance, customers participating in a beta program might have all calls to a service routed to a specific cluster of services that are running a different version of code from what everyone else is using.

Authentication and authorization—Because all service calls route through a service gateway, the service gateway is a natural place to check whether the callers of a service have authenticated themselves.

Metric collection and logging—A service gateway can be used to collect metrics and log information as a service call passes through it. You can also use the service gateway to confirm that critical pieces of information are in place for user requests, thereby ensuring that logging is uniform. This doesn’t mean that you shouldn’t collect metrics from within your individual services. Rather, a service gateway allows you to centralize the collection of many of your basic metrics,like the number of times the service is invoked and the service response times.

> isn’t a service gateway a single point of failure and a potential bottleneck?
>
> Aservice gateway, if not implemented correctly, can carry the same risk. Keep the following in mind as you build your service gateway implementation:
>
> Load balancers are useful when placed in front of individual groups of services.In this case, a load balancer sitting in front of multiple service gateway instances is an appropriate design and ensures that your service gateway implementation can scale as needed. But having a load balancer sitting in front of all your service instances isn’t a good idea because it becomes a bottleneck.
>
> Keep any code you write for your service gateway stateless. Don’t store any information in memory for the service gateway. If you aren’t careful, you can limit the scalability of the gateway. Then, you will need to ensure that the data gets replicated across all service gateway instances.
>
> Keep the code you write for your service gateway light. The service gateway is the “chokepoint” for your service invocation. Complex code with multiple database calls can be the source of difficult-to-track performance problems inthe service gateway.

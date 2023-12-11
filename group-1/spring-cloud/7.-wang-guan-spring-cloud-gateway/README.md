# 7. 网关Spring Cloud Gateway

In a distributed architecture like a microservice, there will come a point where we’ll need to ensure that critical behaviors such as security, logging, and tracking users across multiple service calls occur. To implement this functionality, we’ll want these attributes to be consistently enforced across all of our services without the need for each individual development team to build their own solution. While it’s possible to use a common library or framework to assist with building these capabilities directly in an individual service, doing so has these implications:

It’s challenging to implement these capabilities in each service consistently. Developers are focused on delivering functionality, and in the whirlwind of day-to-day activity, they can easily forget to implement service logging or tracking unless they work in a regulated industry where it’s required.

Pushing the responsibilities to implement cross-cutting concerns like security and logging down to the individual development teams greatly increases the odds that someone will not implement them properly or will forget to do them.

It’s possible to create a hard dependency across all our services. The more capabilities we build into a common framework shared across all our services, the more difficult it is to change or add behavior in our common code without having to recompile and redeploy all our services. Suddenly an upgrade of core capabilities built into a shared library becomes a long migration process.

To solve this problem, we need to abstract these cross-cutting concerns into a service that can sit independently and act as a filter and router for all the microservice calls in our architecture. We call this service a gateway. Our service clients no longer directly call a microservice. Instead, all calls are routed through the service gateway, which acts as a single Policy Enforcement Point (PEP), and are then routed to a final destination.

# Configuring routes in Spring Cloud Gateway

At its heart, the Spring Cloud Gateway is a reverse proxy.

In the case of a microservice architecture, Spring Cloud Gateway (our reverse proxy) takes a microservice call from a client and forwards it to the upstream service.The service client thinks it’s only communicating with the gateway.To communicate with the upstream services, the gateway has to know how to map the incoming call to the upstream route. The Spring Cloud Gateway has several mechanisms to do this, including:

Automated mapping of routes using service discovery

Manual mapping of routes using service discovery

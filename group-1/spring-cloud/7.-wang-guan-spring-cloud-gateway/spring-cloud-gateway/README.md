# Spring Cloud Gateway

This gateway is a nonblocking gateway. What does nonblocking mean? Nonblocking applications are written in such a way that the main threads are never blocked. Instead, these threads are always available to serve requests and to process them asynchronously in the background to return a response once processing is done.&#x20;

Spring Cloud Gateway offers several capabilities, including:

Mapping the routes for all the services in your application to a single URL. The Spring Cloud Gateway isn’t limited to a single URL, however. Actually, with it, we can define multiple route entry points, making route mapping extremely fine-grained (each service endpoint gets its own route mapping).&#x20;

Building filters that can inspect and act on the requests and responses coming through the gateway. These filters allow us to inject policy enforcement points in our code and to perform a wide number of actions on all of our service calls in a consistent fashion. In other words, these filters allow us to modify the incoming and outgoing HTTP requests and responses.

Building predicates, which are objects that allow us to check if the requests fulfill a set of given conditions before executing or processing a request. The Spring Cloud Gateway includes a set of built-in Route Predicate Factories.

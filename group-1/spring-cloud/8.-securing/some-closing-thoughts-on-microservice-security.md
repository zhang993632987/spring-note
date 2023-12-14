# Some closing thoughts on microservice security

As you shape your microservices for production use, you should also build your microservice security around the following practices:

* Use HTTPS/Secure Sockets Layer (SSL) for all service communications.
* Use an API gateway for all service calls.
* Provide zones for your services (for example, a public API and private API).
* Limit the attack surface of your microservices by locking down unneeded network ports.

Figure 9.28 shows how these different pieces fit together.

<figure><img src="../../../.gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

## Use HTTPS secure sockets layer (SSL) for all service communication

In a production environment, your microservices should communicate only through the encrypted channels provided through HTTPS and SSL. Note that the configuration and setup of HTTPS can be automated through your DevOps scripts.

## Use a service gateway to access your microservices

The individual servers, service endpoints, and ports your services are running on should never be directly accessible to the client. Instead, use a service gateway to act asan entry point and gatekeeper for your service calls. Configure the network layer on the operating system or container that your microservices are running in to only accept traffic from the service gateway. Remember, the service gateway can act as a Policy Enforcement Point (PEP), which can be enforced in all services.

Putting service calls through a service gateway allows you to be consistent in how you’re securing and auditing your services. A service gateway also allows you to lock down what port and endpoints you’re going to expose to the outside world.

## Zone your services into a public API and private API

Security, in general, is all about building layers of access and enforcing the concept of least privilege. Least privilege implies that a user should have only the bare minimum network access and privileges to do their day-to-day job. To this end, you should implementleast privilege by separating your services into two distinct zones: public and private.

The public zone contains all the public APIs that will be consumed by your clients. Public API microservices should carry out narrow tasks that are workflow oriented. These microservices tend to be service aggregators, pulling data and carrying out tasks across multiple services. Public microservices should also be behind their own service gateway and have their own authentication service for performing authentication and authorization. Access to public services by client applications should go through a single route protected by the service gateway. Also, the public zone should have its own authentication service.

The private zone acts as a wall to protect your core application functionality and data. It should only be accessible through a single, well-known port and should be locked down to only accept network traffic from the network subnet where the private services are running. The private zone should have its own gateway and authentication service. Public API services should authenticate against the private zone’s authentication service. All application data should at least be in the private zone’s network subnet and only accessible by microservices residing in the private zone.

## Limit the attack surface of your microservices by locking down unneeded network ports

Configure the operating system your service is running on to only allow inbound and outbound access to the ports or a piece of infrastructure needed by your service (monitoring, log aggregation).

Don’t focus only on inbound access ports. Many developers forget to lock down their outbound ports. Locking down your outbound ports can prevent data from being leaked out of your service if an attacker has compromised the service itself. Also, make sure you look at network port access in both your public and private API zones.

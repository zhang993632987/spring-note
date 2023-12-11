# Custom filters

The Spring Cloud Gateway allows us to build custom logic using a filter within the gateway.Spring Cloud Gateway supports the following two types of filters.&#x20;

Pre-filters—A pre-filter is invoked before the actual request is sent to the target destination. A pre-filter usually carries out the task of making sure that the service has a consistent message format (key HTTP headers are in place, for example) or acts as a gatekeeper to ensure that the user calling the service is authenticated (they are whom they say they are).

Post-filters—A post-filter is invoked after the target service, and a response is sent back to the client. Usually, we implement a post-filter to log the response back from the target service, handle errors, or audit the response for sensitive information.

> A pre-filter, however, cannot redirect the user to a different endpoint or service.

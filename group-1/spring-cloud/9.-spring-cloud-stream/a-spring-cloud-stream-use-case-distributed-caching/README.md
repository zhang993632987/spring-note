# A Spring Cloud Stream use case: Distributed caching

the licensing service will always check a distributed Redis cache for the organization data associated with a particular license. If the organization data exists in the cache, we’ll return the data from the cache. If it doesn’t, we’ll call the organization service and cache the results of the call in a Redis hash.

When the data is updated in the organization service, the organization service will issue a message to Kafka. The licensing service will pick up the message and issue a DELETE against Redis to clear the cache.

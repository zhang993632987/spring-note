# Defining custom channels

To create a custom channel, we’ll call inboundOrgChanges in the licensing service. You can define the channel with the CustomChannels interface, asshown in the following listing..

```java
public interface CustomChannels {
    
    
    // Returns a SubscribableChannel class for each channel exposed by @Input
    @Input("inboundOrgChanges") // Names the channel
    SubscribableChannel orgs();  
}
```

The key takeaway from listing 10.16 is that for each custom input channel we want to expose, we define a method with @Input that returns a SubscribableChannel class.&#x20;

We then use @OutputChannel before the method that will be called if we want to define output channels for publishing messages. In the case of an output channel, the defined method returns a MessageChannel class instead of the SubscribableChannel class used with the input channel. Here’s the call to @OutputChannel:

```java
@OutputChannel(“outboundOrg”)
MessageChannel outboundOrg();
```

Now that we have a custom input channel, we need to modify two more things to use it in the licensing service. First, we need to modify the licensing service to map the Kafka topic’s custom input channel name in the licensing configuration file. The following listing shows this change.

```properties
spring.cloud.stream.bindings.inboundOrgChanges.destination=orgChangeTopic
spring.cloud.stream.bindings.inboundOrgChanges.content-type=application/json
spring.cloud.stream.bindings.inboundOrgChanges.group=licensingGroup
spring.cloud.stream.kafka.binder.zkNodes=localhost
spring.cloud.stream.kafka.binder.brokers=localhost
```

Next, we need to inject the CustomChannels interface previously defined into a class that’s going to use it to process messages. For the distributed caching example, we’ve moved the code for handling an incoming message to the OrganizationChangeHandler licensingservice class.

The following listing shows the message handling code that we’ll use with the inboundOrgChanges channel we just defined.

<pre class="language-java"><code class="lang-java">/* 
Moves the @EnableBindings out of  Application.java and into OrganizationChangeHandler.
This time instead of using the Sink class, we use CustomChannels as the parameter to pass.
*/
@EnableBinding(CustomChannels.class)    
public class OrganizationChangeHandler {
<strong>     private static final Logger logger =
</strong>         LoggerFactory.getLogger(OrganizationChangeHandler.class);
     private OrganizationRedisRepository 
                organizationRedisRepository;  
     
     @StreamListener("inboundOrgChanges")   
     public void loggerSink(
               OrganizationChangeModel organization) {  
          logger.debug("Received a message of type " +
                organization.getType());
          logger.debug("Received a message with an event {} from the
               organization service for the organization id {} ", 
               organization.getType(), organization.getType());
         }
     }
</code></pre>

Now, let’s create an organization and then find it.

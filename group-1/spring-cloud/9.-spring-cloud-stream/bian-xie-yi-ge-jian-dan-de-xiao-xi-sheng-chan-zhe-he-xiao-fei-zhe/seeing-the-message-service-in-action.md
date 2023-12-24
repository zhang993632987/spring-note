# Seeing the message service in action

We’ll use the http://localhost:8072/organization/v1/organization/endpoint and send the following body on the POST call to the endpoint.

```javascript
@Slf4j
@Component
public class SimpleSourceBean {

    private MessageChannel sendOrgChannel;

    public void publishOrganizationChange(ActionEnum action,
                                          String organizationId) {
        log.debug("Sending Kafka message {} for Organization Id: {}",
                action, organizationId);
        // Publishes a Java POJO message
        OrganizationChangeModel change = new OrganizationChangeModel(
                OrganizationChangeModel.class.getTypeName(),
                action.toString(),
                organizationId,
                UserContextHolder.getContext().getCorrelationId()
        );
        // Sends the message from a channel defined in the Source class
        sendOrgChannel.send(MessageBuilder.withPayload(change).build());
    }

    @Lazy
    @Qualifier("send-org")
    @Autowired(required = false)
    public void setSendOrgChannel(MessageChannel sendOrgChannel) {
        this.sendOrgChannel = sendOrgChannel;
    }
}
```

Once we make the organization service call, we should see the output shown in figure 10.8 in the console window running the services.

{% code overflow="wrap" lineNumbers="true" %}
```json
$ docker exec -it tools-kafka-1 kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic orgChangeTopic
{"typeName":"com.study.organization.model.OrganizationChangeModel","action":"GET","organizationId":"1","correlationId":null}
```
{% endcode %}

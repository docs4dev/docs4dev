## 46. Tracing Bus Events

Bus events (subclasses of  `RemoteApplicationEvent` ) can be traced by setting  `spring.cloud.bus.trace.enabled=true` . If you do so, the Spring Boot  `TraceRepository`  (if it is present) shows each event sent and all the acks from each service instance. The following example comes from the  `/trace`  endpoint:

```java
{
"timestamp": "2015-11-26T10:24:44.411+0000",
"info": {
"signal": "spring.cloud.bus.ack",
"type": "RefreshRemoteApplicationEvent",
"id": "c4d374b7-58ea-4928-a312-31984def293b",
"origin": "stores:8081",
"destination": "*:**"
}
},
{
"timestamp": "2015-11-26T10:24:41.864+0000",
"info": {
"signal": "spring.cloud.bus.sent",
"type": "RefreshRemoteApplicationEvent",
"id": "c4d374b7-58ea-4928-a312-31984def293b",
"origin": "customers:9000",
"destination": "*:**"
}
},
{
"timestamp": "2015-11-26T10:24:41.862+0000",
"info": {
"signal": "spring.cloud.bus.ack",
"type": "RefreshRemoteApplicationEvent",
"id": "c4d374b7-58ea-4928-a312-31984def293b",
"origin": "customers:9000",
"destination": "*:**"
}
}
```

The preceding trace shows that a  `RefreshRemoteApplicationEvent`  was sent from  `customers:9000` , broadcast to all services, and received (acked) by  `customers:9000`  and  `stores:8081` .

To handle the ack signals yourself, you could add an  `@EventListener`  for the  `AckRemoteApplicationEvent`  and  `SentApplicationEvent`  types to your app (and enable tracing). Alternatively, you could tap into the  `TraceRepository`  and mine the data from there.

> Any Bus application can trace acks. However, sometimes, it is useful to do this in a central service that can do more complex queries on the data or forward it to a specialized tracing service.


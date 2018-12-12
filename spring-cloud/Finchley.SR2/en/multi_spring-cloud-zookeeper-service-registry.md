## 75. Spring Cloud Zookeeper and Service Registry

Spring Cloud Zookeeper implements the  `ServiceRegistry`  interface, letting developers register arbitrary services in a programmatic way.

The  `ServiceInstanceRegistration`  class offers a  `builder()`  method to create a  `Registration`  object that can be used by the  `ServiceRegistry` , as shown in the following example:

```java
@Autowired
private ZookeeperServiceRegistry serviceRegistry;

public void registerThings() {
ZookeeperRegistration registration = ServiceInstanceRegistration.builder()
.defaultUriSpec()
.address("anyUrl")
.port(10)
.name("/a/b/c/d/anotherservice")
.build();
this.serviceRegistry.register(registration);
}
```

## 75.1 Instance Status

Netflix Eureka supports having instances that are  `OUT_OF_SERVICE`  registered with the server. These instances are not returned as active service instances. This is useful for behaviors such as blue/green deployments. (Note that the Curator Service Discovery recipe does not support this behavior.) Taking advantage of the flexible payload has let Spring Cloud Zookeeper implement  `OUT_OF_SERVICE`  by updating some specific metadata and then filtering on that metadata in the Ribbon  `ZookeeperServerList` . The  `ZookeeperServerList`  filters out all non-null instance statuses that do not equal  `UP` . If the instance status field is empty, it is considered to be  `UP`  for backwards compatibility. To change the status of an instance, make a  `POST`  with  `OUT_OF_SERVICE`  to the  `ServiceRegistry`  instance status actuator endpoint, as shown in the following example:

```java
$ http POST http://localhost:8081/service-registry status=OUT_OF_SERVICE
```

> The preceding example uses the  `http`  command from [https://httpie.org](https://httpie.org).


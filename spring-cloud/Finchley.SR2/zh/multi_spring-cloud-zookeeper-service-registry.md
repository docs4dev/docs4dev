## 75. Spring Cloud Zookeeper和Service Registry

Spring Cloud Zookeeper实现了 `ServiceRegistry` 接口，允许开发人员以编程方式注册任意服务.

`ServiceInstanceRegistration` 类提供 `builder()` 方法来创建 `Registration` 可以使用的 `Registration` 对象，如以下示例所示：

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

## 75.1实例状态

Netflix Eureka支持在服务器上注册 `OUT_OF_SERVICE` 的实例.这些实例不作为活动服务实例返回.这对于蓝/绿部署等行为非常有用. （请注意，Curator Service Discovery配方不支持此行为.）利用灵活的有效负载，Spring Cloud Zookeeper可以通过更新某些特定元数据然后在功能区_13817中过滤该元数据来实现 `OUT_OF_SERVICE` .  `ZookeeperServerList` 过滤掉所有不等于 `UP` 的非空实例状态.如果实例状态字段为空，则认为它是 `UP` 向后兼容性.要更改实例的状态，请使用 `POST`   `ServiceRegistry` 实例状态Actuatorendpoints，如以下示例所示：

```java
$ http POST http://localhost:8081/service-registry status=OUT_OF_SERVICE
```

> 前面的示例使用[https://httpie.org](https://httpie.org)中的 `http` 命令.


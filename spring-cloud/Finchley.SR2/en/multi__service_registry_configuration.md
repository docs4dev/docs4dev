## 107. Service Registry Configuration

You can use a  `DiscoveryClient`  (such as from Spring Cloud Consul) to locate a Vault server by setting spring.cloud.vault.discovery.enabled=true (default  `false` ). The net result of that is that your apps need a bootstrap.yml (or an environment variable) with the appropriate discovery configuration. The benefit is that the Vault can change its co-ordinates, as long as the discovery service is a fixed point. The default service id is  `vault`  but you can change that on the client with  `spring.cloud.vault.discovery.serviceId` .

The discovery client implementations all support some kind of metadata map (e.g. for Eureka we have eureka.instance.metadataMap). Some additional properties of the service may need to be configured in its service registration metadata so that clients can connect correctly. Service registries that do not provide details about transport layer security need to provide a  `scheme`  metadata entry to be set either to  `https`  or  `http` . If no scheme is configured and the service is not exposed as secure service, then configuration defaults to  `spring.cloud.vault.scheme`  which is  `https`  when it’s not set.

```java
spring.cloud.vault.discovery:
enabled: true
service-id: my-vault-service
```


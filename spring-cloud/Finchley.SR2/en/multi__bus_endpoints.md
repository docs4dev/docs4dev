## 41. Bus Endpoints

Spring Cloud Bus provides two endpoints,  `/actuator/bus-refresh`  and  `/actuator/bus-env`  that correspond to individual actuator endpoints in Spring Cloud Commons,  `/actuator/refresh`  and  `/actuator/env`  respectively.

## 41.1 Bus Refresh Endpoint

The  `/actuator/bus-refresh`  endpoint clears the  `RefreshScope`  cache and rebinds  `@ConfigurationProperties` . See the [Refresh Scope](multi__spring_cloud_context_application_context_services.html#refresh-scope) documentation for more information.

To expose the  `/actuator/bus-refresh`  endpoint, you need to add following configuration to your application:

```java
management.endpoints.web.exposure.include=bus-refresh
```

## 41.2 Bus Env Endpoint

The  `/actuator/bus-env`  endpoint updates each instances environment with the specified key/value pair across multiple instances.

To expose the  `/actuator/bus-env`  endpoint, you need to add following configuration to your application:

```java
management.endpoints.web.exposure.include=bus-env
```

The  `/actuator/bus-env`  endpoint accepts  `POST`  requests with the following shape:

```java
{
	"name": "key1",
	"value": "value1"
}
```


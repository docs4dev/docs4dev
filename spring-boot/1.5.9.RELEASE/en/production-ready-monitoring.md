## 54. Monitoring and Management over HTTP

If you are developing a web application, Spring Boot Actuator auto-configures all enabled endpoints to be exposed over HTTP. The default convention is to use the  `id`  of the endpoint with a prefix of  `/actuator`  as the URL path. For example,  `health`  is exposed as  `/actuator/health` . TIP: Actuator is supported natively with Spring MVC, Spring WebFlux, and Jersey.

## 54.1 Customizing the Management Endpoint Paths

Sometimes, it is useful to customize the prefix for the management endpoints. For example, your application might already use  `/actuator`  for another purpose. You can use the  `management.endpoints.web.base-path`  property to change the prefix for your management endpoint, as shown in the following example:

```java
management.endpoints.web.base-path=/manage
```

The preceding  `application.properties`  example changes the endpoint from  `/actuator/{id}`  to  `/manage/{id}`  (for example,  `/manage/info` ).

> Unless the management port has been configured to [expose endpoints by using a different HTTP port](production-ready-monitoring.html#production-ready-customizing-management-server-port),  `management.endpoints.web.base-path`  is relative to  `server.servlet.context-path` . If  `management.server.port`  is configured,  `management.endpoints.web.base-path`  is relative to  `management.server.servlet.context-path` .

If you want to map endpoints to a different path, you can use the  `management.endpoints.web.path-mapping`  property.

The following example remaps  `/actuator/health`  to  `/healthcheck` :

**application.properties.**  

```java
management.endpoints.web.base-path=/
management.endpoints.web.path-mapping.health=healthcheck
```

## 54.2 Customizing the Management Server Port

Exposing management endpoints by using the default HTTP port is a sensible choice for cloud-based deployments. If, however, your application runs inside your own data center, you may prefer to expose endpoints by using a different HTTP port.

You can set the  `management.server.port`  property to change the HTTP port, as shown in the following example:

```java
management.server.port=8081
```

## 54.3 Configuring Management-specific SSL

When configured to use a custom port, the management server can also be configured with its own SSL by using the various  `management.server.ssl.*`  properties. For example, doing so lets a management server be available over HTTP while the main application uses HTTPS, as shown in the following property settings:

```java
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:store.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=false
```

Alternatively, both the main server and the management server can use SSL but with different key stores, as follows:

```java
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:main.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=true
management.server.ssl.key-store=classpath:management.jks
management.server.ssl.key-password=secret
```

## 54.4 Customizing the Management Server Address

You can customize the address that the management endpoints are available on by setting the  `management.server.address`  property. Doing so can be useful if you want to listen only on an internal or ops-facing network or to listen only for connections from  `localhost` .

> You can listen on a different address only when the port differs from the main server port.

The following example  `application.properties`  does not allow remote management connections:

```java
management.server.port=8081
management.server.address=127.0.0.1
```

## 54.5 Disabling HTTP Endpoints

If you do not want to expose endpoints over HTTP, you can set the management port to  `-1` , as shown in the following example:

```java
management.server.port=-1
```

This can be achieved using the  `management.endpoints.web.exposure.exclude`  property as well, as shown in following example:

```java
management.endpoints.web.exposure.exclude=*
```


## 15. Hystrix Timeouts And Ribbon Clients

When using Hystrix commands that wrap Ribbon clients you want to make sure your Hystrix timeout is configured to be longer than the configured Ribbon timeout, including any potential retries that might be made. For example, if your Ribbon connection timeout is one second and the Ribbon client might retry the request three times, than your Hystrix timeout should be slightly more than three seconds.

## 15.1 How to Include the Hystrix Dashboard

To include the Hystrix Dashboard in your project, use the starter with a group ID of  `org.springframework.cloud`  and an artifact ID of  `spring-cloud-starter-netflix-hystrix-dashboard` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

To run the Hystrix Dashboard, annotate your Spring Boot main class with  `@EnableHystrixDashboard` . Then visit  `/hystrix`  and point the dashboard to an individual instance’s  `/hystrix.stream`  endpoint in a Hystrix client application.

> When connecting to a  `/hystrix.stream`  endpoint that uses HTTPS, the certificate used by the server must be trusted by the JVM. If the certificate is not trusted, you must import the certificate into the JVM in order for the Hystrix Dashboard to make a successful connection to the stream endpoint.

## 15.2 Turbine

Looking at an individual instance’s Hystrix data is not very useful in terms of the overall health of the system. [Turbine](https://github.com/Netflix/Turbine) is an application that aggregates all of the relevant  `/hystrix.stream`  endpoints into a combined  `/turbine.stream`  for use in the Hystrix Dashboard. Individual instances are located through Eureka. Running Turbine requires annotating your main class with the  `@EnableTurbine`  annotation (for example, by using spring-cloud-starter-netflix-turbine to set up the classpath). All of the documented configuration properties from [the Turbine 1 wiki](https://github.com/Netflix/Turbine/wiki/Configuration-(1.x)) apply. The only difference is that the  `turbine.instanceUrlSuffix`  does not need the port prepended, as this is handled automatically unless  `turbine.instanceInsertPort=false` .

> By default, Turbine looks for the  `/hystrix.stream`  endpoint on a registered instance by looking up its  `hostName`  and  `port`  entries in Eureka and then appending  `/hystrix.stream`  to it. If the instance’s metadata contains  `management.port` , it is used instead of the  `port`  value for the  `/hystrix.stream`  endpoint. By default, the metadata entry called  `management.port`  is equal to the  `management.port`  configuration property. It can be overridden though with following configuration:

```java
eureka:
instance:
metadata-map:
management.port: ${management.port:8081}
```

The  `turbine.appConfig`  configuration key is a list of Eureka serviceIds that turbine uses to lookup instances. The turbine stream is then used in the Hystrix dashboard with a URL similar to the following:

`http://my.turbine.server:8080/turbine.stream?cluster=CLUSTERNAME` 

The cluster parameter can be omitted if the name is  `default` . The  `cluster`  parameter must match an entry in  `turbine.aggregator.clusterConfig` . Values returned from Eureka are upper-case. Consequently, the following example works if there is an application called  `customers`  registered with Eureka:

```java
turbine:
aggregator:
clusterConfig: CUSTOMERS
appConfig: customers
```

If you need to customize which cluster names should be used by Turbine (because you do not want to store cluster names in  `turbine.aggregator.clusterConfig`  configuration), provide a bean of type  `TurbineClustersProvider` .

The  `clusterName`  can be customized by a SPEL expression in  `turbine.clusterNameExpression`  with root as an instance of  `InstanceInfo` . The default value is  `appName` , which means that the Eureka  `serviceId`  becomes the cluster key (that is, the  `InstanceInfo`  for customers has an  `appName`  of  `CUSTOMERS` ). A different example is  `turbine.clusterNameExpression=aSGName` , which gets the cluster name from the AWS ASG name. The following listing shows another example:

```java
turbine:
aggregator:
clusterConfig: SYSTEM,USER
appConfig: customers,stores,ui,admin
clusterNameExpression: metadata['cluster']
```

In the preceding example, the cluster name from four services is pulled from their metadata map and is expected to have values that include  `SYSTEM`  and  `USER` .

To use the “default” cluster for all apps, you need a string literal expression (with single quotes and escaped with double quotes if it is in YAML as well):

```java
turbine:
appConfig: customers,stores
clusterNameExpression: "'default'"
```

Spring Cloud provides a  `spring-cloud-starter-netflix-turbine`  that has all the dependencies you need to get a Turbine server running. To ad Turnbine, create a Spring Boot application and annotate it with  `@EnableTurbine` .

> By default, Spring Cloud lets Turbine use the host and port to allow multiple processes per host, per cluster. If you want the native Netflix behavior built into Turbine to not allow multiple processes per host, per cluster (the key to the instance ID is the hostname), set  `turbine.combineHostPort=false` .

### 15.2.1 Clusters Endpoint

In some situations it might be useful for other applications to know what custers have been configured in Turbine. To support this you can use the  `/clusters`  endpoint which will return a JSON array of all the configured clusters.

**GET /clusters.**  

```java
[
{
"name": "RACES",
"link": "http://localhost:8383/turbine.stream?cluster=RACES"
},
{
"name": "WEB",
"link": "http://localhost:8383/turbine.stream?cluster=WEB"
}
]
```

This endpoint can be disabled by setting  `turbine.endpoints.clusters.enabled`  to  `false` .

## 15.3 Turbine Stream

In some environments (such as in a PaaS setting), the classic Turbine model of pulling metrics from all the distributed Hystrix commands does not work. In that case, you might want to have your Hystrix commands push metrics to Turbine. Spring Cloud enables that with messaging. To do so on the client, add a dependency to  `spring-cloud-netflix-hystrix-stream`  and the  `spring-cloud-starter-stream-*`  of your choice. See the [Spring Cloud Stream documentation](https://docs.spring.io/spring-cloud-stream/docs/current/reference/htmlsingle/) for details on the brokers and how to configure the client credentials. It should work out of the box for a local broker.

On the server side, create a Spring Boot application and annotate it with  `@EnableTurbineStream` . The Turbine Stream server requires the use of Spring Webflux, therefore  `spring-boot-starter-webflux`  needs to be included in your project. By default  `spring-boot-starter-webflux`  is included when adding  `spring-cloud-starter-netflix-turbine-stream`  to your application.

You can then point the Hystrix Dashboard to the Turbine Stream Server instead of individual Hystrix streams. If Turbine Stream is running on port 8989 on myhost, then put  `http://myhost:8989`  in the stream input field in the Hystrix Dashboard. Circuits are prefixed by their respective  `serviceId` , followed by a dot ( `.` ), and then the circuit name.

Spring Cloud provides a  `spring-cloud-starter-netflix-turbine-stream`  that has all the dependencies you need to get a Turbine Stream server running. You can then add the Stream binder of your choice — such as  `spring-cloud-starter-stream-rabbit` .

Turbine Stream server also supports the  `cluster`  parameter. Unlike Turbine server, Turbine Stream uses eureka serviceIds as cluster names and these are not configurable.

If Turbine Stream server is running on port 8989 on  `my.turbine.server`  and you have two eureka serviceIds  `customers`  and  `products`  in your environment, the following URLs will be available on your Turbine Stream server.  `default`  and empty cluster name will provide all metrics that Turbine Stream server receives.

```java
http://my.turbine.sever:8989/turbine.stream?cluster=customers
http://my.turbine.sever:8989/turbine.stream?cluster=products
http://my.turbine.sever:8989/turbine.stream?cluster=default
http://my.turbine.sever:8989/turbine.stream
```

So, you can use eureka serviceIds as cluster names for your Turbine dashboard (or any compatible dashboard). You don’t need to configure any properties like  `turbine.appConfig` ,  `turbine.clusterNameExpression`  and  `turbine.aggregator.clusterConfig`  for your Turbine Stream server.

> Turbine Stream server gathers all metrics from the configured input channel with Spring Cloud Stream. It means that it doesn’t gather Hystrix metrics actively from each instance. It just can provide metrics that were already gathered into the input channel by each instance.


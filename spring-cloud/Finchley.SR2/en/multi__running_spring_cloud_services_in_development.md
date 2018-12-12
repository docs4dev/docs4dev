## 80. Running Spring Cloud Services in Development

The Launcher CLI can be used to run common services like Eureka, Config Server etc. from the command line. To list the available services you can do  `spring cloud --list` , and to launch a default set of services just  `spring cloud` . To choose the services to deploy, just list them on the command line, e.g.

```java
$ spring cloud eureka configserver h2 kafka stubrunner zipkin
```

Summary of supported deployables:

|Service|Name|Address|Description|
|----|----|----|----|
|eureka |Eureka Server |[http://localhost:8761](http://localhost:8761) |Eureka server for service registration and discovery. All the other services show up in its catalog by default. |
|configserver |Config Server |[http://localhost:8888](http://localhost:8888) |Spring Cloud Config Server running in the "native" profile and serving configuration from the local directory ./launcher |
|h2 |H2 Database |[http://localhost:9095](http://localhost:9095) (console), jdbc:h2:tcp://localhost:9096/{data} |Relation database service. Use a file path for  `{data}`  (e.g.  `./target/test` ) when you connect. Remember that you can add  `;MODE=MYSQL`  or  `;MODE=POSTGRESQL`  to connect with compatibility to other server types. |
|kafka |Kafka Broker |[http://localhost:9091](http://localhost:9091) (actuator endpoints), localhost:9092 | |
|hystrixdashboard |Hystrix Dashboard |[http://localhost:7979](http://localhost:7979) |Any Spring Cloud app that declares Hystrix circuit breakers publishes metrics on  `/hystrix.stream` . Type that address into the dashboard to visualize all the metrics, |
|dataflow |Dataflow Server |[http://localhost:9393](http://localhost:9393) |Spring Cloud Dataflow server with UI at /admin-ui. Connect the Dataflow shell to target at root path. |
|zipkin |Zipkin Server |[http://localhost:9411](http://localhost:9411) |Zipkin Server with UI for visualizing traces. Stores span data in memory and accepts them via HTTP POST of JSON data. |
|stubrunner |Stub Runner Boot |[http://localhost:8750](http://localhost:8750) |Downloads WireMock stubs, starts WireMock and feeds the started servers with stored stubs. Pass  `stubrunner.ids`  to pass stub coordinates and then go to  `http://localhost:8750/stubs` . |

Each of these apps can be configured using a local YAML file with the same name (in the current working directory or a subdirectory called "config" or in  `~/.spring-cloud` ). E.g. in  `configserver.yml`  you might want to do something like this to locate a local git repository for the backend:

**configserver.yml.**  

```java
spring:
profiles:
active: git
cloud:
config:
server:
git:
uri: file://${user.home}/dev/demo/config-repo
```

E.g. in Stub Runner app you could fetch stubs from your local  `.m2`  in the following way.

**stubrunner.yml.**  

```java
stubrunner:
workOffline: true
ids:
- com.example:beer-api-producer:+:9876
```

## 80.1 Adding Additional Applications

Additional applications can be added to  `./config/cloud.yml`  (not  `./config.yml`  because that would replace the defaults), e.g. with

**config/cloud.yml.**  

```java
spring:
cloud:
launcher:
deployables:
source:
coordinates: maven://com.example:source:0.0.1-SNAPSHOT
port: 7000
sink:
coordinates: maven://com.example:sink:0.0.1-SNAPSHOT
port: 7001
```

when you list the apps:

```java
$ spring cloud --list
source sink configserver dataflow eureka h2 hystrixdashboard kafka stubrunner zipkin
```

(notice the additional apps at the start of the list).


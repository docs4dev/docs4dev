## 80.在开发中运行Spring Cloud Services

Launcher CLI可用于从命令行运行Eureka，Config Server等常用服务.要列出可用的服务 `spring cloud --list` ，并启动一组默认服务，只需 `spring cloud` .要选择要部署的服务，只需在命令行中列出它们，例如：

```java
$ spring cloud eureka configserver h2 kafka stubrunner zipkin
```

支持的可部署的摘要：

|客服中心|名称|地址|说明|
| ---- | ---- | ---- | ---- |
| eureka | Eureka Server | [http://localhost:8761](http://localhost:8761) |用于服务注册和发现的Eureka服务器.默认情况下，所有其他服务都显示在其目录中. |
| configserver | Config Server | [http://localhost:8888](http://localhost:8888) | Spring Cloud Config Server在“本机”配置文件中运行，并从本地目录提供配置./launcher |
| h2 | H2数据库| [http://localhost:9095](http://localhost:9095)（控制台），jdbc：h2：tcp：// localhost：9096 / {data} |关系数据库服务.连接时使用 `{data}` 的文件路径（例如 `./target/test` ）.请记住，您可以添加 `;MODE=MYSQL` 或 `;MODE=POSTGRESQL` 以连接与其他服务器类型的兼容性. |
| kafka | Kafka Broker | [http://localhost:9091](http://localhost:9091)（Actuatorendpoints），localhost：9092 | |
| hystrixdashboard | Hystrix仪表板| [http://localhost:7979](http://localhost:7979) |任何宣布Hystrix断路器的Spring Cloud应用程序都会在 `/hystrix.stream` 上发布指标.在仪表板中键入该地址以显示所有指标，|
| dataflow | Dataflow Server | [http://localhost:9393](http://localhost:9393) |带有UI的Spring Cloud Dataflow服务器位于/ admin-ui.将Dataflow shell连接到根路径的目标. |
| zipkin | Zipkin Server | [http://localhost:9411](http://localhost:9411) |带有UI的Zipkin Server，用于可视化轨迹.存储跨越内存中的数据并通过JSON数据的HTTP POST接受它们. |
| stubrunner | Stub Runner Boot | [http://localhost:8750](http://localhost:8750) |下载WireMock存根，启动WireMock并使用存储的存根提供已启动的服务器.传递 `stubrunner.ids` 以传递存根坐标，然后转到 `http://localhost:8750/stubs` . |

可以使用具有相同名称的本地YAML文件（在当前工作目录或名为"config"或 `~/.spring-cloud` 的子目录中）配置这些应用程序中的每一个.例如.在 `configserver.yml` 中你可能想要做这样的事情来找到后端的本地git存储库：

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

例如.在Stub Runner应用程序中，您可以通过以下方式从本地 `.m2` 获取存根.

**stubrunner.yml.** 

```java
stubrunner:
workOffline: true
ids:
- com.example:beer-api-producer:+:9876
```

## 80.1添加其他应用程序

可以将其他应用程序添加到 `./config/cloud.yml` （而不是 `./config.yml` ，因为这将替换默认值），例如同

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

当您列出应用时：

```java
$ spring cloud --list
source sink configserver dataflow eureka h2 hystrixdashboard kafka stubrunner zipkin
```

（注意列表开头的其他应用程序）.


## 67.带Consul的分布式配置

Consul提供[Key/Value Store](https://consul.io/docs/agent/http/kv.html)用于存储配置和其他元数据. Spring Cloud Consul Config是[Config Server and Client](https://github.com/spring-cloud/spring-cloud-config)的替代品.在特殊的"bootstrap"阶段，配置被加载到Spring环境中.默认情况下，配置存储在 `/config` 文件夹中.基于应用程序的名称和模拟Spring Cloud Config解析属性顺序的活动配置文件创建多个 `PropertySource` 实例.例如，名为"testApp"且"dev"配置文件的应用程序将创建以下属性源：

```java
config/testApp,dev/
config/testApp/
config/application,dev/
config/application/
```

最具体的属性源位于顶部，底部的特定属性最少.  `config/application` 文件夹中的属性适用于使用consul进行配置的所有应用程序.  `config/testApp` 文件夹中的属性仅可用于名为"testApp"的服务的实例.

当前在应用程序启动时读取配置.将HTTP POST发送到 `/refresh` 将导致重新加载配置. [Section 67.3, “Config Watch”](multi_spring-cloud-consul-config.html#spring-cloud-consul-config-watch)还将自动检测更改并重新加载应用程序上下文.

## 67.1如何激活

要开始使用Consul Configuration，请使用具有组 `org.springframework.cloud` 和工件ID  `spring-cloud-starter-consul-config` 的启动器.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

这将启用将设置Spring Cloud Consul Config的自动配置.

## 67.2自定义

可以使用以下属性自定义Consul Config：

**bootstrap.yml.** 

```java
spring:
cloud:
consul:
config:
enabled: true
prefix: configuration
defaultContext: apps
profileSeparator: '::'
```

-  `enabled` 将此值设置为"false"禁用Consul Config

-  `prefix` 设置配置值的基本文件夹

-  `defaultContext` 设置所有应用程序使用的文件夹名称

-  `profileSeparator` 设置用于将属性源中的配置文件名称与配置文件分开的分隔符的值

## 67.3 Config Watch

Consul Config Watch利用Consul的能力[watch a key prefix](https://www.consul.io/docs/agent/watches.html#keyprefix). Config Watch进行阻塞Consul HTTP API调用，以确定当前应用程序是否有任何相关配置数据已更改.如果有新配置数据，则发布刷新事件.这相当于调用 `/refresh` Actuatorendpoints.

要更改调用Config Watch时的频率，请更改 `spring.cloud.consul.config.watch.delay` .默认值为1000，以毫秒为单位.延迟是上一次调用结束和下一次调用开始之后的时间量.

要禁用Config Watch set  `spring.cloud.consul.config.watch.enabled=false` .

Watch使用Spring  `TaskScheduler` 来安排对Consul的调用.默认情况下，它是 `ThreadPoolTaskScheduler` ， `poolSize` 为1.要更改 `TaskScheduler` ，请创建一个 `TaskScheduler` 类型的bean，其名称为 `ConsulConfigAutoConfiguration.CONFIG_WATCH_TASK_SCHEDULER_NAME` 常量.

## 67.4 YAML或带配置的属性

以单个键/值对而不是以YAML或属性格式存储属性blob可能更方便.将 `spring.cloud.consul.config.format` 属性设置为 `YAML` 或 `PROPERTIES` .例如，使用YAML：

**bootstrap.yml.** 

```java
spring:
cloud:
consul:
config:
format: YAML
```

必须在Consul中的相应 `data` 键中设置YAML.使用上面的默认值看起来像：

```java
config/testApp,dev/data
config/testApp/data
config/application,dev/data
config/application/data
```

您可以将YAML文档存储在上面列出的任何键中.

您可以使用 `spring.cloud.consul.config.data-key` 更改数据密钥.

## 67.5 git2consul with Config

git2consul是一个Consul社区项目，它将文件从git存储库加载到Consul中.默认情况下，键的名称是文件的名称. YAML和Properties文件分别支持 `.yml` 和 `.properties` 的文件扩展名.将 `spring.cloud.consul.config.format` 属性设置为 `FILES` .例如：

**bootstrap.yml.** 

```java
spring:
cloud:
consul:
config:
format: FILES
```

给定 `/config` 中的以下键， `development` 配置文件和应用程序名称 `foo` ：

```java
.gitignore
application.yml
bar.properties
foo-development.properties
foo-production.yml
foo.properties
master.ref
```

将创建以下属性源：

```java
config/foo-development.properties
config/foo.properties
config/application.yml
```

每个键的值必须是格式正确的YAML或属性文件.

## 67.6快速失败

如果consul不可用于配置，在某些情况下（如本地开发或某些测试场景）可能会很方便.在 `bootstrap.yml` 中设置 `spring.cloud.consul.config.failFast=false` 将导致配置模块记录警告而不是抛出异常.这将允许应用程序正常继续启动.


## 78.使用Zookeeper进行分布式配置

Zookeeper提供[hierarchical namespace](https://zookeeper.apache.org/doc/current/zookeeperOver.html#sc_dataModelNameSpace)，允许客户端存储任意数据，例如配置数据. Spring Cloud Zookeeper Config是[Config Server and Client](https://github.com/spring-cloud/spring-cloud-config)的替代品.在特殊的“bootstrap”阶段，配置被加载到Spring环境中.默认情况下，配置存储在 `/config` 命名空间中.根据应用程序的名称和活动配置文件创建多个 `PropertySource` 实例，以模拟解析属性的Spring Cloud Config顺序.例如，名称为 `testApp` 且 `dev` 配置文件的应用程序具有为其创建的以下属性源：

-  `config/testApp,dev` 

-  `config/testApp` 

-  `config/application,dev` 

-  `config/application` 

最具体的属性源位于顶部，底部的特定属性最少.  `config/application` 名称空间中的属性适用于使用zookeeper进行配置的所有应用程序.  `config/testApp` 命名空间中的属性仅适用于名为 `testApp` 的服务的实例.

当前在应用程序启动时读取配置.向 `/refresh` 发送HTTP  `POST` 请求会导致重新加载配置.目前尚未实现监视配置命名空间（Zookeeper支持）.

## 78.1激活

包含对 `org.springframework.cloud:spring-cloud-starter-zookeeper-config` 的依赖性可启用设置Spring Cloud Zookeeper Config的自动配置.

> 使用Zookeeper 3.4版时，您需要更改包含依赖关系的方式，如[here](multi_spring-cloud-zookeeper-install.html)所述.

## 78.2自定义

可以通过设置以下属性来自定义Zookeeper配置：

**bootstrap.yml.** 

```java
spring:
cloud:
zookeeper:
config:
enabled: true
root: configuration
defaultContext: apps
profileSeparator: '::'
```

-  `enabled` ：将此值设置为 `false` 将禁用Zookeeper配置.

-  `root` ：设置配置值的基本命名空间.

-  `defaultContext` ：设置所有应用程序使用的名称.

-  `profileSeparator` ：设置用于在属性中分隔概要文件名称的分隔符的值资料来源.

## 78.3访问控制列表（ACL）

您可以通过调用 `CuratorFramework`  bean的 `addAuthInfo` 方法为Zookeeper ACL添加身份验证信息.实现此目的的一种方法是提供自己的 `CuratorFramework`  bean，如以下示例所示：

```java
@BoostrapConfiguration
public class CustomCuratorFrameworkConfig {

@Bean
public CuratorFramework curatorFramework() {
CuratorFramework curator = new CuratorFramework();
curator.addAuthInfo("digest", "user:password".getBytes());
return curator;
}

}
```

请参阅[the ZookeeperAutoConfiguration class](https://github.com/spring-cloud/spring-cloud-zookeeper/blob/master/spring-cloud-zookeeper-core/src/main/java/org/springframework/cloud/zookeeper/ZookeeperAutoConfiguration.java)以了解 `CuratorFramework`  bean的默认配置.

或者，您可以从依赖于现有 `CuratorFramework`  bean的类添加凭据，如以下示例所示：

```java
@BoostrapConfiguration
public class DefaultCuratorFrameworkConfig {

public ZookeeperConfig(CuratorFramework curator) {
curator.addAuthInfo("digest", "user:password".getBytes());
}

}
```

这个bean的创建必须在boostrapping阶段进行.您可以在此阶段注册要运行的配置类，方法是使用 `@BootstrapConfiguration` 对它们进行注释，并将它们包含在以逗号分隔的列表中，并将其设置为 `resources/META-INF/spring.factories` 文件中 `org.springframework.cloud.bootstrap.BootstrapConfiguration` 属性的值，如以下示例所示：

**resources/META-INF/spring.factories.** 

```java
org.springframework.cloud.bootstrap.BootstrapConfiguration=\
my.project.CustomCuratorFrameworkConfig,\
my.project.DefaultCuratorFrameworkConfig
```


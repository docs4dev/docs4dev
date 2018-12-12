## 78. Distributed Configuration with Zookeeper

Zookeeper provides a [hierarchical namespace](https://zookeeper.apache.org/doc/current/zookeeperOver.html#sc_dataModelNameSpace) that lets clients store arbitrary data, such as configuration data. Spring Cloud Zookeeper Config is an alternative to the [Config Server and Client](https://github.com/spring-cloud/spring-cloud-config). Configuration is loaded into the Spring Environment during the special “bootstrap” phase. Configuration is stored in the  `/config`  namespace by default. Multiple  `PropertySource`  instances are created, based on the application’s name and the active profiles, to mimic the Spring Cloud Config order of resolving properties. For example, an application with a name of  `testApp`  and with the  `dev`  profile has the following property sources created for it:

-  `config/testApp,dev` 

-  `config/testApp` 

-  `config/application,dev` 

-  `config/application` 

The most specific property source is at the top, with the least specific at the bottom. Properties in the  `config/application`  namespace apply to all applications that use zookeeper for configuration. Properties in the  `config/testApp`  namespace are available only to the instances of the service named  `testApp` .

Configuration is currently read on startup of the application. Sending a HTTP  `POST`  request to  `/refresh`  causes the configuration to be reloaded. Watching the configuration namespace (which Zookeeper supports) is not currently implemented.

## 78.1 Activating

Including a dependency on  `org.springframework.cloud:spring-cloud-starter-zookeeper-config`  enables autoconfiguration that sets up Spring Cloud Zookeeper Config.

> When working with version 3.4 of Zookeeper you need to change the way you include the dependency as described [here](multi_spring-cloud-zookeeper-install.html).

## 78.2 Customizing

Zookeeper Config may be customized by setting the following properties:

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

-  `enabled` : Setting this value to  `false`  disables Zookeeper Config.

-  `root` : Sets the base namespace for configuration values.

-  `defaultContext` : Sets the name used by all applications.

-  `profileSeparator` : Sets the value of the separator used to separate the profile name in property sources with profiles.

## 78.3 Access Control Lists (ACLs)

You can add authentication information for Zookeeper ACLs by calling the  `addAuthInfo`  method of a  `CuratorFramework`  bean. One way to accomplish this is to provide your own  `CuratorFramework`  bean, as shown in the following example:

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

Consult [the ZookeeperAutoConfiguration class](https://github.com/spring-cloud/spring-cloud-zookeeper/blob/master/spring-cloud-zookeeper-core/src/main/java/org/springframework/cloud/zookeeper/ZookeeperAutoConfiguration.java) to see how the  `CuratorFramework`  bean’s default configuration.

Alternatively, you can add your credentials from a class that depends on the existing  `CuratorFramework`  bean, as shown in the following example:

```java
@BoostrapConfiguration
public class DefaultCuratorFrameworkConfig {

public ZookeeperConfig(CuratorFramework curator) {
curator.addAuthInfo("digest", "user:password".getBytes());
}

}
```

The creation of this bean must occur during the boostrapping phase. You can register configuration classes to run during this phase by annotating them with  `@BootstrapConfiguration`  and including them in a comma-separated list that you set as the value of the  `org.springframework.cloud.bootstrap.BootstrapConfiguration`  property in the  `resources/META-INF/spring.factories`  file, as shown in the following example:

**resources/META-INF/spring.factories.**  

```java
org.springframework.cloud.bootstrap.BootstrapConfiguration=\
my.project.CustomCuratorFrameworkConfig,\
my.project.DefaultCuratorFrameworkConfig
```


## 10. Spring Cloud Config Client

A Spring Boot application can take immediate advantage of the Spring Config Server (or other external property sources provided by the application developer). It also picks up some additional useful features related to  `Environment`  change events.

## 10.1 Config First Bootstrap

The default behavior for any application that has the Spring Cloud Config Client on the classpath is as follows: When a config client starts, it binds to the Config Server (through the  `spring.cloud.config.uri`  bootstrap configuration property) and initializes Spring  `Environment`  with remote property sources.

The net result of this behavior is that all client applciations that want to consume the Config Server need a  `bootstrap.yml`  (or an environment variable) with the server address set in  `spring.cloud.config.uri`  (it defaults to "http://localhost:8888").

## 10.2 Discovery First Bootstrap

If you use a `DiscoveryClient implementation, such as Spring Cloud Netflix and Eureka Service Discovery or Spring Cloud Consul, you can have the Config Server register with the Discovery Service. However, in the default “Config First” mode, clients cannot take advantage of the registration.

If you prefer to use  `DiscoveryClient`  to locate the Config Server, you can do so by setting  `spring.cloud.config.discovery.enabled=true`  (the default is  `false` ). The net result of doing so is that client applications all need a  `bootstrap.yml`  (or an environment variable) with the appropriate discovery configuration. For example, with Spring Cloud Netflix, you need to define the Eureka server address (for example, in  `eureka.client.serviceUrl.defaultZone` ). The price for using this option is an extra network round trip on startup, to locate the service registration. The benefit is that, as long as the Discovery Service is a fixed point, the Config Server can change its coordinates. The default service ID is  `configserver` , but you can change that on the client by setting  `spring.cloud.config.discovery.serviceId`  (and on the server, in the usual way for a service, such as by setting  `spring.application.name` ).

The discovery client implementations all support some kind of metadata map (for example, we have  `eureka.instance.metadataMap`  for Eureka). Some additional properties of the Config Server may need to be configured in its service registration metadata so that clients can connect correctly. If the Config Server is secured with HTTP Basic, you can configure the credentials as  `username`  and  `password` . Also, if the Config Server has a context path, you can set  `configPath` . For example, the following YAML file is for a Config Server that is a Eureka client:

**bootstrap.yml.**  

```java
eureka:
instance:
...
metadataMap:
user: osufhalskjrtl
password: lviuhlszvaorhvlo5847
configPath: /config
```

## 10.3 Config Client Fail Fast

In some cases, you may want to fail startup of a service if it cannot connect to the Config Server. If this is the desired behavior, set the bootstrap configuration property  `spring.cloud.config.fail-fast=true`  to make the client halt with an Exception.

## 10.4 Config Client Retry

If you expect that the config server may occasionally be unavailable when your application starts, you can make it keep trying after a failure. First, you need to set  `spring.cloud.config.fail-fast=true` . Then you need to add  `spring-retry`  and  `spring-boot-starter-aop`  to your classpath. The default behavior is to retry six times with an initial backoff interval of 1000ms and an exponential multiplier of 1.1 for subsequent backoffs. You can configure these properties (and others) by setting the  `spring.cloud.config.retry.*`  configuration properties.

> To take full control of the retry behavior, add a  `@Bean`  of type  `RetryOperationsInterceptor`  with an ID of  `configServerRetryInterceptor` . Spring Retry has a  `RetryInterceptorBuilder`  that supports creating one.

## 10.5 Locating Remote Configuration Resources

The Config Service serves property sources from  `/{name}/{profile}/{label}` , where the default bindings in the client app are as follows:

- "name" =  `${spring.application.name}` 

- "profile" =  `${spring.profiles.active}`  (actually  `Environment.getActiveProfiles()` )

- "label" = "master"

> When setting the property  `${spring.application.name}`  do not prefix your app name with the reserved word  `application-`  to prevent issues resolving the correct property source.

You can override all of them by setting  `spring.cloud.config.*`  (where  `*`  is  `name` ,  `profile`  or  `label` ). The  `label`  is useful for rolling back to previous versions of configuration. With the default Config Server implementation, it can be a git label, branch name, or commit ID. Label can also be provided as a comma-separated list. In that case, the items in the list are tried one by one until one succeeds. This behavior can be useful when working on a feature branch. For instance, you might want to align the config label with your branch but make it optional (in that case, use  `spring.cloud.config.label=myfeature,develop` ).

## 10.6 Specifying Multiple Urls for the Config Server

To ensure high availability when you have multiple instances of Config Server deployed and expect one or more instances to be unavailable from time to time, you can either specify multiple URLs (as a comma-separated list under the  `spring.cloud.config.uri`  property) or have all your instances register in a Service Registry like Eureka ( if using Discovery-First Bootstrap mode ). Note that doing so ensures high availability only when the Config Server is not running (that is, when the application has exited) or when a connection timeout has occurred. For example, if the Config Server returns a 500 (Internal Server Error) response or the Config Client receives a 401 from the Config Server (due to bad credentials or other causes), the Config Client does not try to fetch properties from other URLs. An error of that kind indicates a user issue rather than an availability problem.

If you use HTTP basic security on your Config Server, it is currently possible to support per-Config Server auth credentials only if you embed the credentials in each URL you specify under the  `spring.cloud.config.uri`  property. If you use any other kind of security mechanism, you cannot (currently) support per-Config Server authentication and authorization.

## 10.7 Configuring Read Timeouts

If you want to configure read timeout, this can be done by using the property  `spring.cloud.config.request-read-timeout` .

## 10.8 Security

If you use HTTP Basic security on the server, clients need to know the password (and username if it is not the default). You can specify the username and password through the config server URI or via separate username and password properties, as shown in the following example:

**bootstrap.yml.**  

```java
spring:
cloud:
config:
uri: https://user:[emailprotected]
```

The following example shows an alternate way to pass the same information:

**bootstrap.yml.**  

```java
spring:
cloud:
config:
uri: https://myconfig.mycompany.com
username: user
password: secret
```

The  `spring.cloud.config.password`  and  `spring.cloud.config.username`  values override anything that is provided in the URI.

If you deploy your apps on Cloud Foundry, the best way to provide the password is through service credentials (such as in the URI, since it does not need to be in a config file). The following example works locally and for a user-provided service on Cloud Foundry named  `configserver` :

**bootstrap.yml.**  

```java
spring:
cloud:
config:
uri: ${vcap.services.configserver.credentials.uri:http://user:[emailprotected]:8888}
```

If you use another form of security, you might need to [provide a RestTemplate](multi__spring_cloud_config_client.html#custom-rest-template) to the  `ConfigServicePropertySourceLocator`  (for example, by grabbing it in the bootstrap context and injecting it).

### 10.8.1 Health Indicator

The Config Client supplies a Spring Boot Health Indicator that attempts to load configuration from the Config Server. The health indicator can be disabled by setting  `health.config.enabled=false` . The response is also cached for performance reasons. The default cache time to live is 5 minutes. To change that value, set the  `health.config.time-to-live`  property (in milliseconds).

### 10.8.2 Providing A Custom RestTemplate

In some cases, you might need to customize the requests made to the config server from the client. Typically, doing so involves passing special  `Authorization`  headers to authenticate requests to the server. To provide a custom  `RestTemplate` :

Create a new configuration bean with an implementation of PropertySourceLocator, as shown in the following example:

**CustomConfigServiceBootstrapConfiguration.java.**  

```java
@Configuration
public class CustomConfigServiceBootstrapConfiguration {
@Bean
public ConfigServicePropertySourceLocator configServicePropertySourceLocator() {
ConfigClientProperties clientProperties = configClientProperties();
ConfigServicePropertySourceLocator configServicePropertySourceLocator =  new ConfigServicePropertySourceLocator(clientProperties);
configServicePropertySourceLocator.setRestTemplate(customRestTemplate(clientProperties));
return configServicePropertySourceLocator;
}
}
```

In resources/META-INF, create a file called spring.factories and specify your custom configuration, as shown in the following example:

**spring.factories.**  

```java
org.springframework.cloud.bootstrap.BootstrapConfiguration = com.my.config.client.CustomConfigServiceBootstrapConfiguration
```

### 10.8.3 Vault

When using Vault as a backend to your config server, the client needs to supply a token for the server to retrieve values from Vault. This token can be provided within the client by setting  `spring.cloud.config.token`  in  `bootstrap.yml` , as shown in the following example:

**bootstrap.yml.**  

```java
spring:
cloud:
config:
token: YourVaultToken
```

## 10.9 Nested Keys In Vault

Vault supports the ability to nest keys in a value stored in Vault, as shown in the following example:

`echo -n '{"appA": {"secret": "appAsecret"}, "bar": "baz"}' | vault write secret/myapp -` 

This command writes a JSON object to your Vault. To access these values in Spring, you would use the traditional dot( `.` ) annotation, as shown in the following example

```java
@Value("${appA.secret}")
String name = "World";
```

The preceding code would sets the value of the  `name`  variable to  `appAsecret` .


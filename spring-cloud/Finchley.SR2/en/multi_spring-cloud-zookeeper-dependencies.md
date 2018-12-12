## 76. Zookeeper Dependencies

The following topics cover how to work with Spring Cloud Zookeeper dependencies:

- [Section 76.1, “Using the Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-using)

- [Section 76.2, “Activating Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-activating)

- [Section 76.3, “Setting up Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-setting-up)

- [Section 76.4, “Configuring Spring Cloud Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-configuring)

## 76.1 Using the Zookeeper Dependencies

Spring Cloud Zookeeper gives you a possibility to provide dependencies of your application as properties. As dependencies, you can understand other applications that are registered in Zookeeper and which you would like to call through [Feign](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-feign) (a REST client builder) and [Spring RestTemplate](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-ribbon).

You can also use the Zookeeper Dependency Watchers functionality to control and monitor the state of your dependencies.

## 76.2 Activating Zookeeper Dependencies

Including a dependency on  `org.springframework.cloud:spring-cloud-starter-zookeeper-discovery`  enables autoconfiguration that sets up Spring Cloud Zookeeper Dependencies. Even if you provide the dependencies in your properties, you can turn off the dependencies. To do so, set the  `spring.cloud.zookeeper.dependency.enabled`  property to false (it defaults to  `true` ).

## 76.3 Setting up Zookeeper Dependencies

Consider the following example of dependency representation:

**application.yml.**  

```java
spring.application.name: yourServiceName
spring.cloud.zookeeper:
dependencies:
newsletter:
path: /path/where/newsletter/has/registered/in/zookeeper
loadBalancerType: ROUND_ROBIN
contentTypeTemplate: application/vnd.newsletter.$version+json
version: v1
headers:
header1:
- value1
header2:
- value2
required: false
stubs: org.springframework:foo:stubs
mailing:
path: /path/where/mailing/has/registered/in/zookeeper
loadBalancerType: ROUND_ROBIN
contentTypeTemplate: application/vnd.mailing.$version+json
version: v1
required: true
```

The next few sections go through each part of the dependency one by one. The root property name is  `spring.cloud.zookeeper.dependencies` .

### 76.3.1 Aliases

Below the root property you have to represent each dependency as an alias. This is due to the constraints of Ribbon, which requires that the application ID be placed in the URL. Consequently, you cannot pass any complex path, suchas  `/myApp/myRoute/name` ). The alias is the name you use instead of the  `serviceId`  for  `DiscoveryClient` ,  `Feign` , or  `RestTemplate` .

In the previous examples, the aliases are  `newsletter`  and  `mailing` . The following example shows Feign usage with a  `newsletter`  alias:

```java
@FeignClient("newsletter")
public interface NewsletterService {
@RequestMapping(method = RequestMethod.GET, value = "/newsletter")
String getNewsletters();
}
```

### 76.3.2 Path

The path is represented by the  `path`  YAML property and is the path under which the dependency is registered under Zookeeper. As described in the [previous section](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-setting-up-aliases), Ribbon operates on URLs. As a result, this path is not compliant with its requirement. That is why Spring Cloud Zookeeper maps the alias to the proper path.

### 76.3.3 Load Balancer Type

The load balancer type is represented by  `loadBalancerType`  YAML property.

If you know what kind of load-balancing strategy has to be applied when calling this particular dependency, you can provide it in the YAML file, and it is automatically applied. You can choose one of the following load balancing strategies:

- STICKY: Once chosen, the instance is always called.

- RANDOM: Picks an instance randomly.

- ROUND_ROBIN: Iterates over instances over and over again.

### 76.3.4 Content-Type Template and Version

The  `Content-Type`  template and version are represented by the  `contentTypeTemplate`  and  `version`  YAML properties.

If you version your API in the  `Content-Type`  header, you do not want to add this header to each of your requests. Also, if you want to call a new version of the API, you do not want to roam around your code to bump up the API version. That is why you can provide a  `contentTypeTemplate`  with a special  `$version`  placeholder. That placeholder will be filled by the value of the  `version`  YAML property. Consider the following example of a  `contentTypeTemplate` :

```java
application/vnd.newsletter.$version+json
```

Further consider the following  `version` :

```java
v1
```

The combination of  `contentTypeTemplate`  and version results in the creation of a  `Content-Type`  header for each request, as follows:

```java
application/vnd.newsletter.v1+json
```

### 76.3.5 Default Headers

Default headers are represented by the  `headers`  map in YAML.

Sometimes, each call to a dependency requires setting up of some default headers. To not do that in code, you can set them up in the YAML file, as shown in the following example  `headers`  section:

```java
headers:
Accept:
- text/html
- application/xhtml+xml
Cache-Control:
- no-cache
```

That  `headers`  section results in adding the  `Accept`  and  `Cache-Control`  headers with appropriate list of values in your HTTP request.

### 76.3.6 Required Dependencies

Required dependencies are represented by  `required`  property in YAML.

If one of your dependencies is required to be up when your application boots, you can set the  `required: true`  property in the YAML file.

If your application cannot localize the required dependency during boot time, it throws an exception, and the Spring Context fails to set up. In other words, your application cannot start if the required dependency is not registered in Zookeeper.

You can read more about Spring Cloud Zookeeper Presence Checker [later in this document](multi_spring-cloud-zookeeper-dependency-watcher.html#spring-cloud-zookeeper-dependency-watcher-presence-checker).

### 76.3.7 Stubs

You can provide a colon-separated path to the JAR containing stubs of the dependency, as shown in the following example:

`stubs: org.springframework:myApp:stubs` 

where:

-  `org.springframework`  is the  `groupId` .

-  `myApp`  is the  `artifactId` .

-  `stubs`  is the classifier. (Note that  `stubs`  is the default value.)

Because  `stubs`  is the default classifier, the preceding example is equal to the following example:

`stubs: org.springframework:myApp` 

## 76.4 Configuring Spring Cloud Zookeeper Dependencies

You can set the following properties to enable or disable parts of Zookeeper Dependencies functionalities:

-  `spring.cloud.zookeeper.dependencies` : If you do not set this property, you cannot use Zookeeper Dependencies.

-  `spring.cloud.zookeeper.dependency.ribbon.enabled`  (enabled by default): Ribbon requires either explicit global configuration or a particular one for a dependency. By turning on this property, runtime load balancing strategy resolution is possible, and you can use the  `loadBalancerType`  section of the Zookeeper Dependencies. The configuration that needs this property has an implementation of  `LoadBalancerClient`  that delegates to the  `ILoadBalancer`  presented in the next bullet.

-  `spring.cloud.zookeeper.dependency.ribbon.loadbalancer`  (enabled by default): Thanks to this property, the custom  `ILoadBalancer`  knows that the part of the URI passed to Ribbon might actually be the alias that has to be resolved to a proper path in Zookeeper. Without this property, you cannot register applications under nested paths.

-  `spring.cloud.zookeeper.dependency.headers.enabled`  (enabled by default): This property registers a  `RibbonClient`  that automatically appends appropriate headers and content types with their versions, as presented in the Dependency configuration. Without this setting, those two parameters do not work.

-  `spring.cloud.zookeeper.dependency.resttemplate.enabled`  (enabled by default): When enabled, this property modifies the request headers of a  `@LoadBalanced` -annotated  `RestTemplate`  such that it passes headers and content type with the version set in dependency configuration. Without this setting, those two parameters do not work.


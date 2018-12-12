## 76. Zookeeper依赖关系

以下主题介绍了如何使用Spring Cloud Zookeeper依赖项：

- [Section 76.1, “Using the Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-using)

- [Section 76.2, “Activating Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-activating)

- [Section 76.3, “Setting up Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-setting-up)

- [Section 76.4, “Configuring Spring Cloud Zookeeper Dependencies”](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-configuring)

## 76.1使用Zookeeper依赖项

Spring Cloud Zookeeper使您可以将应用程序的依赖项作为属性提供.作为依赖项，您可以了解在Zookeeper中注册的其他应用程序以及您希望通过[Feign](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-feign)（REST客户端构建器）和[Spring RestTemplate](https://github.com/spring-cloud/spring-cloud-netflix/blob/master/docs/src/main/asciidoc/spring-cloud-netflix.adoc#spring-cloud-ribbon)调用的应用程序.

您还可以使用Zookeeper Dependency Watchers功能来控制和监视依赖项的状态.

## 76.2激活Zookeeper依赖项

包含对 `org.springframework.cloud:spring-cloud-starter-zookeeper-discovery` 的依赖性可启用设置Spring Cloud Zookeeper依赖关系的自动配置.即使您在属性中提供依赖项，也可以关闭依赖项.为此，请将 `spring.cloud.zookeeper.dependency.enabled` 属性设置为false（默认为 `true` ）.

## 76.3设置Zookeeper依赖项

请考虑以下依赖关系表示示例：

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

接下来的几节将逐一介绍依赖关系的每个部分.根属性名称是 `spring.cloud.zookeeper.dependencies` .

### 76.3.1别名

在root属性下面，您必须将每个依赖项表示为别名.这是由于Ribbon的限制，它要求将应用程序ID放在URL中.因此，您无法通过任何复杂的路径，例如 `/myApp/myRoute/name` ）.别名是您使用的名称，而不是 `DiscoveryClient` ， `Feign` 或 `RestTemplate` 的 `serviceId` .

在前面的示例中，别名是 `newsletter` 和 `mailing` .以下示例显示带有 `newsletter` 别名的Feign用法：

```java
@FeignClient("newsletter")
public interface NewsletterService {
@RequestMapping(method = RequestMethod.GET, value = "/newsletter")
String getNewsletters();
}
```

### 76.3.2路径

该路径由 `path`  YAML属性表示，并且是在Zookeeper下注册依赖项的路径.如[previous section](multi_spring-cloud-zookeeper-dependencies.html#spring-cloud-zookeeper-dependencies-setting-up-aliases)中所述，功能区对URL进行操作.因此，此路径不符合其要求.这就是为什么Spring Cloud Zookeeper将别名映射到正确的路径.

### 76.3.3负载均衡器类型

负载均衡器类型由 `loadBalancerType`  YAML属性表示.

如果您知道在调用此特定依赖项时必须应用哪种负载平衡策略，则可以在YAML文件中提供它，并自动应用它.您可以选择以下负载平衡策略之一：

- STICKY：选择后，始终调用实例.

- RANDOM：随机选取一个实例.

- ROUND_ROBIN：一遍又一遍地迭代实例.

### 76.3.4内容类型模板和版本

`Content-Type` 模板和版本由 `contentTypeTemplate` 和 `version`  YAML属性表示.

如果您在 `Content-Type` 标头中对API进行版本控制，则不希望将此标头添加到每个请求中.此外，如果您想调用新版本的API，则不希望在代码中漫游以提升API版本.这就是为什么你可以为 `contentTypeTemplate` 提供一个特殊的 `$version` 占位符.该占位符将由 `version`  YAML属性的值填充.考虑以下 `contentTypeTemplate` 的示例：

```java
application/vnd.newsletter.$version+json
```

进一步考虑以下 `version` ：

```java
v1
```

`contentTypeTemplate` 和version的组合导致为每个请求创建 `Content-Type` 标头，如下所示：

```java
application/vnd.newsletter.v1+json
```

### 76.3.5默认Headers

默认标头由YAML中的 `headers` Map表示.

有时，每次调用依赖项都需要设置一些默认标头.若要不在代码中执行此操作，可以在YAML文件中进行设置，如以下示例 `headers` 部分所示：

```java
headers:
Accept:
- text/html
- application/xhtml+xml
Cache-Control:
- no-cache
```

该 `headers` 部分导致在HTTP请求中添加 `Accept` 和 `Cache-Control` 标头以及适当的值列表.

### 76.3.6必需的依赖项

所需的依赖项由YAML中的 `required` 属性表示.

如果在应用程序引导时需要启用某个依赖项，则可以在YAML文件中设置 `required: true` 属性.

如果您的应用程序无法在引导期间本地化所需的依赖项，则会引发异常，并且Spring Context无法设置.换句话说，如果未在Zookeeper中注册所需的依赖项，则无法启动应用程序.

您可以阅读有关Spring Cloud Zookeeper Presence Checker [later in this document](multi_spring-cloud-zookeeper-dependency-watcher.html#spring-cloud-zookeeper-dependency-watcher-presence-checker)的更多信息.

### 76.3.7存根

您可以为包含依赖项存根的JAR提供冒号分隔的路径，如以下示例所示：

`stubs: org.springframework:myApp:stubs` 

哪里：

-  `org.springframework` 是 `groupId` .

-  `myApp` 是 `artifactId` .

-  `stubs` 是分类器. （注意 `stubs` 是默认值.）

因为 `stubs` 是默认分类器，所以前面的示例等于以下示例：

`stubs: org.springframework:myApp` 

## 76.4配置Spring Cloud Zookeeper依赖项

您可以设置以下内容用于启用或禁用Zookeeper依赖项功能部分的属性：

-  `spring.cloud.zookeeper.dependencies` ：如果未设置此属性，则无法使用Zookeeper依赖项.

-  `spring.cloud.zookeeper.dependency.ribbon.enabled` （默认情况下启用）：功能区需要显式全局配置或特定的配置才能用于依赖项.通过启用此属性，可以进行运行时负载平衡策略解析，并且可以使用Zookeeper依赖关系的 `loadBalancerType` 部分.需要此属性的配置具有 `LoadBalancerClient` 的实现，该实现委托给下一个项目符号中显示的 `ILoadBalancer` .

-  `spring.cloud.zookeeper.dependency.ribbon.loadbalancer` （默认情况下启用）：由于此属性，自定义 `ILoadBalancer` 知道传递给Ribbon的URI部分实际上可能是必须在Zookeeper中解析为正确路径的别名.如果没有此属性，则无法在嵌套路径下注册应用程序.

-  `spring.cloud.zookeeper.dependency.headers.enabled` （默认情况下启用）：此属性注册 `RibbonClient` ，它会自动在其版本中附加适当的标头和内容类型，如依赖关系配置中所示.没有此设置，这两个参数不起作用.

-  `spring.cloud.zookeeper.dependency.resttemplate.enabled` （默认情况下启用）：启用后，此属性会修改 `@LoadBalanced` -annotated  `RestTemplate` 的请求标头，以便它在依赖项配置中使用版本集传递标头和内容类型.没有此设置，这两个参数不起作用.


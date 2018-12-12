## 8.嵌入配置服务器

Config Server作为独立应用程序运行最佳.但是，如果需要，您可以将其嵌入另一个应用程序中.为此，请使用 `@EnableConfigServer` 注释.在这种情况下，名为 `spring.cloud.config.server.bootstrap` 的可选属性很有用.它是一个标志，指示服务器是否应从其自己的远程存储库配置自身.默认情况下，该标志处于关闭状态，因为它可能会延迟启动.但是，当嵌入到另一个应用程序中时，以与任何其他应用程序相同的方式初始化是有意义的.将 `spring.cloud.config.server.bootstrap` 设置为 `true` 时，还必须使用[composite environment repository configuration](multi__spring_cloud_config_server.html#composite-environment-repositories).例如

```java
spring:
application:
name: configserver
profiles:
active: composite
cloud:
config:
server:
composite:
- type: native
search-locations: ${HOME}/Desktop/config
bootstrap: true
```

> 如果使用bootstrap标志，则配置服务器需要在 `bootstrap.yml` 中配置其名称和存储库URI.

要更改服务器endpoints的位置，您可以（可选）设置 `spring.cloud.config.server.prefix` （例如， `/config` ），以便为前缀下的资源提供服务.前缀应该开始但不以 `/` 结束.它应用于Config Server中的 `@RequestMappings` （即Spring Boot  `server.servletPath` 和 `server.contextPath` 前缀下面）.

如果您想直接从后端存储库（而不是从配置服务器）读取应用程序的配置，您基本上是嵌入式配置服务器没有endpoints.您可以通过不使用 `@EnableConfigServer` 注释（设置 `spring.cloud.config.server.bootstrap=true` ）完全关闭endpoints.

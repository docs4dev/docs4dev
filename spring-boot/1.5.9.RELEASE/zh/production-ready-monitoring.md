## 54.通过HTTP进行监控和管理

如果您正在开发Web应用程序，则Spring Boot Actuator会自动配置所有已启用的endpoints以通过HTTP公开.默认约定是使用endpoints `id` ，前缀为 `/actuator` 作为URL路径.例如， `health` 公开为 `/actuator/health` .提示：Spring MVC，Spring WebFlux和Jersey本身支持Actuator.

## 54.1自定义管理endpoints路径

有时，自定义管理endpoints的前缀很有用.例如，您的应用程序可能已将 `/actuator` 用于其他目的.您可以使用 `management.endpoints.web.base-path` 属性更改管理endpoints的前缀，如以下示例所示：

```java
management.endpoints.web.base-path=/manage
```

前面的 `application.properties` 示例将endpoints从 `/actuator/{id}` 更改为 `/manage/{id}` （例如， `/manage/info` ）.

> 除非管理端口已配置为[expose endpoints by using a different HTTP port](production-ready-monitoring.html#production-ready-customizing-management-server-port)， `management.endpoints.web.base-path` 相对于 `server.servlet.context-path` .如果配置了 `management.server.port` ， `management.endpoints.web.base-path` 相对于 `management.server.servlet.context-path` .

如果要将endpoints映射到其他路径，可以使用 `management.endpoints.web.path-mapping` 属性.

以下示例将 `/actuator/health` 重新映射为 `/healthcheck` ：

**application.properties.** 

```java
management.endpoints.web.base-path=/
management.endpoints.web.path-mapping.health=healthcheck
```

## 54.2自定义管理服务器端口

使用默认HTTP端口公开管理endpoints是基于Cloud的部署的明智选择.但是，如果你的应用程序在您自己的数据中心内运行，您可能更喜欢使用不同的HTTP端口公开endpoints.

您可以设置 `management.server.port` 属性以更改HTTP端口，如以下示例所示：

```java
management.server.port=8081
```

## 54.3配置特定于管理的SSL

配置为使用自定义端口时，还可以使用各种 `management.server.ssl.*` 属性为管理服务器配置自己的SSL.例如，这样做可以让主应用程序使用HTTPS时管理服务器通过HTTP可用，如以下属性设置所示：

```java
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:store.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=false
```

或者，主服务器和管理服务器都可以使用SSL但具有不同的密钥库，如下所示：

```java
server.port=8443
server.ssl.enabled=true
server.ssl.key-store=classpath:main.jks
server.ssl.key-password=secret
management.server.port=8080
management.server.ssl.enabled=true
management.server.ssl.key-store=classpath:management.jks
management.server.ssl.key-password=secret
```

## 54.4自定义管理服务器地址

您可以通过设置 `management.server.address` 属性来自定义管理endpoints可用的地址.如果您只想在内部或面向操作的网络上侦听或仅侦听来自 `localhost` 的连接，那么这样做会非常有用.

> 只有当端口与主服务器端口不同时，才能侦听不同的地址.

以下示例 `application.properties` 不允许远程管理连接：

```java
management.server.port=8081
management.server.address=127.0.0.1
```

## 54.5禁用HTTPendpoints

如果您不想通过HTTP公开endpoints，可以将管理端口设置为 `-1` ，如以下示例所示：

```java
management.server.port=-1
```

这也可以使用 `management.endpoints.web.exposure.exclude` 属性来实现，如下例所示：

```java
management.endpoints.web.exposure.exclude=*
```


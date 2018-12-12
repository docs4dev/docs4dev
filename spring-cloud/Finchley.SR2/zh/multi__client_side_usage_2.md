## 102.客户端使用情况

要在应用程序中使用这些功能，只需将其构建为依赖于 `spring-cloud-vault-config` 的Spring Boot应用程序（例如，请参阅测试用例）.示例Maven配置：

**Example 102.1. pom.xml** 

```xml
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>2.0.0.RELEASE</version>
<relativePath /> <!-- lookup parent from repository -->
</parent>

<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-vault-config</artifactId>
<version>Finchley.SR2</version>
</dependency>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-test</artifactId>
<scope>test</scope>
</dependency>
</dependencies>

<build>
<plugins>
<plugin>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
</plugins>
</build>

<!-- repositories also needed for snapshots and milestones -->
```

然后，您可以创建一个标准的Spring Boot应用程序，就像这个简单的HTTP服务器：

```java
@SpringBootApplication
@RestController
public class Application {

@RequestMapping("/")
public String home() {
return "Hello World!";
}

public static void main(String[] args) {
SpringApplication.run(Application.class, args);
}
}
```

运行时，如果它正在运行，它将从端口 `8200` 上的默认本地Vault服务器获取外部配置.要修改启动行为，您可以使用 `bootstrap.properties` 更改Vault服务器的位置（如 `application.properties` ，但对于应用程序上下文的引导阶段），例如，

**Example 102.2. bootstrap.yml** 

```java
spring.cloud.vault:
host: localhost
port: 8200
scheme: https
uri: https://localhost:8200
connection-timeout: 5000
read-timeout: 15000
config:
order: -10
```

-  `host` 设置Vault主机的主机名.主机名将用于SSL证书验证

-  `port` 设置Vault端口

-  `scheme` 将方案设置为 `http` 将使用纯HTTP.支持的方案是 `http` 和 `https` .

-  `uri` 使用URI配置Vaultendpoints.优先于主机/端口/方案配置

-  `connection-timeout` 设置连接超时（以毫秒为单位）

-  `read-timeout` 设置读取超时（以毫秒为单位）

-  `config.order` 设置属性源的顺序

启用进一步的集成需要额外的依赖性和配置.根据您设置Vault的方式，您可能需要其他配置，例如[SSL](https://cloud.spring.io/spring-cloud-vault/spring-cloud-vault.html#vault.config.ssl)和[authentication](https://cloud.spring.io/spring-cloud-vault/spring-cloud-vault.html#vault.config.authentication).

如果应用程序导入 `spring-boot-starter-actuator` 项目，则可通过 `/health` endpoints获取Vault服务器的状态.

可以通过属性 `management.health.vault.enabled` （默认为 `true` ）启用或禁用Vault运行状况指示器.

## 102.1身份验证

保险柜需要[authentication mechanism](https://www.vaultproject.io/docs/concepts/auth.html)至[authorize client requests](https://www.vaultproject.io/docs/concepts/tokens.html).

Spring Cloud Vault支持多个[authentication mechanisms](https://cloud.spring.io/spring-cloud-vault/spring-cloud-vault.html#vault.config.authentication)使用Vault对应用程序进行身份验证.

对于快速入门，请使用[Vault initialization](multi__quick_start_4.html#quickstart.vault.start)打印的根令牌.

**Example 102.3. bootstrap.yml** 

```java
spring.cloud.vault:
token: 19aefa97-cccc-bbbb-aaaa-225940e63d76
```

> 仔细考虑您的安全要求.如果您想快速开始使用Vault，静态令牌身份验证很好，但静态令牌不会受到任何进一步的保护.对非预期方的任何披露都允许Vault与相关的令牌角色一起使用.


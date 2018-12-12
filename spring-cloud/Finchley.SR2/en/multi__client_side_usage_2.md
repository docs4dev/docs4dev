## 102. Client Side Usage

To use these features in an application, just build it as a Spring Boot application that depends on  `spring-cloud-vault-config`  (e.g. see the test cases). Example Maven configuration:

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

Then you can create a standard Spring Boot application, like this simple HTTP server:

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

When it runs it will pick up the external configuration from the default local Vault server on port  `8200`  if it is running. To modify the startup behavior you can change the location of the Vault server using  `bootstrap.properties`  (like  `application.properties`  but for the bootstrap phase of an application context), e.g.

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

-  `host`  sets the hostname of the Vault host. The host name will be used for SSL certificate validation

-  `port`  sets the Vault port

-  `scheme`  setting the scheme to  `http`  will use plain HTTP. Supported schemes are  `http`  and  `https` .

-  `uri`  configure the Vault endpoint with an URI. Takes precedence over host/port/scheme configuration

-  `connection-timeout`  sets the connection timeout in milliseconds

-  `read-timeout`  sets the read timeout in milliseconds

-  `config.order`  sets the order for the property source

Enabling further integrations requires additional dependencies and configuration. Depending on how you have set up Vault you might need additional configuration like [SSL](https://cloud.spring.io/spring-cloud-vault/spring-cloud-vault.html#vault.config.ssl) and [authentication](https://cloud.spring.io/spring-cloud-vault/spring-cloud-vault.html#vault.config.authentication).

If the application imports the  `spring-boot-starter-actuator`  project, the status of the vault server will be available via the  `/health`  endpoint.

The vault health indicator can be enabled or disabled through the property  `management.health.vault.enabled`  (default to  `true` ).

## 102.1 Authentication

Vault requires an [authentication mechanism](https://www.vaultproject.io/docs/concepts/auth.html) to [authorize client requests](https://www.vaultproject.io/docs/concepts/tokens.html).

Spring Cloud Vault supports multiple [authentication mechanisms](https://cloud.spring.io/spring-cloud-vault/spring-cloud-vault.html#vault.config.authentication) to authenticate applications with Vault.

For a quickstart, use the root token printed by the [Vault initialization](multi__quick_start_4.html#quickstart.vault.start).

**Example 102.3. bootstrap.yml** 

```java
spring.cloud.vault:
token: 19aefa97-cccc-bbbb-aaaa-225940e63d76
```

> Consider carefully your security requirements. Static token authentication is fine if you want quickly get started with Vault, but a static token is not protected any further. Any disclosure to unintended parties allows Vault use with the associated token roles.


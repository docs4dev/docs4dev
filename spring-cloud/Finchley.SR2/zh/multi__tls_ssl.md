## 117. TLS / SSL

网关可以通过遵循通常的Spring服务器配置来监听https上的请求.例：

**application.yml.** 

```java
server:
ssl:
enabled: true
key-alias: scg
key-store-password: scg1234
key-store: classpath:scg-keystore.p12
key-store-type: PKCS12
```

网关路由可以路由到http和https后端.如果路由到https后端，则可以将网关配置为信任具有以下配置的所有下游证书：

**application.yml.** 

```java
spring:
cloud:
gateway:
httpclient:
ssl:
useInsecureTrustManager: true
```

使用不安全的信任管理器不适合生产环境.对于生产环境部署，可以使用以下配置配置一组可信任的已知证书：

**application.yml.** 

```java
spring:
cloud:
gateway:
httpclient:
ssl:
trustedX509Certificates:
- cert1.pem
- cert2.pem
```

如果Spring Cloud Gateway未配置可信证书，则使用默认信任库（可以使用系统属性javax.net.ssl.trustStore覆盖）.

## 117.1 TLS握手

网关维护一个客户端池，用于路由到后端.通过https进行通信时，客户端会启动TLS握手.这次握手会有很多超时.可以配置这些超时（显示默认值）：

**application.yml.** 

```java
spring:
cloud:
gateway:
httpclient:
ssl:
handshake-timeout-millis: 10000
close-notify-flush-timeout-millis: 3000
close-notify-read-timeout-millis: 0
```


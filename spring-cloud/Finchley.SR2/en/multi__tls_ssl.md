## 117. TLS / SSL

The Gateway can listen for requests on https by following the usual Spring server configuration. Example:

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

Gateway routes can be routed to both http and https backends. If routing to a https backend then the Gateway can be configured to trust all downstream certificates with the following configuration:

**application.yml.**  

```java
spring:
cloud:
gateway:
httpclient:
ssl:
useInsecureTrustManager: true
```

Using an insecure trust manager is not suitable for production. For a production deployment the Gateway can be configured with a set of known certificates that it can trust with the follwing configuration:

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

If the Spring Cloud Gateway is not provisioned with trusted certificates the default trust store is used (which can be overriden with system property javax.net.ssl.trustStore).

## 117.1 TLS Handshake

The Gateway maintains a client pool that it uses to route to backends. When communicating over https the client initiates a TLS handshake. A number of timeouts are assoicated with this handshake. These timeouts can be configured (defaults shown):

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


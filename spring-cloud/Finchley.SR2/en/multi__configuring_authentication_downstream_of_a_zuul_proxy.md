## 85. Configuring Authentication Downstream of a Zuul Proxy

You can control the authorization behaviour downstream of an  `@EnableZuulProxy`  through the  `proxy.auth.*`  settings. Example:

**application.yml.**  

```java
proxy:
auth:
routes:
customers: oauth2
stores: passthru
recommendations: none
```

In this example the "customers" service gets an OAuth2 token relay, the "stores" service gets a passthrough (the authorization header is just passed downstream), and the "recommendations" service has its authorization header removed. The default behaviour is to do a token relay if there is a token available, and passthru otherwise.

See [ProxyAuthenticationProperties](https://github.com/spring-cloud/spring-cloud-security/tree/master/src/main/java/org/springframework/cloud/security/oauth2/proxy/ProxyAuthenticationProperties) for full details.

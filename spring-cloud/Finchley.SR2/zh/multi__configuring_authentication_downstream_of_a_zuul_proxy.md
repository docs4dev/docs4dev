## 85.在Zuul代理的下游配置身份验证

您可以通过 `proxy.auth.*` 设置控制 `@EnableZuulProxy` 下游的授权行为.例：

**application.yml.** 

```java
proxy:
auth:
routes:
customers: oauth2
stores: passthru
recommendations: none
```

在此示例中，“customers”服务获取OAuth2令牌中继，“stores”服务获得直通（授权标头仅传递到下游），“推荐”服务已删除其授权标头.如果有可用的令牌，则默认行为是执行令牌中继，否则执行passthru.

有关详细信息，请参阅[ProxyAuthenticationProperties](https://github.com/spring-cloud/spring-cloud-security/tree/master/src/main/java/org/springframework/cloud/security/oauth2/proxy/ProxyAuthenticationProperties).

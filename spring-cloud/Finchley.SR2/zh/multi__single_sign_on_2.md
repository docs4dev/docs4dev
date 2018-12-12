## 87.单点登录

> 所有OAuth2 SSO和资源服务器功能都在1.3版本中移至Spring Boot.您可以在[Spring Boot user guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)中找到文档.

此项目提供从CloudFoundry服务凭据到Spring Boot功能的自动绑定.例如，如果您有一个名为"sso"的CloudFoundry服务，其凭据包含"client_id"，"client_secret"和"auth_domain"，它将自动绑定到您使用 `@EnableOAuth2Sso` （来自Spring Boot）启用的Spring OAuth2客户端.可以使用 `spring.oauth2.sso.serviceId` 参数化服务的名称.

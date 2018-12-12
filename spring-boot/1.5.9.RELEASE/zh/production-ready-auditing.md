## 58.审计

一旦Spring Security发挥作用，Spring Boot Actuator就会有一个灵活的审计框架来发布事件（默认情况下，“身份验证成功”，“失败”和“访问被拒绝”例外）.此功能对于报告和基于身份验证失败实施锁定策略非常有用.要自定义已发布的安全事件，您可以提供自己的 `AbstractAuthenticationAuditListener` 和 `AbstractAuthorizationAuditListener` 实现.

您还可以将审计服务用于您自己的业务事件.为此，请将现有的 `AuditEventRepository` 注入您自己的组件并直接使用它或使用Spring  `ApplicationEventPublisher` 发布 `AuditApplicationEvent` （通过实现 `ApplicationEventPublisherAware` ）.

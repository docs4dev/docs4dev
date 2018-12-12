## 58. Auditing

Once Spring Security is in play, Spring Boot Actuator has a flexible audit framework that publishes events (by default, “authentication success”, “failure” and “access denied” exceptions). This feature can be very useful for reporting and for implementing a lock-out policy based on authentication failures. To customize published security events, you can provide your own implementations of  `AbstractAuthenticationAuditListener`  and  `AbstractAuthorizationAuditListener` .

You can also use the audit services for your own business events. To do so, either inject the existing  `AuditEventRepository`  into your own components and use that directly or publish an  `AuditApplicationEvent`  with the Spring  `ApplicationEventPublisher`  (by implementing  `ApplicationEventPublisherAware` ).

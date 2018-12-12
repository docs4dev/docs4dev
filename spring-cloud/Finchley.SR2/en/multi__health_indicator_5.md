## 34. Health Indicator

Spring Cloud Stream provides a health indicator for binders. It is registered under the name  `binders`  and can be enabled or disabled by setting the  `management.health.binders.enabled`  property.

By default  `management.health.binders.enabled`  is set to  `false` . Setting  `management.health.binders.enabled`  to  `true`  enables the health indicator, allowing you to access the  `/health`  endpoint to retrieve the binder health indicators.

Health indicators are binder-specific and certain binder implementations may not necessarily provide a health indicator.

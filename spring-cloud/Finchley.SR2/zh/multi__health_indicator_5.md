## 34.Health指标

Spring Cloud Stream为Binders提供Health指示器.它以 `binders` 的名称注册，可以通过设置 `management.health.binders.enabled` 属性来启用或禁用.

默认情况下 `management.health.binders.enabled` 设置为 `false` .将 `management.health.binders.enabled` 设置为 `true` 将启用运行状况指示器，允许您访问 `/health` endpoints以检索Binders运行状况指示器.

Health指标是特定于Binders的，某些Binders实施可能不一定提供Health指标.

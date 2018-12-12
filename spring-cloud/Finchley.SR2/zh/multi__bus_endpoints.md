## 41.总线endpoints

Spring Cloud Bus提供两个endpoints `/actuator/bus-refresh` 和 `/actuator/bus-env` ，分别对应Spring Cloud Commons中的各个Actuatorendpoints， `/actuator/refresh` 和 `/actuator/env` .

## 41.1总线刷新endpoints

`/actuator/bus-refresh` endpoints清除 `RefreshScope` 缓存并重新绑定 `@ConfigurationProperties` .有关更多信息，请参见[Refresh Scope](multi__spring_cloud_context_application_context_services.html#refresh-scope)文档.

要公开 `/actuator/bus-refresh` endpoints，您需要将以下配置添加到您的应用程序：

```java
management.endpoints.web.exposure.include=bus-refresh
```

## 41.2总线Envendpoints

`/actuator/bus-env` endpoints在多个实例中使用指定的键/值对更新每个实例环境.

要公开 `/actuator/bus-env` endpoints，您需要将以下配置添加到您的应用程序：

```java
management.endpoints.web.exposure.include=bus-env
```

`/actuator/bus-env` endpoints接受具有以下形状的 `POST` 请求：

```java
{
	"name": "key1",
	"value": "value1"
}
```


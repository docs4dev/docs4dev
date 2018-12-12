## 56.Logger

Spring Boot Actuator包括在运行时查看和配置应用程序日志级别的功能.您可以查看整个列表或单个Logger的配置，该配置由显式配置的日志记录级别以及日志记录框架为其提供的有效日志记录级别组成.这些级别可以是以下之一：

-  `TRACE` 

-  `DEBUG` 

-  `INFO` 

-  `WARN` 

-  `ERROR` 

-  `FATAL` 

-  `OFF` 

-  `null` 

`null` 表示没有显式配置.

## 56.1配置Logger

要配置给定的Logger， `POST` 是资源URI的部分实体，如以下示例所示：

```java
{
	"configuredLevel": "DEBUG"
}
```

> “重置”Logger的特定级别（并使用默认配置），可以将 `null` 的值作为 `configuredLevel` 传递.


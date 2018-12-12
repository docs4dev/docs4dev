## 55.对JMX的监控和管理

Java Management Extensions（JMX）提供了一种监视和管理应用程序的标准机制.默认情况下，Spring Boot将管理endpoints公开为 `org.springframework.boot` 域下的JMX MBean.

## 55.1自定义MBean名称

MBean的名称通常是从endpoints的 `id` 生成的.例如， `health` endpoints显示为 `org.springframework.boot:type=Endpoint,name=Health` .

如果您的应用程序包含多个Spring  `ApplicationContext` ，您可能会发现名称发生冲突.要解决此问题，可以将 `spring.jmx.unique-names` 属性设置为 `true` ，以便MBean名称始终是唯一的.

您还可以自定义公开endpoints的JMX域.以下设置显示了在 `application.properties` 中执行此操作的示例：

```java
spring.jmx.unique-names=true
management.endpoints.jmx.domain=com.example.myapp
```

## 55.2禁用JMXendpoints

如果您不想通过JMX公开endpoints，可以将 `management.endpoints.jmx.exposure.exclude` 属性设置为 `*` ，如以下示例所示：

```java
management.endpoints.jmx.exposure.exclude=*
```

## 55.3通过HTTP使用Jolokia for JMX

Jolokia是一个JMX-HTTP桥，它提供了一种访问JMX bean的替代方法.要使用Jolokia，请包含对 `org.jolokia:jolokia-core` 的依赖项.例如，使用Maven，您将添加以下依赖项：

```xml
<dependency>
	<groupId>org.jolokia</groupId>
	<artifactId>jolokia-core</artifactId>
</dependency>
```

然后可以通过向 `management.endpoints.web.exposure.include` 属性添加 `jolokia` 或 `*` 来公开Jolokiaendpoints.然后，您可以在管理HTTP服务器上使用 `/actuator/jolokia` 来访问它.

### 55.3.1自定义Jolokia

Jolokia有许多设置，您可以通过设置servlet参数来进行传统配置.使用Spring Boot，您可以使用 `application.properties` 文件.为此，请在参数前加上 `management.endpoint.jolokia.config.` ，如以下示例所示：

```java
management.endpoint.jolokia.config.debug=true
```

### 55.3.2禁用Jolokia

如果您使用Jolokia但不希望Spring Boot配置它，请将 `management.endpoint.jolokia.enabled` 属性设置为 `false` ，如下所示：

```java
management.endpoint.jolokia.enabled=false
```


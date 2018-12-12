## 47.广播自己的活动

总线可以承载 `RemoteApplicationEvent` 类型的任何事件.默认传输是JSON，反序列化器需要知道将提前使用哪些类型.要注册新类型，必须将其放在 `org.springframework.cloud.bus.event` 的子包中.

要自定义事件名称，可以在自定义类上使用 `@JsonTypeName` 或依赖默认策略，即使用类的简单名称.

> 生产环境者和消费者需要访问类定义.

## 47.1在自定义包中注册事件

如果您不能或不想为自定义事件使用 `org.springframework.cloud.bus.event` 的子包，则必须使用 `@RemoteApplicationEventScan` 注释指定要扫描 `RemoteApplicationEvent` 类型事件的包.使用 `@RemoteApplicationEventScan` 指定的包包含子包.

例如，请考虑以下自定义事件，称为 `MyEvent` ：

```java
package com.acme;

public class MyEvent extends RemoteApplicationEvent {
...
}
```

您可以通过以下方式使用反序列化器注册该事件：

```java
package com.acme;

@Configuration
@RemoteApplicationEventScan
public class BusConfiguration {
...
}
```

如果不指定值，则会注册使用 `@RemoteApplicationEventScan` 的类的包.在此示例中， `com.acme` 使用 `BusConfiguration` 包注册.

您还可以使用 `@RemoteApplicationEventScan` 上的 `value` ， `basePackages` 或 `basePackageClasses` 属性显式指定要扫描的包，如以下示例所示：

```java
package com.acme;

@Configuration
//@RemoteApplicationEventScan({"com.acme", "foo.bar"})
//@RemoteApplicationEventScan(basePackages = {"com.acme", "foo.bar", "fizz.buzz"})
@RemoteApplicationEventScan(basePackageClasses = BusConfiguration.class)
public class BusConfiguration {
...
}
```

所有前面的 `@RemoteApplicationEventScan` 示例都是等效的，因为通过在 `@RemoteApplicationEventScan` 上显式指定包来注册 `com.acme` 包.

> 您可以指定要扫描的多个基本包.


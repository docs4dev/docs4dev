## 25.个人档案

Spring Profiles提供了一种隔离应用程序配置部分并使其仅在特定环境中可用的方法.任何 `@Component` 或 `@Configuration` 都可以用 `@Profile` 标记，以便在加载时加以限制，如下例所示：

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {

	// ...

}
```

您可以使用 `spring.profiles.active`   `Environment` 属性指定哪些配置文件处于活动状态.您可以使用本章前面介绍的任何方法指定属性.例如，您可以将其包含在 `application.properties` 中，如以下示例所示：

```java
spring.profiles.active=dev,hsqldb
```

您还可以使用以下开关在命令行上指定它： `--spring.profiles.active=dev,hsqldb` .

## 25.1添加活动配置文件

`spring.profiles.active` 属性遵循与其他属性相同的排序规则：最高 `PropertySource` 获胜.这意味着您可以使用命令行开关在 `application.properties` 中指定活动配置文件，然后在 **replace** 中指定活动配置文件.

有时，将特定于配置文件的属性 **add** 用于活动配置文件而不是替换它们是有用的.  `spring.profiles.include` 属性可用于无条件地添加活动配置文件.  `SpringApplication` 入口点还有一个用于设置其他配置文件的Java API（即，在 `spring.profiles.active` 属性激活的配置文件之上）.请参阅[SpringApplication](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/SpringApplication.html)中的 `setAdditionalProfiles()` 方法.

例如，当使用开关 `--spring.profiles.active=prod` 运行具有以下属性的应用程序时，也会激活 `proddb` 和 `prodmq` 配置文件：

```java
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
- proddb
- prodmq
```

> 请注意，可以在YAML文档中定义 `spring.profiles` 属性，以确定配置中何时包含此特定文档.有关详细信息，请参阅[Section 77.7, “Change Configuration Depending on the Environment”](howto-properties-and-configuration.html#howto-change-configuration-depending-on-the-environment).

## 25.2以编程方式设置配置文件

您可以在应用程序运行之前通过调用 `SpringApplication.setAdditionalProfiles(…)` 以编程方式设置活动配置文件.也可以使用Spring的 `ConfigurableEnvironment` 界面激活配置文件.

## 25.3特定于配置文件的配置文件

特定于配置文件的 `application.properties` （或 `application.yml` ）变体和通过 `@ConfigurationProperties` 引用的文件被视为文件并已加载.有关详细信息，请参阅“[Section 24.4, “Profile-specific Properties”](boot-features-external-config.html#boot-features-external-config-profile-specific-properties)”.


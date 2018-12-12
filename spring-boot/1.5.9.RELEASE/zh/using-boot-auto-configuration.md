## 16.自动配置

Spring Boot自动配置尝试根据您添加的jar依赖项自动配置Spring应用程序.例如，如果 `HSQLDB` 在您的类路径上，并且您尚未手动配置任何数据库连接bean，则Spring Boot会自动配置内存数据库.

您需要通过向 `@Configuration` 类之一添加 `@EnableAutoConfiguration` 或 `@SpringBootApplication` 注释来选择自动配置.

> 您应该只添加一个 `@SpringBootApplication` 或 `@EnableAutoConfiguration` 注释.我们通常建议您仅将一个或另一个添加到主 `@Configuration` 类中.

## 16.1逐渐更换自动配置

自动配置是非侵入性的.在任何时候，您都可以开始定义自己的配置以替换自动配置的特定部分.例如，如果添加自己的 `DataSource`  bean，则默认的嵌入式数据库支持会退回.

如果您需要了解当前正在应用的自动配置以及原因，请使用 `--debug` 开关启动应用程序.这样做可以为选择的核心Logger启用调试日志，并将条件报告记录到控制台.

## 16.2禁用特定的自动配置类

如果您发现不需要的特定自动配置类在应用时，您可以使用 `@EnableAutoConfiguration` 的exclude属性禁用它们，如以下示例所示：

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

如果类不在类路径中，则可以使用注释的 `excludeName` 属性并指定完全限定名称.最后，您还可以使用 `spring.autoconfigure.exclude` 属性控制要排除的自动配置类列表.

> 您可以在注释级别和使用属性定义排除项.


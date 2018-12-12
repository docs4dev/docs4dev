## 16. Auto-configuration

Spring Boot auto-configuration attempts to automatically configure your Spring application based on the jar dependencies that you have added. For example, if  `HSQLDB`  is on your classpath, and you have not manually configured any database connection beans, then Spring Boot auto-configures an in-memory database.

You need to opt-in to auto-configuration by adding the  `@EnableAutoConfiguration`  or  `@SpringBootApplication`  annotations to one of your  `@Configuration`  classes.

> You should only ever add one  `@SpringBootApplication`  or  `@EnableAutoConfiguration`  annotation. We generally recommend that you add one or the other to your primary  `@Configuration`  class only.

## 16.1 Gradually Replacing Auto-configuration

Auto-configuration is non-invasive. At any point, you can start to define your own configuration to replace specific parts of the auto-configuration. For example, if you add your own  `DataSource`  bean, the default embedded database support backs away.

If you need to find out what auto-configuration is currently being applied, and why, start your application with the  `--debug`  switch. Doing so enables debug logs for a selection of core loggers and logs a conditions report to the console.

## 16.2 Disabling Specific Auto-configuration Classes

If you find that specific auto-configuration classes that you do not want are being applied, you can use the exclude attribute of  `@EnableAutoConfiguration`  to disable them, as shown in the following example:

```java
import org.springframework.boot.autoconfigure.*;
import org.springframework.boot.autoconfigure.jdbc.*;
import org.springframework.context.annotation.*;

@Configuration
@EnableAutoConfiguration(exclude={DataSourceAutoConfiguration.class})
public class MyConfiguration {
}
```

If the class is not on the classpath, you can use the  `excludeName`  attribute of the annotation and specify the fully qualified name instead. Finally, you can also control the list of auto-configuration classes to exclude by using the  `spring.autoconfigure.exclude`  property.

> You can define exclusions both at the annotation level and by using the property.


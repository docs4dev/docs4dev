## 18.使用@SpringBootApplication批注

许多Spring Boot开发人员喜欢他们的应用程序使用自动配置，组件扫描并能够在"application class"上定义额外的配置.单个 `@SpringBootApplication` 注释可用于启用这三个功能，即：

-  `@EnableAutoConfiguration` ：启用[Spring Boot’s auto-configuration mechanism](using-boot-auto-configuration.html)

-  `@ComponentScan` ：在应用程序所在的包上启用 `@Component` 扫描（请参阅[the best practices](using-boot-structuring-your-code.html)）

-  `@Configuration` ：允许在上下文中注册额外的bean或导入其他配置类

`@SpringBootApplication` 注释等效于使用 `@Configuration` ， `@EnableAutoConfiguration` 和 `@ComponentScan` 及其默认属性，如以下示例所示：

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

>  `@SpringBootApplication` 还提供别名来自定义 `@EnableAutoConfiguration` 和 `@ComponentScan` 的属性.

> 这些功能中的任何一个都是必需的，您可以选择通过它启用的任何功能替换此单个注释.例如，您可能不希望在应用程序中使用组件扫描：


## 15.配置类

Spring Boot支持基于Java的配置.虽然可以将 `SpringApplication` 与XML源一起使用，但我们通常建议您的主要源是单个 `@Configuration` 类.通常，定义 `main` 方法的类是主 `@Configuration` 的良好候选者.

> 许多Spring配置示例已在Internet上发布，使用XML配置.如果可能，请始终尝试使用等效的基于Java的配置.搜索 `Enable*` 注释可能是一个很好的起点.

## 15.1导入其他配置类

您无需将所有 `@Configuration` 放入单个类中.  `@Import` 注释可用于导入其他配置类.或者，您可以使用 `@ComponentScan` 自动获取所有Spring组件，包括 `@Configuration` 类.

## 15.2导入XML配置

如果您绝对必须使用基于XML的配置，我们建议您仍然使用 `@Configuration` 类.然后，您可以使用 `@ImportResource` 批注来加载XML配置文件.


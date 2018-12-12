## 90.热插拔

Spring Boot支持热交换.本节回答有关其工作原理的问题.

## 90.1重新加载静态内容

热重新加载有几种选择.建议的方法是使用[spring-boot-devtools](using-boot-devtools.html)，因为它提供了额外的开发时间功能，例如支持快速应用程序重启和LiveReload以及合理的开发时配置（例如模板缓存）. Devtools通过监视类路径进行更改来工作.这意味着静态资源更改必须为"built"才能使更改生效.默认情况下，当您保存更改时，这会在Eclipse中自动发生.在IntelliJ IDEA中，Make Project命令会触发必要的构建.由于[default restart exclusions](using-boot-devtools.html#using-boot-devtools-restart-exclude)，对静态资源的更改不会触发应用程序的重新启动.但是，它们会触发实时重新加载.

或者，在IDE中运行（特别是在调试时）是进行开发的好方法（所有现代IDE都允许重新加载静态资源，并且通常还允许热插拔Java类更改）.

最后，可以配置[Maven and Gradle plugins](build-tool-plugins.html)（参见 `addResources` 属性）以支持从命令行运行，并直接从源重新加载静态文件.如果您使用更高级别的工具编写该代码，则可以将其与外部css / js编译器进程一起使用.

## 90.2重新加载模板而不重新启动容器

Spring Boot支持的大多数模板技术都包含一个配置选项禁用缓存（在本文档后面介绍）.如果使用 `spring-boot-devtools` 模块，则在开发时这些属性为[automatically configured](using-boot-devtools.html#using-boot-devtools-property-defaults).

### 90.2.1 Thymeleaf模板

如果您使用Thymeleaf，请将 `spring.thymeleaf.cache` 设置为 `false` .有关其他Thymeleaf自定义选项，请参阅[ThymeleafAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java).

### 90.2.2 FreeMarker模板

如果您使用FreeMarker，请将 `spring.freemarker.cache` 设置为 `false` .有关其他FreeMarker自定义选项，请参阅[FreeMarkerAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java).

### 90.2.3 Groovy模板

如果使用Groovy模板，请将 `spring.groovy.template.cache` 设置为 `false` .有关其他Groovy自定义选项，请参阅[GroovyTemplateAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java).

## 90.3快速申请重启

`spring-boot-devtools` 模块包括对自动应用程序重启的支持.虽然没有像[JRebel](http://zeroturnaround.com/software/jrebel/)这样的技术那么快，但它通常比“冷启动”快得多.在调查本文档后面讨论的一些更复杂的重载选项之前，您应该尝试一下.

有关更多详细信息，请参阅[Chapter 20, Developer Tools](using-boot-devtools.html)部分.

## 90.4重新加载Java类而不重新启动容器

许多现代IDE（Eclipse，IDEA和其他）支持字节码的热交换.因此，如果您进行的更改不会影响类或方法签名，则应该干净地重新加载，不会产生任何副作用.


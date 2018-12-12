## 88.Actuator

Spring Boot包括Spring Boot Actuator.本节回答了其使用中经常出现的问题.

## 88.1更改Actuatorendpoints的HTTP端口或地址

在独立应用程序中，Actuator HTTP端口默认与主HTTP端口相同.要使应用程序侦听其他端口，请设置外部属性： `management.server.port` .要侦听完全不同的网络地址（例如，当您有用于管理的内部网络和用于用户应用程序的外部网络时），您还可以将 `management.server.address` 设置为服务器能够绑定的有效IP地址.

有关更多详细信息，请参阅“生产环境就绪功能”部分中的[ManagementServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/web/server/ManagementServerProperties.java)源代码和“[Section 54.2, “Customizing the Management Server Port”](production-ready-monitoring.html#production-ready-customizing-management-server-port)”.

## 88.2自定义'whitelabel'错误页面

Spring Boot安装了一个如果遇到服务器错误，您在浏览器客户端中看到的“whitelabel”错误页面（使用JSON和其他媒体类型的计算机客户端应该看到具有正确错误代码的合理响应）.

> Set  `server.error.whitelabel.enabled=false` 关闭默认错误页面.这样做会恢复您正在使用的servlet容器的默认值.请注意，Spring Boot仍会尝试解析错误视图，因此您应该添加自己的错误页面，而不是完全禁用它.

使用您自己的错误页面覆盖错误页面取决于您使用的模板技术.例如，如果您使用Thymeleaf，则可以添加 `error.html` 模板.如果您使用FreeMarker，则可以添加 `error.ftl` 模板.通常，您需要 `View` ，其解析名称为 `error` 或 `@Controller` 处理 `/error` 路径.除非你替换了一些默认配置，否则你应该在 `ApplicationContext` 中找到 `BeanNameViewResolver` ，所以名为 `error` 的 `@Bean` 将是一种简单的方法.有关更多选项，请参阅[ErrorMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.java).

有关如何在servlet容器中注册处理程序的详细信息，另请参阅“[Error Handling](boot-features-developing-web-applications.html#boot-features-error-handling)”一节.

## 88.3消除明智的Value观

`env` 和 `configprops` endpoints返回的信息可能有些敏感，因此默认情况下会清除匹配某个模式的密钥（即它们的值将被 `******` 替换）.

Spring Boot为这些键使用合理的默认值：例如，任何以"password"，"secret"，"key"或"token"结尾的键都会被清理.也可以使用正则表达式，例如 `*credentials.*` 来清理任何包含单词 `credentials` 作为键的一部分的键.

可以使用 `management.endpoint.env.keys-to-sanitize` 和 `management.endpoint.configprops.keys-to-sanitize` 分别自定义要使用的模式.


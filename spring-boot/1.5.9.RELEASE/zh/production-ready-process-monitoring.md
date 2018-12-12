## 60.过程监控

在 `spring-boot` 模块中，您可以找到两个类来创建通常对进程监视有用的文件：

-  `ApplicationPidFileWriter` 创建一个包含应用程序PID的文件（默认情况下，在文件名为 `application.pid` 的应用程序目录中）.

-  `WebServerPortFileWriter` 创建包含正在运行的Web服务器端口的文件（默认情况下，在文件名为 `application.port` 的应用程序目录中）.

默认情况下，这些编写器未激活，但您可以启用：

- [By Extending Configuration](production-ready-process-monitoring.html#production-ready-process-monitoring-configuration)

- [Section 60.2, “Programmatically”](production-ready-process-monitoring.html#production-ready-process-monitoring-programmatically)

## 60.1扩展配置

在 `META-INF/spring.factories` 文件中，您可以激活写入PID文件的侦听器，如以下示例所示：

```java
org.springframework.context.ApplicationListener=\
org.springframework.boot.context.ApplicationPidFileWriter,\
org.springframework.boot.web.context.WebServerPortFileWriter
```

## 60.2以编程方式

您还可以通过调用 `SpringApplication.addListeners(…)` 方法并传递相应的 `Writer` 对象来激活侦听器.此方法还允许您在 `Writer` 构造函数中自定义文件名和路径.


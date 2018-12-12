## 44.对JMX的监控和管理

Java Management Extensions（JMX）提供了一种监视和管理应用程序的标准机制.默认情况下，Spring Boot会创建一个ID为 `mbeanServer` 的 `MBeanServer`  bean，并公开使用Spring JMX注释（ `@ManagedResource` ， `@ManagedAttribute` 或 `@ManagedOperation` ）注释的任何bean.

有关详细信息，请参阅[JmxAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.java)类.

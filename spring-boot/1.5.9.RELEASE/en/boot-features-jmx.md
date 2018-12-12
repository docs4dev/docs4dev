## 44. Monitoring and Management over JMX

Java Management Extensions (JMX) provide a standard mechanism to monitor and manage applications. By default, Spring Boot creates an  `MBeanServer`  bean with an ID of  `mbeanServer`  and exposes any of your beans that are annotated with Spring JMX annotations ( `@ManagedResource` ,  `@ManagedAttribute` , or  `@ManagedOperation` ).

See the [JmxAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/jmx/JmxAutoConfiguration.java) class for more details.

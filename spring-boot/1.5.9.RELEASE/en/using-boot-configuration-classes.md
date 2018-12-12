## 15. Configuration Classes

Spring Boot favors Java-based configuration. Although it is possible to use  `SpringApplication`  with XML sources, we generally recommend that your primary source be a single  `@Configuration`  class. Usually the class that defines the  `main`  method is a good candidate as the primary  `@Configuration` .

> Many Spring configuration examples have been published on the Internet that use XML configuration. If possible, always try to use the equivalent Java-based configuration. Searching for  `Enable*`  annotations can be a good starting point.

## 15.1 Importing Additional Configuration Classes

You need not put all your  `@Configuration`  into a single class. The  `@Import`  annotation can be used to import additional configuration classes. Alternatively, you can use  `@ComponentScan`  to automatically pick up all Spring components, including  `@Configuration`  classes.

## 15.2 Importing XML Configuration

If you absolutely must use XML based configuration, we recommend that you still start with a  `@Configuration`  class. You can then use an  `@ImportResource`  annotation to load XML configuration files.


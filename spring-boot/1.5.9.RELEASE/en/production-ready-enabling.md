## 52. Enabling Production-ready Features

The [spring-boot-actuator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator) module provides all of Spring Boot’s production-ready features. The simplest way to enable the features is to add a dependency to the  `spring-boot-starter-actuator`  ‘Starter’.

----
**Definition of Actuator** 

An actuator is a manufacturing term that refers to a mechanical device for moving or controlling something. Actuators can generate a large amount of motion from a small change.

----

To add the actuator to a Maven based project, add the following ‘Starter’ dependency:

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
</dependencies>
```

For Gradle, use the following declaration:

```java
dependencies {
	compile("org.springframework.boot:spring-boot-starter-actuator")
}
```

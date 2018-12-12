## 52.启用生产环境就绪功能

[spring-boot-actuator](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator)模块提供Spring Boot的所有生产环境就绪功能.启用这些功能的最简单方法是向 `spring-boot-starter-actuator` 'Starter'添加依赖项.

----
**Definition of Actuator** 

致动器是制造术语，指的是用于移动或控制某物的机械装置.Actuator可以通过微小的变化产生大量的运动.

----

要将Actuator添加到基于Maven的项目，请添加以下“Starter”依赖项：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
</dependencies>
```

对于Gradle，请使用以下声明：

```java
dependencies {
	compile("org.springframework.boot:spring-boot-starter-actuator")
}
```

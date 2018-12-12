## 71. Spring Boot Maven插件

[Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin)在Maven中提供Spring Boot支持，允许您打包可执行jar或war档案并“就地”运行应用程序.要使用它，您必须使用Maven 3.2（或更高版本）.

> 查看[Spring Boot Maven Plugin Site](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin)以获取完整的插件文档.

## 71.1包括插件

要使用Spring Boot Maven插件，请在 `pom.xml` 的 `plugins` 部分中包含相应的XML，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>
	<!-- ... -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
				<version>2.1.0.RELEASE</version>
				<executions>
					<execution>
						<goals>
							<goal>repackage</goal>
						</goals>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>
</project>
```

上述配置重新打包在Maven生命周期的 `package` 阶段构建的jar或war.以下示例显示了重新打包的jar以及 `target` 目录中的原始jar：

```java
$ mvn package
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```

如果您不包含 `<execution/>` 配置，如前面的示例所示，您可以单独运行插件（但仅在使用包目标时），如以下示例所示：

```java
$ mvn package spring-boot:repackage
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```

如果使用里程碑或快照版本，则还需要添加相应的 `pluginRepository` 元素，如下面的清单所示：

```xml
<pluginRepositories>
	<pluginRepository>
		<id>spring-snapshots</id>
		<url>https://repo.spring.io/snapshot</url>
	</pluginRepository>
	<pluginRepository>
		<id>spring-milestones</id>
		<url>https://repo.spring.io/milestone</url>
	</pluginRepository>
</pluginRepositories>
```

## 71.2包装可执行的jar和战争文件

一旦 `spring-boot-maven-plugin` 包含在 `pom.xml` 中，它就会自动尝试使用 `spring-boot:repackage` 目标重写存档以使其可执行.您应该使用通常的 `packaging` 元素将项目配置为构建jar或war（视情况而定），如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<!-- ... -->
	<packaging>jar</packaging>
	<!-- ... -->
</project>
```

在 `package` 阶段，Spring Boot会增强您现有的存档.您可以通过使用配置选项或通常在清单中添加 `Main-Class` 属性来指定要启动的主类办法.如果未指定主类，则插件将使用 `public static void main(String[] args)` 方法搜索类.

要构建和运行项目工件，可以键入以下内容：

```java
$ mvn package
$ java -jar target/mymodule-0.0.1-SNAPSHOT.jar
```

要构建可执行且可部署到外部容器的war文件，需要将嵌入式容器依赖项标记为“已提供”，如以下示例所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<!-- ... -->
	<packaging>war</packaging>
	<!-- ... -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-tomcat</artifactId>
			<scope>provided</scope>
		</dependency>
		<!-- ... -->
	</dependencies>
</project>
```

> 有关如何创建可部署的war文件的更多详细信息，请参阅“[Section 92.1, “Create a Deployable War File”](howto-traditional-deployment.html#howto-create-a-deployable-war-file)”部分.

[plugin info page](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin)中提供了高级配置选项和示例.


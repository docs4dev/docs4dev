## 71. Spring Boot Maven Plugin

The [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin) provides Spring Boot support in Maven, letting you package executable jar or war archives and run an application “in-place”. To use it, you must use Maven 3.2 (or later).

> See the [Spring Boot Maven Plugin Site](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin) for complete plugin documentation.

## 71.1 Including the Plugin

To use the Spring Boot Maven Plugin, include the appropriate XML in the  `plugins`  section of your  `pom.xml` , as shown in the following example:

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

The preceding configuration repackages a jar or war that is built during the  `package`  phase of the Maven lifecycle. The following example shows both the repackaged jar as well as the original jar in the  `target`  directory:

```java
$ mvn package
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```

If you do not include the  `<execution/>`  configuration, as shown in the prior example, you can run the plugin on its own (but only if the package goal is used as well), as shown in the following example:

```java
$ mvn package spring-boot:repackage
$ ls target/*.jar
target/myproject-1.0.0.jar target/myproject-1.0.0.jar.original
```

If you use a milestone or snapshot release, you also need to add the appropriate  `pluginRepository`  elements, as shown in the following listing:

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

## 71.2 Packaging Executable Jar and War Files

Once  `spring-boot-maven-plugin`  has been included in your  `pom.xml` , it automatically tries to rewrite archives to make them executable by using the  `spring-boot:repackage`  goal. You should configure your project to build a jar or war (as appropriate) by using the usual  `packaging`  element, as shown in the following example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<!-- ... -->
	<packaging>jar</packaging>
	<!-- ... -->
</project>
```

Your existing archive is enhanced by Spring Boot during the  `package`  phase. The main class that you want to launch can be specified either by using a configuration option or by adding a  `Main-Class`  attribute to the manifest in the usual way. If you do not specify a main class, the plugin searches for a class with a  `public static void main(String[] args)`  method.

To build and run a project artifact, you can type the following:

```java
$ mvn package
$ java -jar target/mymodule-0.0.1-SNAPSHOT.jar
```

To build a war file that is both executable and deployable into an external container, you need to mark the embedded container dependencies as “provided”, as shown in the following example:

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

> See the “[Section 92.1, “Create a Deployable War File”](howto-traditional-deployment.html#howto-create-a-deployable-war-file)” section for more details on how to create a deployable war file.

Advanced configuration options and examples are available in the [plugin info page](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin).


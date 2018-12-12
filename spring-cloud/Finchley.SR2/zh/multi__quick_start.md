## 4.快速入门

快速入门使用Spring Cloud Config Server的服务器和客户端.

首先，启动服务器，如下所示：

```java
$ cd spring-cloud-config-server
$ ../mvnw spring-boot:run
```

服务器是一个Spring Boot应用程序，因此如果您愿意，可以从IDE运行它（主类是 `ConfigServerApplication` ）.

接下来尝试一个客户端，如下所示：

```java
$ curl localhost:8888/foo/development
{"name":"foo","label":"master","propertySources":[
{"name":"https://github.com/scratches/config-repo/foo-development.properties","source":{"bar":"spam"}},
{"name":"https://github.com/scratches/config-repo/foo.properties","source":{"foo":"bar"}}
]}
```

查找属性源的默认策略是克隆git存储库（在 `spring.cloud.config.server.git.uri` ）并使用它来初始化mini  `SpringApplication` .迷你应用程序的 `Environment` 用于枚举属性源并在JSONendpoints发布它们.

HTTP服务具有以下形式的资源：

```java
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

其中 `application` 作为 `SpringApplication` 注入 `SpringApplication` （常规Spring Boot应用程序中通常是 `application` ）， `profile` 是活动配置文件（或以逗号分隔的属性列表）， `label` 是可选的git标签（默认为 `master` . ）

Spring Cloud Config Server从git存储库（必须提供）中提取远程客户端的配置，如以下示例所示：

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
```

## 4.1客户端使用情况

要在应用程序中使用这些功能，可以将其构建为依赖于spring-cloud-config-client的Spring Boot应用程序（例如，请参阅config-client或示例应用程序的测试用例）.添加依赖项最方便的方法是使用Spring Boot starter  `org.springframework.cloud:spring-cloud-starter-config` . Maven用户还有一个父pom和BOM（ `spring-cloud-starter-parent` ），以及Gradle和Spring CLI用户的Spring IO版本管理属性文件.以下示例显示了典型的Maven配置：

**pom.xml.** 

```xml
<parent>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-parent</artifactId>
<version>{spring-boot-docs-version}</version>
<relativePath /> <!-- lookup parent from repository -->
</parent>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>{spring-cloud-version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>

<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-test</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>

<build>
	<plugins>
<plugin>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-maven-plugin</artifactId>
</plugin>
	</plugins>
</build>

<!-- repositories also needed for snapshots and milestones -->
```

现在您可以创建一个标准的Spring Boot应用程序，例如以下HTTP服务器：

```java
@SpringBootApplication
@RestController
public class Application {

@RequestMapping("/")
public String home() {
return "Hello World!";
}

public static void main(String[] args) {
SpringApplication.run(Application.class, args);
}

}
```

当此HTTP服务器运行时，它从端口8888上的默认本地配置服务器（如果它正在运行）中获取外部配置.要修改启动行为，可以使用 `bootstrap.properties` 更改配置服务器的位置（类似于 `application.properties` 但对于应用程序上下文的引导阶段），如以下示例所示：

```java
spring.cloud.config.uri: http://myconfigserver.com
```

引导属性在 `/env` endpoints中显示为高优先级属性源，如以下示例所示.

```java
$ curl localhost:8080/env
{
"profiles":[],
"configService:https://github.com/spring-cloud-samples/config-repo/bar.properties":{"foo":"bar"},
"servletContextInitParams":{},
"systemProperties":{...},
...
}
```

名为 ```configService:<URL of remote repository>/<file name>` 的属性源包含 `foo` 属性，其值为 `bar` 并且具有最高优先级.

> 属性源名称中的URL是git存储库，而不是配置服务器URL.


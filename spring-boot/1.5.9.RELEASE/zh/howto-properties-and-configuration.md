## 77.属性和配置

本节包括有关设置和读取属性和配置设置及其与Spring Boot应用程序交互的主题.

## 77.1在构建时自动展开属性

您可以使用现有的构建配置自动扩展它们，而不是硬编码在项目的构建配置中也指定的某些属性.这在Maven和Gradle都是可行的.

### 77.1.1使用Maven自动扩展属性

您可以使用资源过滤从Maven项目自动扩展属性.如果使用 `spring-boot-starter-parent` ，则可以使用 `@[email protected]` 占位符引用Maven的“项目属性”，如以下示例所示：

```java
app.encoding[emailprotected]@
app.java.version[emailprotected]@
```

> 仅以这种方式过滤生产环境配置（换句话说，不对 `src/test/resources` 应用过滤）.

> 如果启用 `addResources` 标志， `spring-boot:run` 目标可以直接将 `src/main/resources` 添加到类路径中（用于热重新加载）.这样做可以绕过资源过滤和此功能.相反，您可以使用 `exec:java` 目标或自定义插件的配置.有关详细信息，请参阅[plugin usage page](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/usage.html).

如果您不使用starter父级，则需要在 `pom.xml` 的 `<build/>` 元素中包含以下元素：

```xml
<resources>
	<resource>
		<directory>src/main/resources</directory>
		<filtering>true</filtering>
	</resource>
</resources>
```

您还需要在 `<plugins/>` 中包含以下元素：

```xml
<plugin>
	<groupId>org.apache.maven.plugins</groupId>
	<artifactId>maven-resources-plugin</artifactId>
	<version>2.7</version>
	<configuration>
		<delimiters>
			<delimiter>@</delimiter>
		</delimiters>
		<useDefaultDelimiters>false</useDefaultDelimiters>
	</configuration>
</plugin>
```

> 如果在配置中使用标准的Spring占位符（例如 `${placeholder}` ），则 `useDefaultDelimiters` 属性很重要.如果该属性未设置为 `false` ，则可以通过构建扩展这些属性.

### 77.1.2使用Gradle自动扩展属性

您可以通过配置Java插件的 `processResources` 任务来自动扩展Gradle项目中的属性，如以下示例所示：

```java
processResources {
	expand(project.properties)
}
```

然后，您可以使用占位符来引用Gradle项目的属性，如以下示例所示：

```java
app.name=${name}
app.description=${description}
```

> Gradle的 `expand` 方法使用Groovy的 `SimpleTemplateEngine` ，它可以转换 `${..}` 标记.  `${..}` 样式与Spring自己的属性占位符机制冲突.要将Spring属性占位符与自动扩展一起使用，请按如下方式转义Spring属性占位符： `\${..}` .

## 77.2外部化SpringApplication的配置

`SpringApplication` 具有bean属性（主要是setter），因此您可以在创建应用程序时使用其Java API来修改其行为.或者，您可以通过在 `spring.main.*` 中设置属性来外部化配置.例如，在 `application.properties` 中，您可能具有以下设置：

```java
spring.main.web-application-type=none
spring.main.banner-mode=off
```

然后在启动时不打印Spring BootBanner，并且应用程序未启动嵌入式Web服务器.

外部配置中定义的属性会覆盖使用Java API指定的值，但用于创建 `ApplicationContext` 的源的明显例外.考虑以下应用程序：

```java
new SpringApplicationBuilder()
	.bannerMode(Banner.Mode.OFF)
	.sources(demo.MyApp.class)
	.run(args);
```

现在考虑以下配置：

```java
spring.main.sources=com.acme.Config,com.acme.ExtraConfig
spring.main.banner-mode=console
```

实际应用程序现在显示Banner（由配置覆盖）并使用三个源 `ApplicationContext` （按以下顺序）： `demo.MyApp` ， `com.acme.Config` 和 `com.acme.ExtraConfig` .

## 77.3更改应用程序外部属性的位置

默认情况下，来自不同源的属性以定义的顺序添加到Spring  `Environment` （有关确切顺序，请参阅“Spring Boot features”部分中的“[Chapter 24, Externalized Configuration](boot-features-external-config.html)”）.

增加和修改此排序的一种好方法是在应用程序中添加 `@PropertySource` 注释源.传递给 `SpringApplication` 静态便捷方法的类和使用 `setSources()` 添加的类将被检查以查看它们是否具有 `@PropertySources` .如果他们这样做，那么这些属性会尽早添加到 `Environment` ，以便在 `ApplicationContext` 生命周期的所有阶段中使用.以这种方式添加的属性的优先级低于使用默认位置（例如 `application.properties` ），系统属性，环境变量或命令行添加的属性.

您还可以提供以下系统属性（或环境变量）来更改行为：

-  `spring.config.name` （ `SPRING_CONFIG_NAME` ）：默认为 `application` 作为文件名的根.

-  `spring.config.location` （ `SPRING_CONFIG_LOCATION` ）：要加载的文件（例如类路径资源或URL）.为此文档设置了单独的 `Environment` 属性源，它可以被系统属性，环境变量或命令行覆盖.

无论您在环境中设置什么，Spring Boot始终如上所述加载 `application.properties` .默认情况下，如果使用YAML，则扩展名为“.yml”的文件也会添加到列表中.

Spring Boot记录在 `DEBUG` 级别加载的配置文件以及在 `TRACE` 级别找不到的候选项.

有关详细信息，请参阅[ConfigFileApplicationListener](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot/src/main/java/org/springframework/boot/context/config/ConfigFileApplicationListener.java).

## 77.4使用“短”命令行参数

有些人喜欢使用（例如） `--port=9000` 而不是 `--server.port=9000` 来在命令行上设置配置属性.您可以通过在 `application.properties` 中使用占位符来启用此行为，如以下示例所示：

```java
server.port=${port:8080}
```

> 如果从 `spring-boot-starter-parent`  POM继承， `maven-resources-plugins` 的默认过滤器令牌已从 `${*}` 更改为 `@` （即 `@[email protected]` 而不是 `${maven.token}` ），以防止与Spring样式占位符冲突.如果您已直接为 `application.properties` 启用了Maven过滤，则可能还需要更改默认过滤器令牌以使用[other delimiters](https://maven.apache.org/plugins/maven-resources-plugin/resources-mojo.html#delimiters).

> 在这种特定情况下，端口绑定可在Paoku环境（如Heroku或Cloud Foundry）中运行.在这两个平台中， `PORT` 环境变量自动设置，Spring可以绑定到 `Environment` 属性的大写同义词.

## 77.5使用YAML作为外部属性

YAML是JSON的超集，因此，它是以分层格式存储外部属性的便捷语法，如以下示例所示：

```java
spring:
	application:
		name: cruncher
	datasource:
		driverClassName: com.mysql.jdbc.Driver
		url: jdbc:mysql://localhost/test
server:
	port: 9000
```

创建一个名为 `application.yml` 的文件并将其放在类路径的根目录中.然后将 `snakeyaml` 添加到您的依赖项（Maven坐标 `org.yaml:snakeyaml` ，如果您使用 `spring-boot-starter` 已经包含）. YAML文件被解析为Java  `Map<String,Object>` （类似于JSON对象），并且Spring Boot将Map展平，使其深度为一级并具有句点分隔键，因为许多人习惯使用Java中的 `Properties` 文件.

前面的示例YAML对应于以下 `application.properties` 文件：

```java
spring.application.name=cruncher
spring.datasource.driverClassName=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost/test
server.port=9000
```

有关YAML的更多信息，请参阅“Spring Boot功能”部分中的“[Section 24.7, “Using YAML Instead of Properties”](boot-features-external-config.html#boot-features-external-config-yaml)”.

## 77.6设置活动spring配置文件

Spring  `Environment` 有一个API，但你通常会设置一个System属性（ `spring.profiles.active` ）或一个OS环境变量（ `SPRING_PROFILES_ACTIVE` ）.此外，您可以使用 `-D` 参数启动应用程序（请记住将其放在主类或jar存档之前），如下所示：

```java
$ java -jar -Dspring.profiles.active=production demo-0.0.1-SNAPSHOT.jar
```

在Spring Boot中，您还可以在 `application.properties` 中设置活动配置文件，如以下示例所示：

```java
spring.profiles.active=production
```

以这种方式设置的值将由System属性或环境变量设置替换，但不会由 `SpringApplicationBuilder.profiles()` 方法替换.因此，后一个Java API可用于扩充配置文件而不更改默认值.

有关详细信息，请参阅“Spring Boot功能”部分中的“[Chapter 25, Profiles](boot-features-profiles.html)”.

## 77.7根据环境更改配置

YAML文件实际上是由 `---` 行分隔的文档序列，每个文档都被单独解析为展平的Map.

如果YAML文档包含 `spring.profiles` 键，则配置文件值（以逗号分隔的配置文件列表）将被输入Spring  `Environment.acceptsProfiles()` 方法.如果这些配置文件中的任何一个处于活动状态，那么该文档将包含在最终合并中（否则，它不会），如以下示例所示：

```java
server:
	port: 9000
---

spring:
	profiles: development
server:
	port: 9001

---

spring:
	profiles: production
server:
	port: 0
```

在前面的示例中，默认端口为9000.但是，如果名为“development”的Spring配置文件处于活动状态，则端口为9001.如果“production”处于活动状态，则端口为0.

>  YAML文档按其遇到的顺序合并.以后的值会覆盖以前的值.

要对属性文件执行相同操作，可以使用 `application-${profile}.properties` 指定特定于配置文件的值.

## 77.8发现外部属性的内置选项

Spring Boot在运行时将外部属性从 `application.properties` （或 `.yml` 文件和其他位置）绑定到应用程序中.没有（并且在技术上不可能）单个位置中所有受支持属性的详尽列表，因为贡献可以来自类路径上的其他jar文件.

具有Actuator功能的正在运行的应用程序具有 `configprops` endpoints，该endpoints显示通过 `@ConfigurationProperties` 可用的所有绑定和可绑定属性.

附录包含一个[application.properties](common-application-properties.html)示例，其中列出了最常见的列表Spring Boot支持的属性.最终列表来自于搜索 `@ConfigurationProperties` 和 `@Value` 注释的源代码以及偶尔使用 `Binder` .有关加载属性的确切顺序的更多信息，请参阅“[Chapter 24, Externalized Configuration](boot-features-external-config.html)”.


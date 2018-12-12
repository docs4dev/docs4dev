## 11.开发您的第一个Spring Boot应用程序

本节介绍如何开发一个简单的“Hello World！”Web应用程序，该应用程序突出了Spring Boot的一些主要功能.我们使用Maven来构建这个项目，因为大多数IDE都支持它.

>  [spring.io](https://spring.io)网站包含许多使用Spring Boot的“入门”[guides](https://spring.io/guides).如果您需要解决特定问题，请先检查一下.

在开始之前，打开终端并运行以下命令以确保安装了有效的Java和Maven版本：

```java
$ java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) 64-Bit Server VM (build 25.102-b14, mixed mode)
```

```java
$ mvn -v
Apache Maven 3.5.4 (1edded0938998edf8bf061f1ceb3cfdeccf443fe; 2018-06-17T14:33:14-04:00)
Maven home: /usr/local/Cellar/maven/3.3.9/libexec
Java version: 1.8.0_102, vendor: Oracle Corporation
```

> T此样本需要在自己的文件夹中创建.后续说明假定您已创建合适的文件夹，并且它是您当前的目录.

## 11.1创建POM

我们需要从创建Maven  `pom.xml` 文件开始.  `pom.xml` 是用于构建项目的配方.打开您喜欢的文本编辑器并添加以下内容：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.0.RELEASE</version>
	</parent>

	<!-- Additional lines to be added here... -->

</project>
```

上面的清单应该为您提供有效的构建.您可以通过运行 `mvn package` 来测试它（现在，您可以忽略“jar将为空 - 没有内容被标记为包含！”警告）.

> 此时，您可以将项目导入IDE（大多数现代Java IDE包括对Maven的内置支持）.为简单起见，我们继续为此示例使用纯文本编辑器.

## 11.2添加类路径依赖项

Spring Boot提供了许多“Starters”，可以将jar添加到类路径中.我们的示例应用程序已经在POM的 `parent` 部分中使用了 `spring-boot-starter-parent` .  `spring-boot-starter-parent` 是一个特殊的启动器，提供有用的Maven默认值.它还提供[dependency-management](using-boot-build-systems.html#using-boot-dependency-management)部分，以便您可以省略 `version` 标记以获得“祝福”的依赖关系.

其他“Starters”提供了在开发特定类型的应用程序时可能需要的依赖项.由于我们正在开发Web应用程序，因此我们添加了 `spring-boot-starter-web` 依赖项.在此之前，我们可以通过运行以下命令来查看当前的内容：

```java
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

`mvn dependency:tree` 命令打印项目依赖项的树表示.您可以看到 `spring-boot-starter-parent` 本身不提供依赖关系.要添加必要的依赖项，请编辑 `pom.xml` 并在 `parent` 部分的正下方添加 `spring-boot-starter-web` 依赖项：

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```

如果再次运行 `mvn dependency:tree` ，您会看到现在有许多其他依赖项，包括Tomcat Web服务器和Spring Boot本身.

## 11.3编写代码

要完成我们的应用程序，我们需要创建一个Java文件.默认情况下，Maven编译来自 `src/main/java` 的源，因此您需要创建该文件夹结构，然后添加名为 `src/main/java/Example.java` 的文件以包含以下代码：

```java
import org.springframework.boot.*;
import org.springframework.boot.autoconfigure.*;
import org.springframework.web.bind.annotation.*;

@RestController
@EnableAutoConfiguration
public class Example {

	@RequestMapping("/")
	String home() {
		return "Hello World!";
	}

	public static void main(String[] args) throws Exception {
		SpringApplication.run(Example.class, args);
	}

}
```

虽然这里的代码不多，但还是有很多代码.我们将在接下来的几节中逐步介绍重要部分.

### 11.3.1 @RestController和@RequestMapping Annotations

我们的 `Example` 类的第一个注释是 `@RestController` .这被称为构造型注释.它为阅读代码的人提供了提示，而对于Spring来说，该类扮演着特定的角色.在这种情况下，我们的类是一个web  `@Controller` ，所以Spring在处理传入的Web请求时会考虑它.

`@RequestMapping` 注释提供“路由”信息.它告诉Spring，任何带有 `/` 路径的HTTP请求都应该映射到 `home` 方法.  `@RestController` 注释告诉Spring将结果字符串直接呈现给调用者.

>   `@RestController` 和 `@RequestMapping` 注释是Spring MVC注释. （它们不是Spring Boot特有的.）有关详细信息，请参阅Spring参考文档中的[MVC section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc).

### 11.3.2 @EnableAutoConfiguration Annotation

第二个类级别注释是 `@EnableAutoConfiguration` .这个注释告诉Spring Boot根据你添加的jar依赖关系“猜测”你想要如何配置Spring.由于 `spring-boot-starter-web` 添加了Tomcat和Spring MVC，因此自动配置假定您正在开发Web应用程序并相应地设置Spring.

----
**Starters and Auto-configuration** 

自动配置旨在与“启动器”配合使用，但这两个概念并不直接相关.您可以自由选择并在首发之外选择jar依赖项. Spring Boot仍然尽力自动配置您的应用程序.

----

### 11.3.3“主要”方法

我们的应用程序的最后一部分是 `main` 方法.这只是遵循应用程序入口点的Java约定的标准方法.我们的main方法通过调用 `run` 委托Spring Boot的 `SpringApplication` 类.  `SpringApplication` 引导我们的应用程序，启动Spring，然后启动自动配置的Tomcat Web服务器.我们需要将 `Example.class` 作为参数传递给 `run` 方法，以告诉 `SpringApplication` 哪个是主要的Spring组件.  `args` 数组也被传递以公开任何命令行参数.

## 11.4运行示例

此时，您的应用程序应该可以工作.由于您使用了 `spring-boot-starter-parent`  POM，因此您可以使用一个有用的 `run` 目标来启动应用程序.从根项目目录中键入 `mvn spring-boot:run` 以启动应用程序.您应该看到类似于以下内容的输出：

```java
$ mvn spring-boot:run

.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::  (v2.1.0.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.222 seconds (JVM running for 6.514)
```

如果您打开Web浏览器 `localhost:8080` ，您应该看到以下输出：

```java
Hello World!
```

要正常退出应用程序，请按 `ctrl-c` .

## 11.5创建一个可执行的Jar

我们通过创建一个完全自包含的可执行jar文件来完成我们的示例，我们可以在生产环境中运行它.可执行jar（有时称为“fat jar”）是包含已编译类以及代码需要运行的所有jar依赖项的归档.

----
**Executable jars and Java** 

Java没有提供加载嵌套jar文件的标准方法（jar文件本身包含在jar中）.如果您要分发自包含的应用程序，这可能会有问题.

为了解决这个问题，许多开发人员使用“超级”jar. uber jar将所有应用程序依赖项中的所有类打包到一个存档中.这种方法的问题在于很难看出应用程序中有哪些库.如果在多个jar中使用相同的文件名（但具有不同的内容），也可能会有问题.

Spring Boot需要一个[different approach](executable-jar.html)，让你直接嵌套jar.

----

要创建可执行jar，我们需要将 `spring-boot-maven-plugin` 添加到 `pom.xml` .为此，请在 `dependencies` 部分下方插入以下行：

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

>   `spring-boot-starter-parent`  POM包含 `<executions>` 配置以绑定 `repackage` 目标.如果您不使用父POM，则需要自己声明此配置.有关详细信息，请参阅[plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/usage.html).

保存 `pom.xml` 并从命令行运行 `mvn package` ，如下所示：

```java
$ mvn package

[INFO] Scanning for projects...
[INFO]
[INFO] ------------------------------------------------------------------------
[INFO] Building myproject 0.0.1-SNAPSHOT
[INFO] ------------------------------------------------------------------------
[INFO] .... ..
[INFO] --- maven-jar-plugin:2.4:jar (default-jar) @ myproject ---
[INFO] Building jar: /Users/developer/example/spring-boot-example/target/myproject-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:2.1.0.RELEASE:repackage (default) @ myproject ---
[INFO] ------------------------------------------------------------------------
[INFO] BUILD SUCCESS
[INFO] ------------------------------------------------------------------------
```

如果你查看 `target` 目录，你应该看到 `myproject-0.0.1-SNAPSHOT.jar` .该文件大小应为10 MB左右.如果要查看内部，可以使用 `jar tvf` ，如下所示：

```java
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

您还应该在 `target` 目录中看到一个名为 `myproject-0.0.1-SNAPSHOT.jar.original` 的小得多的文件.这是Maven在Spring Boot重新打包之前创建的原始jar文件.

要运行该应用程序，请使用 `java -jar` 命令，如下所示：

```java
$ java -jar target/myproject-0.0.1-SNAPSHOT.jar

.   ____          _            __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
\\/  ___)| |_)| | | | | || (_| |  ) ) ) )
'  |____| .__|_| |_|_| |_\__, | / / / /
=========|_|==============|___/=/_/_/_/
:: Spring Boot ::  (v2.1.0.RELEASE)
....... . . .
....... . . . (log output here)
....... . . .
........ Started Example in 2.536 seconds (JVM running for 2.864)
```

和以前一样，要退出应用程序，请按 `ctrl-c` .


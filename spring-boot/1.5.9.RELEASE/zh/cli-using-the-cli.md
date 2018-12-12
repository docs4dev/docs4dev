## 67.使用CLI

安装CLI后，可以通过键入 `spring` 并在命令行按Enter键来运行它.如果在没有任何参数的情况下运行 `spring` ，则会显示一个简单的帮助屏幕，如下所示：

```xml
$ spring
usage: spring [--help] [--version]
<command> [<args>]

Available commands are:

run [options] <files> [--] [args]
Run a spring groovy script

... more command help is shown here
```

您可以键入 `spring help` 以获取有关任何支持的命令的更多详细信息，如以下示例所示：

```xml
$ spring help run
spring run - Run a spring groovy script

usage: spring run [options] <files> [--] [args]

Option                     Description
------                     -----------
--autoconfigure [Boolean]  Add autoconfigure compiler
transformations (default: true)
--classpath, -cp           Additional classpath entries
-e, --edit                 Open the file with the default system
editor
--no-guess-dependencies    Do not attempt to guess dependencies
--no-guess-imports         Do not attempt to guess imports
-q, --quiet                Quiet logging
-v, --verbose              Verbose logging of dependency
resolution
--watch                    Watch the specified file for changes
```

`version` 命令提供了一种快速方法来检查您正在使用的Spring Boot版本，如下所示：

```java
$ spring version
Spring CLI v2.1.0.RELEASE
```

## 67.1使用CLI运行应用程序

您可以使用 `run` 命令编译和运行Groovy源代码. Spring Boot CLI是完全独立的，因此您不需要任何外部Groovy安装.

以下示例显示了使用Groovy编写的“hello world”Web应用程序：

**hello.groovy.** 

```java
@RestController
class WebApplication {

	@RequestMapping("/")
	String home() {
		"Hello World!"
	}

}
```

要编译并运行该应用程序，请键入以下命令：

```java
$ spring run hello.groovy
```

要将命令行参数传递给应用程序，请使用 `--` 将命令与“spring”命令参数分开，如以下示例所示：

```java
$ spring run hello.groovy -- --server.port=9000
```

要设置JVM命令行参数，可以使用 `JAVA_OPTS` 环境变量，如以下示例所示：

```java
$ JAVA_OPTS=-Xmx1024m spring run hello.groovy
```

> 设置 `JAVA_OPTS` 时Microsoft Windows，请务必引用整个指令，例如 `set "JAVA_OPTS=-Xms256m -Xmx2048m"` .这样做可确保将值正确传递给流程.

### 67.1.1扣除了“grab”依赖关系

标准Groovy包含 `@Grab` 注释，它允许您声明对第三方库的依赖性.这个有用的技术让Groovy以与Maven或Gradle相同的方式下载jar，但不需要你使用构建工具.

Spring Boot进一步扩展了这种技术，并尝试根据您的代码推断出“抓取”哪些库.例如，由于前面显示的 `WebApplication` 代码使用了 `@RestController` 注释，因此Spring Boot会抓取"Tomcat"和"Spring MVC".

以下项目用作“抓取提示”：

|资料|收藏|
| ---- | ---- |
|  `JdbcTemplate` ， `NamedParameterJdbcTemplate` ， `DataSource`  | JDBC应用程序. |
|  `@EnableJms`  | JMS应用程序. |
|  `@EnableCaching`  |缓存抽象. |
|  `@Test`  | JUnit. |
|  `@EnableRabbit`  | RabbitMQ. |
| extends  `Specification`  | Spock测试. |
|  `@EnableBatchProcessing`  | Spring Batch. |
|  `@MessageEndpoint`   `@EnableIntegration`  | Spring Integration. |
|  `@Controller`   `@RestController`   `@EnableWebMvc`  | Spring MVC Embedded Tomcat. |
|  `@EnableWebSecurity`  | Spring Security. |
|  `@EnableTransactionManagement`  | Spring Transaction Management. |

> 在Spring Boot CLI源代码中查看[CompilerAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-cli/src/main/java/org/springframework/boot/cli/compiler/CompilerAutoConfiguration.java)的子类，以准确了解自定义的应用方式.

### 67.1.2扣除“grab”坐标

Spring Boot通过允许您指定没有组或版本的依赖项（例如， `@Grab('freemarker')` ）来扩展Groovy的标准 `@Grab` 支持.这样做可以参考Spring Boot的默认依赖关系元数据来推断工件的组和版本.

> 默认元数据与您使用的CLI版本相关联.只有当您移动到新版本的CLI时，它才会更改，让您可以控制依赖项版本何时更改.可以在[appendix](appendix-dependency-versions.html)中找到显示默认元数据中包含的依赖关系及其版本的表.

### 67.1.3默认导入语句

为了帮助减小Groovy代码的大小，自动包含多个 `import` 语句.请注意前面的示例如何引用 `@Component` ， `@RestController` 和 `@RequestMapping` ，而无需使用完全限定名称或 `import` 语句.

> 很多Spring注释在不使用 `import` 语句的情况下工作.尝试运行应用程序以在添加导入之前查看失败的内容.

### 67.1.4自动主要方法

与等效的Java应用程序不同，您不需要在 `Groovy` 脚本中包含 `public static void main(String[] args)` 方法.自动创建 `SpringApplication` ，编译的代码充当 `source` .

### 67.1.5自定义依赖关系管理

默认情况下，CLI在解析 `@Grab` 依赖项时使用 `spring-boot-dependencies` 中声明的依赖关系管理.可以使用 `@DependencyManagementBom` 注释配置覆盖默认依赖关系管理的其他依赖关系管理.注释的值应指定一个或多个Maven BOM的坐标（ `groupId:artifactId:version` ）.

例如，请考虑以下声明：

```java
@DependencyManagementBom("com.example.custom-bom:1.0.0")
```

前面的声明在 `com/example/custom-versions/1.0.0/` 下的Maven存储库中选取 `custom-bom-1.0.0.pom` .

指定多个BOM时，它们将按您声明的顺序应用，如以下示例所示：

```java
@DependencyManagementBom(["com.example.custom-bom:1.0.0",
		"com.example.another-bom:1.0.0"])
```

上面的示例表明 `another-bom` 中的依赖关系管理会覆盖 `custom-bom` 中的依赖关系管理.

您可以在任何可以使用 `@Grab` 的地方使用 `@DependencyManagementBom` .但是，为了确保依赖关系管理的一致排序，您最多可以在应用程序中使用 `@DependencyManagementBom` .一个有用的依赖关系管理源（它是Spring Boot的依赖关系管理的超集）是[Spring IO Platform](https://platform.spring.io/)，您可能包含以下行：

```java
@DependencyManagementBom('io.spring.platform:platform-bom:1.1.2.RELEASE')
```

## 67.2具有多个源文件的应用程序

您可以对所有接受文件输入的命令使用“shell globbing”.这样做可以让您使用单个目录中的多个文件，如以下示例所示：

```java
$ spring run *.groovy
```

## 67.3打包您的应用程序

您可以使用 `jar` 命令将应用程序打包到一个自包含的可执行jar文件中，如以下示例所示：

```java
$ spring jar my-app.jar *.groovy
```

生成的jar包含通过编译应用程序和所有应用程序的依赖项生成的类，以便可以使用 `java -jar` 运行它. jar文件还包含应用程序类路径中的条目.您可以使用 `--include` 和 `--exclude` 添加和删除jar的显式路径.两者都以逗号分隔，并且都以“”和“ - ”的形式接受前缀，以表示它们应该从默认值中删除.默认包括如下：

```java
public/**, resources/**, static/**, templates/**, META-INF/**, *
```

默认排除如下：

```java
.*, repository/**, build/**, target/**, **/*.jar, **/*.groovy
```

在命令行上键入 `spring help jar` 以获取更多信息.

## 67.4初始化一个新项目

`init` 命令允许您在不离开shell的情况下使用[start.spring.io](https://start.spring.io)创建新项目，如以下示例所示：

```java
$ spring init --dependencies=web,data-jpa my-project
Using service at https://start.spring.io
Project extracted to '/Users/developer/example/my-project'
```

上面的示例使用基于Maven的项目创建 `my-project` 目录，该项目使用 `spring-boot-starter-web` 和 `spring-boot-starter-data-jpa` .您可以使用 `--list` 标志列出服务的功能，如以下示例所示：

```java
$ spring init --list
=======================================
Capabilities of https://start.spring.io
=======================================

Available dependencies:
-----------------------
actuator - Actuator: Production ready features to help you monitor and manage your application
...
web - Web: Support for full-stack web development, including Tomcat and spring-webmvc
websocket - Websocket: Support for WebSocket development
ws - WS: Support for Spring Web Services

Available project types:
------------------------
gradle-build -  Gradle Config [format:build, build:gradle]
gradle-project -  Gradle Project [format:project, build:gradle]
maven-build -  Maven POM [format:build, build:maven]
maven-project -  Maven Project [format:project, build:maven] (default)

...
```

`init` 命令支持许多选项.有关详细信息，请参阅 `help` 输出.例如，以下命令创建一个使用Java 8和 `war` 包装的Gradle项目：

```java
$ spring init --build=gradle --java-version=1.8 --dependencies=websocket --packaging=war sample-app.zip
Using service at https://start.spring.io
Content saved to 'sample-app.zip'
```

## 67.5使用嵌入式Shell

Spring Boot包含BASH和zsh shell的命令行完成脚本.如果不使用这些shell中的任何一个（可能是Windows用户），则可以使用 `shell` 命令启动集成shell，如以下示例所示：

```java
$ spring shell
Spring Boot (v2.1.0.RELEASE)
Hit TAB to complete. Type \'help' and hit RETURN for help, and \'exit' to quit.
```

在嵌入式shell中，您可以直接运行其他命令：

```java
$ version
Spring CLI v2.1.0.RELEASE
```

嵌入式shell支持ANSI颜色输出以及 `tab` 完成.如果需要运行本机命令，可以使用 `!` 前缀.要退出嵌入式shell，请按 `ctrl-c` .

## 67.6在CLI中添加扩展

您可以使用 `install` 命令向CLI添加扩展.该命令采用 `group:artifact:version` 格式的一组或多组工件坐标，如以下示例所示：

```java
$ spring install com.example:spring-boot-cli-extension:1.0.0.RELEASE
```

除了安装由您提供的坐标标识的工件外，还会安装所有工件的依赖项.

要卸载依赖项，请使用 `uninstall` 命令.与 `install` 命令一样，它采用 `group:artifact:version` 格式的一组或多组工件坐标，如以下示例所示：

```java
$ spring uninstall com.example:spring-boot-cli-extension:1.0.0.RELEASE
```

它会卸载由您提供的坐标及其依赖项标识的工件.

要卸载所有其他依赖项，可以使用 `--all` 选项，如以下示例所示：

```java
$ spring uninstall --all
```


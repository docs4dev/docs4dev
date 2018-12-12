## 67. Using the CLI

Once you have installed the CLI, you can run it by typing  `spring`  and pressing Enter at the command line. If you run  `spring`  without any arguments, a simple help screen is displayed, as follows:

```xml
$ spring
usage: spring [--help] [--version]
<command> [<args>]

Available commands are:

run [options] <files> [--] [args]
Run a spring groovy script

... more command help is shown here
```

You can type  `spring help`  to get more details about any of the supported commands, as shown in the following example:

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

The  `version`  command provides a quick way to check which version of Spring Boot you are using, as follows:

```java
$ spring version
Spring CLI v2.1.0.RELEASE
```

## 67.1 Running Applications with the CLI

You can compile and run Groovy source code by using the  `run`  command. The Spring Boot CLI is completely self-contained, so you do not need any external Groovy installation.

The following example shows a “hello world” web application written in Groovy:

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

To compile and run the application, type the following command:

```java
$ spring run hello.groovy
```

To pass command-line arguments to the application, use  `--`  to separate the commands from the “spring” command arguments, as shown in the following example:

```java
$ spring run hello.groovy -- --server.port=9000
```

To set JVM command line arguments, you can use the  `JAVA_OPTS`  environment variable, as shown in the following example:

```java
$ JAVA_OPTS=-Xmx1024m spring run hello.groovy
```

> When setting  `JAVA_OPTS`  on Microsoft Windows, make sure to quote the entire instruction, such as  `set "JAVA_OPTS=-Xms256m -Xmx2048m"` . Doing so ensures the values are properly passed to the process.

### 67.1.1 Deduced “grab” Dependencies

Standard Groovy includes a  `@Grab`  annotation, which lets you declare dependencies on third-party libraries. This useful technique lets Groovy download jars in the same way as Maven or Gradle would but without requiring you to use a build tool.

Spring Boot extends this technique further and tries to deduce which libraries to “grab” based on your code. For example, since the  `WebApplication`  code shown previously uses  `@RestController`  annotations, Spring Boot grabs "Tomcat" and "Spring MVC".

The following items are used as “grab hints”:

|Items|Grabs|
|----|----|
| `JdbcTemplate` ,  `NamedParameterJdbcTemplate` ,  `DataSource`  |JDBC Application. |
| `@EnableJms`  |JMS Application. |
| `@EnableCaching`  |Caching abstraction. |
| `@Test`  |JUnit. |
| `@EnableRabbit`  |RabbitMQ. |
|extends  `Specification`  |Spock test. |
| `@EnableBatchProcessing`  |Spring Batch. |
| `@MessageEndpoint`   `@EnableIntegration`  |Spring Integration. |
| `@Controller`   `@RestController`   `@EnableWebMvc`  |Spring MVC + Embedded Tomcat. |
| `@EnableWebSecurity`  |Spring Security. |
| `@EnableTransactionManagement`  |Spring Transaction Management. |

> See subclasses of [CompilerAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-cli/src/main/java/org/springframework/boot/cli/compiler/CompilerAutoConfiguration.java) in the Spring Boot CLI source code to understand exactly how customizations are applied.

### 67.1.2 Deduced “grab” Coordinates

Spring Boot extends Groovy’s standard  `@Grab`  support by letting you specify a dependency without a group or version (for example,  `@Grab('freemarker')` ). Doing so consults Spring Boot’s default dependency metadata to deduce the artifact’s group and version.

> The default metadata is tied to the version of the CLI that you use. it changes only when you move to a new version of the CLI, putting you in control of when the versions of your dependencies may change. A table showing the dependencies and their versions that are included in the default metadata can be found in the [appendix](appendix-dependency-versions.html).

### 67.1.3 Default Import Statements

To help reduce the size of your Groovy code, several  `import`  statements are automatically included. Notice how the preceding example refers to  `@Component` ,  `@RestController` , and  `@RequestMapping`  without needing to use fully-qualified names or  `import`  statements.

> Many Spring annotations work without using  `import`  statements. Try running your application to see what fails before adding imports.

### 67.1.4 Automatic Main Method

Unlike the equivalent Java application, you do not need to include a  `public static void main(String[] args)`  method with your  `Groovy`  scripts. A  `SpringApplication`  is automatically created, with your compiled code acting as the  `source` .

### 67.1.5 Custom Dependency Management

By default, the CLI uses the dependency management declared in  `spring-boot-dependencies`  when resolving  `@Grab`  dependencies. Additional dependency management, which overrides the default dependency management, can be configured by using the  `@DependencyManagementBom`  annotation. The annotation’s value should specify the coordinates ( `groupId:artifactId:version` ) of one or more Maven BOMs.

For example, consider the following declaration:

```java
@DependencyManagementBom("com.example.custom-bom:1.0.0")
```

The preceding declaration picks up  `custom-bom-1.0.0.pom`  in a Maven repository under  `com/example/custom-versions/1.0.0/` .

When you specify multiple BOMs, they are applied in the order in which you declare them, as shown in the following example:

```java
@DependencyManagementBom(["com.example.custom-bom:1.0.0",
		"com.example.another-bom:1.0.0"])
```

The preceding example indicates that the dependency management in  `another-bom`  overrides the dependency management in  `custom-bom` .

You can use  `@DependencyManagementBom`  anywhere that you can use  `@Grab` . However, to ensure consistent ordering of the dependency management, you can use  `@DependencyManagementBom`  at most once in your application. A useful source of dependency management (which is a superset of Spring Boot’s dependency management) is the [Spring IO Platform](https://platform.spring.io/), which you might include with the following line:

```java
@DependencyManagementBom('io.spring.platform:platform-bom:1.1.2.RELEASE')
```

## 67.2 Applications with Multiple Source Files

You can use “shell globbing” with all commands that accept file input. Doing so lets you use multiple files from a single directory, as shown in the following example:

```java
$ spring run *.groovy
```

## 67.3 Packaging Your Application

You can use the  `jar`  command to package your application into a self-contained executable jar file, as shown in the following example:

```java
$ spring jar my-app.jar *.groovy
```

The resulting jar contains the classes produced by compiling the application and all of the application’s dependencies so that it can then be run by using  `java -jar` . The jar file also contains entries from the application’s classpath. You can add and remove explicit paths to the jar by using  `--include`  and  `--exclude` . Both are comma-separated, and both accept prefixes, in the form of “+” and “-”, to signify that they should be removed from the defaults. The default includes are as follows:

```java
public/**, resources/**, static/**, templates/**, META-INF/**, *
```

The default excludes are as follows:

```java
.*, repository/**, build/**, target/**, **/*.jar, **/*.groovy
```

Type  `spring help jar`  on the command line for more information.

## 67.4 Initialize a New Project

The  `init`  command lets you create a new project by using [start.spring.io](https://start.spring.io) without leaving the shell, as shown in the following example:

```java
$ spring init --dependencies=web,data-jpa my-project
Using service at https://start.spring.io
Project extracted to '/Users/developer/example/my-project'
```

The preceding example creates a  `my-project`  directory with a Maven-based project that uses  `spring-boot-starter-web`  and  `spring-boot-starter-data-jpa` . You can list the capabilities of the service by using the  `--list`  flag, as shown in the following example:

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

The  `init`  command supports many options. See the  `help`  output for more details. For instance, the following command creates a Gradle project that uses Java 8 and  `war`  packaging:

```java
$ spring init --build=gradle --java-version=1.8 --dependencies=websocket --packaging=war sample-app.zip
Using service at https://start.spring.io
Content saved to 'sample-app.zip'
```

## 67.5 Using the Embedded Shell

Spring Boot includes command-line completion scripts for the BASH and zsh shells. If you do not use either of these shells (perhaps you are a Windows user), you can use the  `shell`  command to launch an integrated shell, as shown in the following example:

```java
$ spring shell
Spring Boot (v2.1.0.RELEASE)
Hit TAB to complete. Type \'help' and hit RETURN for help, and \'exit' to quit.
```

From inside the embedded shell, you can run other commands directly:

```java
$ version
Spring CLI v2.1.0.RELEASE
```

The embedded shell supports ANSI color output as well as  `tab`  completion. If you need to run a native command, you can use the  `!`  prefix. To exit the embedded shell, press  `ctrl-c` .

## 67.6 Adding Extensions to the CLI

You can add extensions to the CLI by using the  `install`  command. The command takes one or more sets of artifact coordinates in the format  `group:artifact:version` , as shown in the following example:

```java
$ spring install com.example:spring-boot-cli-extension:1.0.0.RELEASE
```

In addition to installing the artifacts identified by the coordinates you supply, all of the artifacts' dependencies are also installed.

To uninstall a dependency, use the  `uninstall`  command. As with the  `install`  command, it takes one or more sets of artifact coordinates in the format of  `group:artifact:version` , as shown in the following example:

```java
$ spring uninstall com.example:spring-boot-cli-extension:1.0.0.RELEASE
```

It uninstalls the artifacts identified by the coordinates you supply and their dependencies.

To uninstall all additional dependencies, you can use the  `--all`  option, as shown in the following example:

```java
$ spring uninstall --all
```


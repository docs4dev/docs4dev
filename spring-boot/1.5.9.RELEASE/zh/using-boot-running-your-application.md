## 19.运行您的应用程序

将应用程序打包为jar并使用嵌入式HTTP服务器的最大优势之一是，您可以像运行任何其他服务器一样运行应用程序.调试Spring Boot应用程序也很容易.您不需要任何特殊的IDE插件或扩展.

> 此部分仅涵盖基于jar的包装.如果您选择将应用程序打包为war文件，则应参阅服务器和IDE文档.

## 19.1从IDE运行

您可以从IDE运行Spring Boot应用程序作为简单的Java应用程序.但是，您首先需要导入项目.导入步骤因IDE和构建系统而异.大多数IDE可以直接导入Maven项目.例如，Eclipse用户可以从 `File` 菜单中选择 `Import…` → `Existing Maven Projects` .

如果无法将项目直接导入IDE，则可以使用构建插件生成IDE元数据. Maven包含[Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/)和[IDEA](https://maven.apache.org/plugins/maven-idea-plugin/)的插件. Gradle为[various IDEs](https://docs.gradle.org/4.2.1/userguide/userguide.html)提供插件.

> 如果您不小心运行了两次Web应用程序，则会看到“端口已在使用中”错误. STS用户可以使用 `Relaunch` 按钮而不是 `Run` 按钮来确保关闭任何现有实例.

## 19.2作为打包应用程序运行

如果使用Spring Boot Maven或Gradle插件创建可执行jar，则可以使用 `java -jar` 运行应用程序，如以下示例所示：

```java
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

也可以运行启用了远程调试支持的打包应用程序.这样做可以将调试器附加到打包的应用程序，如以下示例所示：

```java
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
-jar target/myapplication-0.0.1-SNAPSHOT.jar
```

## 19.3使用Maven插件

Spring Boot Maven插件包含一个 `run` 目标，可用于快速编译和运行您的应用程序.应用程序以分解形式运行，就像在IDE中一样.以下示例显示了运行Spring Boot应用程序的典型Maven命令：

```java
$ mvn spring-boot:run
```

您可能还想使用 `MAVEN_OPTS` 操作系统环境变量，如以下示例所示：

```java
$ export MAVEN_OPTS=-Xmx1024m
```

## 19.4使用Gradle插件

Spring Boot Gradle插件还包含一个 `bootRun` 任务，可用于以爆炸形式运行应用程序.每当您应用 `org.springframework.boot` 和 `java` 插件时都会添加 `bootRun` 任务，如以下示例所示：

```java
$ gradle bootRun
```

您可能还想使用 `JAVA_OPTS` 操作系统环境变量，如以下示例所示：

```java
$ export JAVA_OPTS=-Xmx1024m
```

## 19.5热插拔

由于Spring Boot应用程序只是普通的Java应用程序，因此JVM热交换应该是开箱即用的. JVM热交换在某种程度上受限于它可以替换的字节码.要获得更完整的解决方案，可以使用[JRebel](https://zeroturnaround.com/software/jrebel/).

`spring-boot-devtools` 模块还包括对快速应用程序重启的支持.有关详细信息，请参阅本章后面的[Chapter 20, Developer Tools](using-boot-devtools.html)部分和[Hot swapping “How-to”](howto-hotswapping.html).


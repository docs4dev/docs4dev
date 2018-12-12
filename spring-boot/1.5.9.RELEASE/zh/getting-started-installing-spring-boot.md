## 10.安装Spring Boot

Spring Boot可以与“经典”Java开发工具一起使用，也可以作为命令行工具安装.无论哪种方式，您需要[Java SDK v1.8](https://www.java.com)或更高.在开始之前，您应该使用以下命令检查当前的Java安装：

```java
$ java -version
```

如果您不熟悉Java开发，或者想要尝试Spring Boot，可能需要先尝试[Spring Boot CLI](getting-started-installing-spring-boot.html#getting-started-installing-the-cli)（命令行界面）.否则，请继续阅读“经典”安装说明.

## 10.1 Java Developer的安装说明

您可以像使用Spring Boot一样使用Spring Boot任何标准Java库.为此，请在类路径中包含相应的 `spring-boot-*.jar` 文件. Spring Boot不需要任何特殊工具集成，因此您可以使用任何IDE或文本编辑器.此外，Spring Boot应用程序没有什么特别之处，因此您可以像运行任何其他Java程序一样运行和调试Spring Boot应用程序.

虽然您可以复制Spring Boot jar，但我们通常建议您使用支持依赖关系管理的构建工具（例如Maven或Gradle）.

### 10.1.1 Maven安装

Spring Boot与Apache Maven 3.3或更高版本兼容.如果您尚未安装Maven，则可以按照[maven.apache.org](https://maven.apache.org)上的说明进行操作.

> 在许多操作系统中，Maven可以与软件包管理器一起安装.如果您使用OSX Homebrew，请尝试 `brew install maven` . Ubuntu用户可以运行 `sudo apt-get install maven` .拥有[Chocolatey](https://chocolatey.org/)的Windows用户可以从提升（管理员）提示符运行 `choco install maven` .

Spring Boot依赖项使用 `org.springframework.boot`   `groupId` .通常，您的Maven POM文件继承自 `spring-boot-starter-parent` 项目，并声明对一个或多个[“Starters”](using-boot-build-systems.html#using-boot-starter)的依赖关系. Spring Boot还提供了一个可选的[Maven plugin](build-tool-plugins-maven-plugin.html)来创建可执行jar.

以下清单显示了典型的 `pom.xml` 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>myproject</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<!-- Inherit defaults from Spring Boot -->
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.1.0.RELEASE</version>
	</parent>

	<!-- Add typical dependencies for a web application -->
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<!-- Package as an executable jar -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>

</project>
```

>   `spring-boot-starter-parent` 是一种使用Spring Boot的好方法，但它可能并不适合所有时间.有时您可能需要从其他父POM继承，或者您可能不喜欢我们的默认设置.在这些情况下，请参阅[Section 13.2.2, “Using Spring Boot without the Parent POM”](using-boot-build-systems.html#using-boot-maven-without-a-parent)以获取使用 `import` 范围的替代解决方案.

### 10.1.2 Gradle安装

Spring Boot与Gradle 4.4及更高版本兼容.如果您尚未安装Gradle，则可以按照[gradle.org](https://gradle.org)上的说明进行操作.

可以使用 `org.springframework.boot`   `group` 声明Spring Boot依赖项.通常，您的项目会声明对一个或多个[“Starters”](using-boot-build-systems.html#using-boot-starter)的依赖关系. Spring Boot提供了一个有用的[Gradle plugin](build-tool-plugins-gradle-plugin.html)，可用于简化依赖声明和创建可执行jar.

----
**Gradle Wrapper** 

当您需要构建项目时，Gradle Wrapper提供了一种“获取”Gradle的好方法.它是一个小脚本和库，您可以与代码一起提交以引导构建过程.有关详细信息，请参阅[docs.gradle.org/4.2.1/userguide/gradle_wrapper.html](https://docs.gradle.org/4.2.1/userguide/gradle_wrapper.html).

----

有关Spring Boot和Gradle入门的更多详细信息，请参阅Gradle插件参考指南的[Getting Started section](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html/#getting-started).

## 10.2安装Spring Boot CLI

Spring Boot CLI（命令行界面）是一个命令行工具，您可以使用它来快速使用Spring进行原型设计.它允许您运行[Groovy](http://groovy-lang.org/)脚本，这意味着您拥有熟悉的类似Java的语法，而没有太多的样板代码.

您不需要使用CLI来使用Spring Boot，但它绝对是实现Spring应用程序的最快方法.

### 10.2.1手动安装

您可以从Spring软件库下载Spring CLI发行版：

- [spring-boot-cli-2.1.0.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.1.0.RELEASE/spring-boot-cli-2.1.0.RELEASE-bin.zip)

- [spring-boot-cli-2.1.0.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.1.0.RELEASE/spring-boot-cli-2.1.0.RELEASE-bin.tar.gz)

也可提供前沿[snapshot distributions](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/).

下载完成后，请按照解压缩的存档中的[INSTALL.txt](https://raw.github.com/spring-projects/spring-boot/v2.1.0.RELEASE/spring-boot-project/spring-boot-cli/src/main/content/INSTALL.txt)说明进行操作.总之， `.zip` 文件的 `bin/` 目录中有一个 `spring` 脚本（适用于Windows的 `spring.bat` ）.或者，您可以将 `java -jar` 与 `.jar` 文件一起使用（该脚本可帮助您确保正确设置类路径）.

### 10.2.2使用SDKMAN安装！

SDKMAN！ （软件开发工具包管理器）可用于管理各种二进制SDK的多个版本，包括Groovy和Spring Boot CLI.获取SDKMAN！从[sdkman.io](http://sdkman.io)并使用以下命令安装Spring Boot：

```java
$ sdk install springboot
$ spring --version
Spring Boot v2.1.0.RELEASE
```

如果您为CLI开发功能并希望轻松访问您构建的版本，请使用以下命令：

```java
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-2.1.0.RELEASE-bin/spring-2.1.0.RELEASE/
$ sdk default springboot dev
$ spring --version
Spring CLI v2.1.0.RELEASE
```

前面的说明安装了一个名为 `dev` 实例的 `spring` 的本地实例.它指向您的目标构建位置，因此每次重建Spring Boot时， `spring` 都是最新的.

您可以通过运行以下命令来查看它：

```java
$ sdk ls springboot

================================================================================
Available Springboot Versions
================================================================================
> + dev
* 2.1.0.RELEASE

================================================================================
+ - local version
* - installed
> - currently in use
================================================================================
```

### 10.2.3 OSX Homebrew安装

如果您使用的是Mac并使用[Homebrew](https://brew.sh/)，则可以使用以下命令安装Spring Boot CLI：

```java
$ brew tap pivotal/tap
$ brew install springboot
```

Homebrew将 `spring` 安装到 `/usr/local/bin` .

> 如果您没有看到公式，那么您的brew安装可能已过时.在这种情况下，运行 `brew update` 并再试一次.

### 10.2.4 MacPorts安装

如果您使用的是Mac并使用[MacPorts](https://www.macports.org/)，则可以使用以下命令安装Spring Boot CLI：

```java
$ sudo port install spring-boot-cli
```

### 10.2.5命令行完成

Spring Boot CLI包含为[BASH](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29)和[zsh](https://en.wikipedia.org/wiki/Z_shell) shell提供命令完成的脚本.您可以在任何shell中 `source` 脚本（也称为 `spring` ）或将其放入您的个人或系统范围的bash完成初始化中.在Debian系统上，系统范围的脚本位于 `/shell-completion/bash` ，并且当新shell启动时，该目录中的所有脚本都会执行.例如，要使用SDKMAN！安装，请手动运行脚本，请使用以下命令：

```java
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
grab  help  jar  run  test  version
```

> 如果使用Homebrew或MacPorts安装Spring Boot CLI，命令行完成脚本将自动注册到shell.

### 10.2.6 Windows Scoop安装

如果你在Windows上使用[Scoop](http://scoop.sh/)，可以使用以下命令安装Spring Boot CLI：

```java
> scoop bucket add extras
> scoop install springboot
```

Scoop将 `spring` 安装到 `~/scoop/apps/springboot/current/bin` .

> 如果您没有看到应用程序清单，则您的安装scoop可能已过时.在这种情况下，运行 `scoop update` 并再试一次.

### 10.2.7快速启动Spring CLI示例

您可以使用以下Web应用程序来测试您的安装.首先，创建一个名为 `app.groovy` 的文件，如下所示：

```java
@RestController
class ThisWillActuallyRun {

	@RequestMapping("/")
	String home() {
		"Hello World!"
	}

}
```

然后从shell运行它，如下所示：

```java
$ spring run app.groovy
```

> 应用程序的第一次运行速度很慢，因为下载了依赖项.后续运行要快得多.

在您喜欢的网络浏览器中打开 `localhost:8080` .您应该看到以下输出：

```java
Hello World!
```

## 10.3从早期版本的Spring Boot升级

如果要从早期版本的Spring Boot升级，请检查提供详细升级说明的[“migration guide” on the project wiki](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide).另请查看[“release notes”](https://github.com/spring-projects/spring-boot/wiki)以获取每个版本的“新的和值得注意的”功能列表.

升级到新功能版本时，某些属性可能已重命名或删除. Spring Boot提供了一种在启动时分析应用程序环境和打印诊断的方法，还可以在运行时临时迁移属性.要启用该功能，请将以下依赖项添加到项目中：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-properties-migrator</artifactId>
	<scope>runtime</scope>
</dependency>
```

_0015不会考虑在环境中添加较晚的_例子，例如使用 `@PropertySource` 时.

> 一旦完成迁移，请确保从项目的依赖项中删除此模块.

要升级现有CLI安装，请使用相应的软件包管理器命令（例如， `brew upgrade` ），或者，如果手动安装CLI，请按照[standard instructions](getting-started-installing-spring-boot.html#getting-started-manual-cli-installation)进行操作，记住更新 `PATH` 环境变量以删除任何旧引用.


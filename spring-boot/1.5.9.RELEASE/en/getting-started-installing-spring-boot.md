## 10. Installing Spring Boot

Spring Boot can be used with “classic” Java development tools or installed as a command line tool. Either way, you need [Java SDK v1.8](https://www.java.com) or higher. Before you begin, you should check your current Java installation by using the following command:

```java
$ java -version
```

If you are new to Java development or if you want to experiment with Spring Boot, you might want to try the [Spring Boot CLI](getting-started-installing-spring-boot.html#getting-started-installing-the-cli) (Command Line Interface) first. Otherwise, read on for “classic” installation instructions.

## 10.1 Installation Instructions for the Java Developer

You can use Spring Boot in the same way as any standard Java library. To do so, include the appropriate  `spring-boot-*.jar`  files on your classpath. Spring Boot does not require any special tools integration, so you can use any IDE or text editor. Also, there is nothing special about a Spring Boot application, so you can run and debug a Spring Boot application as you would any other Java program.

Although you could copy Spring Boot jars, we generally recommend that you use a build tool that supports dependency management (such as Maven or Gradle).

### 10.1.1 Maven Installation

Spring Boot is compatible with Apache Maven 3.3 or above. If you do not already have Maven installed, you can follow the instructions at [maven.apache.org](https://maven.apache.org).

> On many operating systems, Maven can be installed with a package manager. If you use OSX Homebrew, try  `brew install maven` . Ubuntu users can run  `sudo apt-get install maven` . Windows users with [Chocolatey](https://chocolatey.org/) can run  `choco install maven`  from an elevated (administrator) prompt.

Spring Boot dependencies use the  `org.springframework.boot`   `groupId` . Typically, your Maven POM file inherits from the  `spring-boot-starter-parent`  project and declares dependencies to one or more [“Starters”](using-boot-build-systems.html#using-boot-starter). Spring Boot also provides an optional [Maven plugin](build-tool-plugins-maven-plugin.html) to create executable jars.

The following listing shows a typical  `pom.xml`  file:

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

> The  `spring-boot-starter-parent`  is a great way to use Spring Boot, but it might not be suitable all of the time. Sometimes you may need to inherit from a different parent POM, or you might not like our default settings. In those cases, see [Section 13.2.2, “Using Spring Boot without the Parent POM”](using-boot-build-systems.html#using-boot-maven-without-a-parent) for an alternative solution that uses an  `import`  scope.

### 10.1.2 Gradle Installation

Spring Boot is compatible with Gradle 4.4 and later. If you do not already have Gradle installed, you can follow the instructions at [gradle.org](https://gradle.org).

Spring Boot dependencies can be declared by using the  `org.springframework.boot`   `group` . Typically, your project declares dependencies to one or more [“Starters”](using-boot-build-systems.html#using-boot-starter). Spring Boot provides a useful [Gradle plugin](build-tool-plugins-gradle-plugin.html) that can be used to simplify dependency declarations and to create executable jars.

----
**Gradle Wrapper** 

The Gradle Wrapper provides a nice way of “obtaining” Gradle when you need to build a project. It is a small script and library that you commit alongside your code to bootstrap the build process. See [docs.gradle.org/4.2.1/userguide/gradle_wrapper.html](https://docs.gradle.org/4.2.1/userguide/gradle_wrapper.html) for details.

----

More details on getting started with Spring Boot and Gradle can be found in the [Getting Started section](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html/#getting-started) of the Gradle plugin’s reference guide.

## 10.2 Installing the Spring Boot CLI

The Spring Boot CLI (Command Line Interface) is a command line tool that you can use to quickly prototype with Spring. It lets you run [Groovy](http://groovy-lang.org/) scripts, which means that you have a familiar Java-like syntax without so much boilerplate code.

You do not need to use the CLI to work with Spring Boot, but it is definitely the quickest way to get a Spring application off the ground.

### 10.2.1 Manual Installation

You can download the Spring CLI distribution from the Spring software repository:

- [spring-boot-cli-2.1.0.RELEASE-bin.zip](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.1.0.RELEASE/spring-boot-cli-2.1.0.RELEASE-bin.zip)

- [spring-boot-cli-2.1.0.RELEASE-bin.tar.gz](https://repo.spring.io/release/org/springframework/boot/spring-boot-cli/2.1.0.RELEASE/spring-boot-cli-2.1.0.RELEASE-bin.tar.gz)

Cutting edge [snapshot distributions](https://repo.spring.io/snapshot/org/springframework/boot/spring-boot-cli/) are also available.

Once downloaded, follow the [INSTALL.txt](https://raw.github.com/spring-projects/spring-boot/v2.1.0.RELEASE/spring-boot-project/spring-boot-cli/src/main/content/INSTALL.txt) instructions from the unpacked archive. In summary, there is a  `spring`  script ( `spring.bat`  for Windows) in a  `bin/`  directory in the  `.zip`  file. Alternatively, you can use  `java -jar`  with the  `.jar`  file (the script helps you to be sure that the classpath is set correctly).

### 10.2.2 Installation with SDKMAN!

SDKMAN! (The Software Development Kit Manager) can be used for managing multiple versions of various binary SDKs, including Groovy and the Spring Boot CLI. Get SDKMAN! from [sdkman.io](http://sdkman.io) and install Spring Boot by using the following commands:

```java
$ sdk install springboot
$ spring --version
Spring Boot v2.1.0.RELEASE
```

If you develop features for the CLI and want easy access to the version you built, use the following commands:

```java
$ sdk install springboot dev /path/to/spring-boot/spring-boot-cli/target/spring-boot-cli-2.1.0.RELEASE-bin/spring-2.1.0.RELEASE/
$ sdk default springboot dev
$ spring --version
Spring CLI v2.1.0.RELEASE
```

The preceding instructions install a local instance of  `spring`  called the  `dev`  instance. It points at your target build location, so every time you rebuild Spring Boot,  `spring`  is up-to-date.

You can see it by running the following command:

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

### 10.2.3 OSX Homebrew Installation

If you are on a Mac and use [Homebrew](https://brew.sh/), you can install the Spring Boot CLI by using the following commands:

```java
$ brew tap pivotal/tap
$ brew install springboot
```

Homebrew installs  `spring`  to  `/usr/local/bin` .

> If you do not see the formula, your installation of brew might be out-of-date. In that case, run  `brew update`  and try again.

### 10.2.4 MacPorts Installation

If you are on a Mac and use [MacPorts](https://www.macports.org/), you can install the Spring Boot CLI by using the following command:

```java
$ sudo port install spring-boot-cli
```

### 10.2.5 Command-line Completion

The Spring Boot CLI includes scripts that provide command completion for the [BASH](https://en.wikipedia.org/wiki/Bash_%28Unix_shell%29) and [zsh](https://en.wikipedia.org/wiki/Z_shell) shells. You can  `source`  the script (also named  `spring` ) in any shell or put it in your personal or system-wide bash completion initialization. On a Debian system, the system-wide scripts are in  `/shell-completion/bash`  and all scripts in that directory are executed when a new shell starts. For example, to run the script manually if you have installed by using SDKMAN!, use the following commands:

```java
$ . ~/.sdkman/candidates/springboot/current/shell-completion/bash/spring
$ spring <HIT TAB HERE>
grab  help  jar  run  test  version
```

> If you install the Spring Boot CLI by using Homebrew or MacPorts, the command-line completion scripts are automatically registered with your shell.

### 10.2.6 Windows Scoop Installation

If you are on a Windows and use [Scoop](http://scoop.sh/), you can install the Spring Boot CLI by using the following commands:

```java
> scoop bucket add extras
> scoop install springboot
```

Scoop installs  `spring`  to  `~/scoop/apps/springboot/current/bin` .

> If you do not see the app manifest, your installation of scoop might be out-of-date. In that case, run  `scoop update`  and try again.

### 10.2.7 Quick-start Spring CLI Example

You can use the following web application to test your installation. To start, create a file called  `app.groovy` , as follows:

```java
@RestController
class ThisWillActuallyRun {

	@RequestMapping("/")
	String home() {
		"Hello World!"
	}

}
```

Then run it from a shell, as follows:

```java
$ spring run app.groovy
```

> The first run of your application is slow, as dependencies are downloaded. Subsequent runs are much quicker.

Open  `localhost:8080`  in your favorite web browser. You should see the following output:

```java
Hello World!
```

## 10.3 Upgrading from an Earlier Version of Spring Boot

If you are upgrading from an earlier release of Spring Boot, check the [“migration guide” on the project wiki](https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-2.0-Migration-Guide) that provides detailed upgrade instructions. Check also the [“release notes”](https://github.com/spring-projects/spring-boot/wiki) for a list of “new and noteworthy” features for each release.

When upgrading to a new feature release, some properties may have been renamed or removed. Spring Boot provides a way to analyze your application’s environment and print diagnostics at startup, but also temporarily migrate properties at runtime for you. To enable that feature, add the following dependency to your project:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-properties-migrator</artifactId>
	<scope>runtime</scope>
</dependency>
```

> Properties that are added late to the environment, such as when using  `@PropertySource` , will not be taken into account.

> Once you’re done with the migration, please make sure to remove this module from your project’s dependencies.

To upgrade an existing CLI installation, use the appropriate package manager command (for example,  `brew upgrade` ) or, if you manually installed the CLI, follow the [standard instructions](getting-started-installing-spring-boot.html#getting-started-manual-cli-installation), remembering to update your  `PATH`  environment variable to remove any older references.


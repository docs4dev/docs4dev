## 19. Running Your Application

One of the biggest advantages of packaging your application as a jar and using an embedded HTTP server is that you can run your application as you would any other. Debugging Spring Boot applications is also easy. You do not need any special IDE plugins or extensions.

> This section only covers jar based packaging. If you choose to package your application as a war file, you should refer to your server and IDE documentation.

## 19.1 Running from an IDE

You can run a Spring Boot application from your IDE as a simple Java application. However, you first need to import your project. Import steps vary depending on your IDE and build system. Most IDEs can import Maven projects directly. For example, Eclipse users can select  `Import…`  →  `Existing Maven Projects`  from the  `File`  menu.

If you cannot directly import your project into your IDE, you may be able to generate IDE metadata by using a build plugin. Maven includes plugins for [Eclipse](https://maven.apache.org/plugins/maven-eclipse-plugin/) and [IDEA](https://maven.apache.org/plugins/maven-idea-plugin/). Gradle offers plugins for [various IDEs](https://docs.gradle.org/4.2.1/userguide/userguide.html).

> If you accidentally run a web application twice, you see a “Port already in use” error. STS users can use the  `Relaunch`  button rather than the  `Run`  button to ensure that any existing instance is closed.

## 19.2 Running as a Packaged Application

If you use the Spring Boot Maven or Gradle plugins to create an executable jar, you can run your application using  `java -jar` , as shown in the following example:

```java
$ java -jar target/myapplication-0.0.1-SNAPSHOT.jar
```

It is also possible to run a packaged application with remote debugging support enabled. Doing so lets you attach a debugger to your packaged application, as shown in the following example:

```java
$ java -Xdebug -Xrunjdwp:server=y,transport=dt_socket,address=8000,suspend=n \
-jar target/myapplication-0.0.1-SNAPSHOT.jar
```

## 19.3 Using the Maven Plugin

The Spring Boot Maven plugin includes a  `run`  goal that can be used to quickly compile and run your application. Applications run in an exploded form, as they do in your IDE. The following example shows a typical Maven command to run a Spring Boot application:

```java
$ mvn spring-boot:run
```

You might also want to use the  `MAVEN_OPTS`  operating system environment variable, as shown in the following example:

```java
$ export MAVEN_OPTS=-Xmx1024m
```

## 19.4 Using the Gradle Plugin

The Spring Boot Gradle plugin also includes a  `bootRun`  task that can be used to run your application in an exploded form. The  `bootRun`  task is added whenever you apply the  `org.springframework.boot`  and  `java`  plugins and is shown in the following example:

```java
$ gradle bootRun
```

You might also want to use the  `JAVA_OPTS`  operating system environment variable, as shown in the following example:

```java
$ export JAVA_OPTS=-Xmx1024m
```

## 19.5 Hot Swapping

Since Spring Boot applications are just plain Java applications, JVM hot-swapping should work out of the box. JVM hot swapping is somewhat limited with the bytecode that it can replace. For a more complete solution, [JRebel](https://zeroturnaround.com/software/jrebel/) can be used.

The  `spring-boot-devtools`  module also includes support for quick application restarts. See the [Chapter 20, Developer Tools](using-boot-devtools.html) section later in this chapter and the [Hot swapping “How-to”](howto-hotswapping.html) for details.


## 11. Developing Your First Spring Boot Application

This section describes how to develop a simple “Hello World!” web application that highlights some of Spring Boot’s key features. We use Maven to build this project, since most IDEs support it.

> The [spring.io](https://spring.io) web site contains many “Getting Started” [guides](https://spring.io/guides) that use Spring Boot. If you need to solve a specific problem, check there first.

Before we begin, open a terminal and run the following commands to ensure that you have valid versions of Java and Maven installed:

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

> This sample needs to be created in its own folder. Subsequent instructions assume that you have created a suitable folder and that it is your current directory.

## 11.1 Creating the POM

We need to start by creating a Maven  `pom.xml`  file. The  `pom.xml`  is the recipe that is used to build your project. Open your favorite text editor and add the following:

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

The preceding listing should give you a working build. You can test it by running  `mvn package`  (for now, you can ignore the “jar will be empty - no content was marked for inclusion!” warning).

> At this point, you could import the project into an IDE (most modern Java IDEs include built-in support for Maven). For simplicity, we continue to use a plain text editor for this example.

## 11.2 Adding Classpath Dependencies

Spring Boot provides a number of “Starters” that let you add jars to your classpath. Our sample application has already used  `spring-boot-starter-parent`  in the  `parent`  section of the POM. The  `spring-boot-starter-parent`  is a special starter that provides useful Maven defaults. It also provides a [dependency-management](using-boot-build-systems.html#using-boot-dependency-management) section so that you can omit  `version`  tags for “blessed” dependencies.

Other “Starters” provide dependencies that you are likely to need when developing a specific type of application. Since we are developing a web application, we add a  `spring-boot-starter-web`  dependency. Before that, we can look at what we currently have by running the following command:

```java
$ mvn dependency:tree

[INFO] com.example:myproject:jar:0.0.1-SNAPSHOT
```

The  `mvn dependency:tree`  command prints a tree representation of your project dependencies. You can see that  `spring-boot-starter-parent`  provides no dependencies by itself. To add the necessary dependencies, edit your  `pom.xml`  and add the  `spring-boot-starter-web`  dependency immediately below the  `parent`  section:

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-web</artifactId>
	</dependency>
</dependencies>
```

If you run  `mvn dependency:tree`  again, you see that there are now a number of additional dependencies, including the Tomcat web server and Spring Boot itself.

## 11.3 Writing the Code

To finish our application, we need to create a single Java file. By default, Maven compiles sources from  `src/main/java` , so you need to create that folder structure and then add a file named  `src/main/java/Example.java`  to contain the following code:

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

Although there is not much code here, quite a lot is going on. We step through the important parts in the next few sections.

### 11.3.1 The @RestController and @RequestMapping Annotations

The first annotation on our  `Example`  class is  `@RestController` . This is known as a stereotype annotation. It provides hints for people reading the code and for Spring that the class plays a specific role. In this case, our class is a web  `@Controller` , so Spring considers it when handling incoming web requests.

The  `@RequestMapping`  annotation provides “routing” information. It tells Spring that any HTTP request with the  `/`  path should be mapped to the  `home`  method. The  `@RestController`  annotation tells Spring to render the resulting string directly back to the caller.

> The  `@RestController`  and  `@RequestMapping`  annotations are Spring MVC annotations. (They are not specific to Spring Boot.) See the [MVC section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/web.html#mvc) in the Spring Reference Documentation for more details.

### 11.3.2 The @EnableAutoConfiguration Annotation

The second class-level annotation is  `@EnableAutoConfiguration` . This annotation tells Spring Boot to “guess” how you want to configure Spring, based on the jar dependencies that you have added. Since  `spring-boot-starter-web`  added Tomcat and Spring MVC, the auto-configuration assumes that you are developing a web application and sets up Spring accordingly.

----
**Starters and Auto-configuration** 

Auto-configuration is designed to work well with “Starters”, but the two concepts are not directly tied. You are free to pick and choose jar dependencies outside of the starters. Spring Boot still does its best to auto-configure your application.

----

### 11.3.3 The “main” Method

The final part of our application is the  `main`  method. This is just a standard method that follows the Java convention for an application entry point. Our main method delegates to Spring Boot’s  `SpringApplication`  class by calling  `run` .  `SpringApplication`  bootstraps our application, starting Spring, which, in turn, starts the auto-configured Tomcat web server. We need to pass  `Example.class`  as an argument to the  `run`  method to tell  `SpringApplication`  which is the primary Spring component. The  `args`  array is also passed through to expose any command-line arguments.

## 11.4 Running the Example

At this point, your application should work. Since you used the  `spring-boot-starter-parent`  POM, you have a useful  `run`  goal that you can use to start the application. Type  `mvn spring-boot:run`  from the root project directory to start the application. You should see output similar to the following:

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

If you open a web browser to  `localhost:8080` , you should see the following output:

```java
Hello World!
```

To gracefully exit the application, press  `ctrl-c` .

## 11.5 Creating an Executable Jar

We finish our example by creating a completely self-contained executable jar file that we could run in production. Executable jars (sometimes called “fat jars”) are archives containing your compiled classes along with all of the jar dependencies that your code needs to run.

----
**Executable jars and Java** 

Java does not provide a standard way to load nested jar files (jar files that are themselves contained within a jar). This can be problematic if you are looking to distribute a self-contained application.

To solve this problem, many developers use “uber” jars. An uber jar packages all the classes from all the application’s dependencies into a single archive. The problem with this approach is that it becomes hard to see which libraries are in your application. It can also be problematic if the same filename is used (but with different content) in multiple jars.

Spring Boot takes a [different approach](executable-jar.html) and lets you actually nest jars directly.

----

To create an executable jar, we need to add the  `spring-boot-maven-plugin`  to our  `pom.xml` . To do so, insert the following lines just below the  `dependencies`  section:

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

> The  `spring-boot-starter-parent`  POM includes  `<executions>`  configuration to bind the  `repackage`  goal. If you do not use the parent POM, you need to declare this configuration yourself. See the [plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/usage.html) for details.

Save your  `pom.xml`  and run  `mvn package`  from the command line, as follows:

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

If you look in the  `target`  directory, you should see  `myproject-0.0.1-SNAPSHOT.jar` . The file should be around 10 MB in size. If you want to peek inside, you can use  `jar tvf` , as follows:

```java
$ jar tvf target/myproject-0.0.1-SNAPSHOT.jar
```

You should also see a much smaller file named  `myproject-0.0.1-SNAPSHOT.jar.original`  in the  `target`  directory. This is the original jar file that Maven created before it was repackaged by Spring Boot.

To run that application, use the  `java -jar`  command, as follows:

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

As before, to exit the application, press  `ctrl-c` .


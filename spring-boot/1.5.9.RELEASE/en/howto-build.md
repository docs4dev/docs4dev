## 91. Build

Spring Boot includes build plugins for Maven and Gradle. This section answers common questions about these plugins.

## 91.1 Generate Build Information

Both the Maven plugin and the Gradle plugin allow generating build information containing the coordinates, name, and version of the project. The plugins can also be configured to add additional properties through configuration. When such a file is present, Spring Boot auto-configures a  `BuildProperties`  bean.

To generate build information with Maven, add an execution for the  `build-info`  goal, as shown in the following example:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<version>2.1.0.RELEASE</version>
			<executions>
				<execution>
					<goals>
						<goal>build-info</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

> See the [Spring Boot Maven Plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin) for more details.

The following example does the same with Gradle:

```java
springBoot {
	buildInfo()
}
```

> See the [Spring Boot Gradle Plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html/#integrating-with-actuator-build-info) for more details.

## 91.2 Generate Git Information

Both Maven and Gradle allow generating a  `git.properties`  file containing information about the state of your  `git`  source code repository when the project was built.

For Maven users, the  `spring-boot-starter-parent`  POM includes a pre-configured plugin to generate a  `git.properties`  file. To use it, add the following declaration to your POM:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>pl.project13.maven</groupId>
			<artifactId>git-commit-id-plugin</artifactId>
		</plugin>
	</plugins>
</build>
```

Gradle users can achieve the same result by using the [gradle-git-properties](https://plugins.gradle.org/plugin/com.gorylenko.gradle-git-properties) plugin, as shown in the following example:

```java
plugins {
	id "com.gorylenko.gradle-git-properties" version "1.5.1"
}
```

> The commit time in  `git.properties`  is expected to match the following format:  `yyyy-MM-dd’T’HH:mm:ssZ` . This is the default format for both plugins listed above. Using this format lets the time be parsed into a  `Date`  and its format, when serialized to JSON, to be controlled by Jackson’s date serialization configuration settings.

## 91.3 Customize Dependency Versions

If you use a Maven build that inherits directly or indirectly from  `spring-boot-dependencies`  (for instance,  `spring-boot-starter-parent` ) but you want to override a specific third-party dependency, you can add appropriate  `<properties>`  elements. Browse the [spring-boot-dependencies](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml) POM for a complete list of properties. For example, to pick a different  `slf4j`  version, you would add the following property:

```xml
<properties>
	<slf4j.version>1.7.5<slf4j.version>
</properties>
```

> Doing so only works if your Maven project inherits (directly or indirectly) from  `spring-boot-dependencies` . If you have added  `spring-boot-dependencies`  in your own  `dependencyManagement`  section with  `<scope>import</scope>` , you have to redefine the artifact yourself instead of overriding the property.

> Each Spring Boot release is designed and tested against this specific set of third-party dependencies. Overriding versions may cause compatibility issues.

To override dependency versions in Gradle, see [this section](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html/#managing-dependencies-customizing) of the Gradle plugin’s documentation.

## 91.4 Create an Executable JAR with Maven

The  `spring-boot-maven-plugin`  can be used to create an executable “fat” JAR. If you use the  `spring-boot-starter-parent`  POM, you can declare the plugin and your jars are repackaged as follows:

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

If you do not use the parent POM, you can still use the plugin. However, you must additionally add an  `<executions>`  section, as follows:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<version>2.1.0.RELEASE</version>
			<executions>
				<execution>
					<goals>
						<goal>repackage</goal>
					</goals>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

See the [plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/usage.html) for full usage details.

## 91.5 Use a Spring Boot Application as a Dependency

Like a war file, a Spring Boot application is not intended to be used as a dependency. If your application contains classes that you want to share with other projects, the recommended approach is to move that code into a separate module. The separate module can then be depended upon by your application and other projects.

If you cannot rearrange your code as recommended above, Spring Boot’s Maven and Gradle plugins must be configured to produce a separate artifact that is suitable for use as a dependency. The executable archive cannot be used as a dependency as the [executable jar format](executable-jar.html#executable-jar-jar-file-structure) packages application classes in  `BOOT-INF/classes` . This means that they cannot be found when the executable jar is used as a dependency.

To produce the two artifacts, one that can be used as a dependency and one that is executable, a classifier must be specified. This classifier is applied to the name of the executable archive, leaving the default archive for use as a dependency.

To configure a classifier of  `exec`  in Maven, you can use the following configuration:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<classifier>exec</classifier>
			</configuration>
		</plugin>
	</plugins>
</build>
```

## 91.6 Extract Specific Libraries When an Executable Jar Runs

Most nested libraries in an executable jar do not need to be unpacked in order to run. However, certain libraries can have problems. For example, JRuby includes its own nested jar support, which assumes that the  `jruby-complete.jar`  is always directly available as a file in its own right.

To deal with any problematic libraries, you can flag that specific nested jars should be automatically unpacked to the “temp folder” when the executable jar first runs.

For example, to indicate that JRuby should be flagged for unpacking by using the Maven Plugin, you would add the following configuration:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<requiresUnpack>
					<dependency>
						<groupId>org.jruby</groupId>
						<artifactId>jruby-complete</artifactId>
					</dependency>
				</requiresUnpack>
			</configuration>
		</plugin>
	</plugins>
</build>
```

## 91.7 Create a Non-executable JAR with Exclusions

Often, if you have an executable and a non-executable jar as two separate build products, the executable version has additional configuration files that are not needed in a library jar. For example, the  `application.yml`  configuration file might by excluded from the non-executable JAR.

In Maven, the executable jar must be the main artifact and you can add a classified jar for the library, as follows:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
		</plugin>
		<plugin>
			<artifactId>maven-jar-plugin</artifactId>
			<executions>
				<execution>
					<id>lib</id>
					<phase>package</phase>
					<goals>
						<goal>jar</goal>
					</goals>
					<configuration>
						<classifier>lib</classifier>
						<excludes>
							<exclude>application.yml</exclude>
						</excludes>
					</configuration>
				</execution>
			</executions>
		</plugin>
	</plugins>
</build>
```

## 91.8 Remote Debug a Spring Boot Application Started with Maven

To attach a remote debugger to a Spring Boot application that was started with Maven, you can use the  `jvmArguments`  property of the [maven plugin](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin).

See [this example](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/examples/run-debug.html) for more details.

## 91.9 Build an Executable Archive from Ant without Using spring-boot-antlib

To build with Ant, you need to grab dependencies, compile, and then create a jar or war archive. To make it executable, you can either use the  `spring-boot-antlib`  module or you can follow these instructions:

If you are building a jar, package the application’s classes and resources in a nested BOOT-INF/classes directory. If you are building a war, package the application’s classes in a nested WEB-INF/classes directory as usual. Add the runtime dependencies in a nested BOOT-INF/lib directory for a jar or WEB-INF/lib for a war. Remember not to compress the entries in the archive. Add the provided (embedded container) dependencies in a nested BOOT-INF/lib directory for a jar or WEB-INF/lib-provided for a war. Remember not to compress the entries in the archive. Add the spring-boot-loader classes at the root of the archive (so that the Main-Class is available). Use the appropriate launcher (such as JarLauncher for a jar file) as a Main-Class attribute in the manifest and specify the other properties it needs as manifest entries — principally, by setting a Start-Class property.

The following example shows how to build an executable archive with Ant:

```xml
<target name="build" depends="compile">
	<jar destfile="target/${ant.project.name}-${spring-boot.version}.jar" compress="false">
		<mappedresources>
			<fileset dir="target/classes" />
			<globmapper from="*" to="BOOT-INF/classes/*"/>
		</mappedresources>
		<mappedresources>
			<fileset dir="src/main/resources" erroronmissingdir="false"/>
			<globmapper from="*" to="BOOT-INF/classes/*"/>
		</mappedresources>
		<mappedresources>
			<fileset dir="${lib.dir}/runtime" />
			<globmapper from="*" to="BOOT-INF/lib/*"/>
		</mappedresources>
		<zipfileset src="${lib.dir}/loader/spring-boot-loader-jar-${spring-boot.version}.jar" />
		<manifest>
			<attribute name="Main-Class" value="org.springframework.boot.loader.JarLauncher" />
			<attribute name="Start-Class" value="${start-class}" />
		</manifest>
	</jar>
</target>
```

The [Ant Sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-ant) has a  `build.xml`  file with a  `manual`  task that should work if you run it with the following command:

```xml
$ ant -lib <folder containing ivy-2.2.jar> clean manual
```

Then you can run the application with the following command:

```java
$ java -jar target/*.jar
```


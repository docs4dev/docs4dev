## 91. Build

Spring Boot包含Maven和Gradle的构建插件.本节回答有关这些插件的常见问题.

## 91.1生成构建信息

Maven插件和Gradle插件都允许生成包含项目的坐标，名称和版本的构建信息.插件还可以配置为通过配置添加其他属性.当存在这样的文件时，Spring Boot会自动配置 `BuildProperties`  bean.

要使用Maven生成构建信息，请为 `build-info` 目标添加执行，如以下示例所示：

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

> 有关详情，请参阅[Spring Boot Maven Plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin).

以下示例对Gradle执行相同操作：

```java
springBoot {
	buildInfo()
}
```

> 有关详情，请参阅[Spring Boot Gradle Plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html/#integrating-with-actuator-build-info).

## 91.2生成Git信息

Maven和Gradle都允许生成 `git.properties` 文件，其中包含有关项目构建时 `git` 源代码存储库状态的信息.

对于Maven用户， `spring-boot-starter-parent`  POM包含一个预先配置的插件，用于生成 `git.properties` 文件.要使用它，请将以下声明添加到POM：

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

Gradle用户可以使用[gradle-git-properties](https://plugins.gradle.org/plugin/com.gorylenko.gradle-git-properties)插件获得相同的结果，如以下示例所示：

```java
plugins {
	id "com.gorylenko.gradle-git-properties" version "1.5.1"
}
```

>   `git.properties` 中的提交时间应符合以下格式： `yyyy-MM-dd’T’HH:mm:ssZ` .这是上面列出的两个插件的默认格式.使用此格式可以将时间解析为 `Date` 及其格式，在序列化为JSON时，由Jackson的日期序列化配置设置控制.

## 91.3自定义依赖版本

如果您使用直接或间接从 `spring-boot-dependencies` 继承的Maven构建（例如， `spring-boot-starter-parent` ）但您想要覆盖特定的第三方依赖项，则可以添加适当的 `<properties>` 元素.浏览[spring-boot-dependencies](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml) POM以获取完整的属性列表.例如，要选择不同的 `slf4j` 版本，您需要添加以下属性：

```xml
<properties>
	<slf4j.version>1.7.5<slf4j.version>
</properties>
```

> 这样只有在你的Maven项目（直接或间接）从 `spring-boot-dependencies` 继承时才有效.如果您使用 `<scope>import</scope>` 在自己的 `dependencyManagement` 部分中添加了 `spring-boot-dependencies` ，则必须自己重新定义工件而不是覆盖该属性.

> 每个Spring Boot版本都是针对这组特定的第三方依赖项进行设计和测试的.覆盖版本可能会导致兼容性问题.

要在Gradle中覆盖依赖项版本，请参阅Gradle插件文档的[this section](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html/#managing-dependencies-customizing).

## 91.4使用Maven创建可执行JAR

`spring-boot-maven-plugin` 可用于创建可执行的“胖”JAR.如果你使用 `spring-boot-starter-parent`  POM，你可以声明插件，你的jar重新包装如下：

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

如果您不使用父POM，您仍然可以使用该插件.但是，您必须另外添加 `<executions>` 部分，如下所示：

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

有关完整用法的详细信息，请参阅[plugin documentation](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/usage.html).

## 91.5使用Spring Boot应用程序作为依赖项

与war文件一样，Spring Boot应用程序不能用作依赖项.如果您的应用程序包含要与其他项目共享的类，则建议的方法是将该代码移动到单独的模块中.然后，您的应用程序和其他项目可以依赖单独的模块.

如果您不能按照上面的建议重新安排代码，则必须配置Spring Boot的Maven和Gradle插件，以生成适合用作依赖项的单独工件.可执行存档不能用作 `BOOT-INF/classes` 中[executable jar format](executable-jar.html#executable-jar-jar-file-structure)包应用程序类的依赖项.这意味着当可执行jar用作依赖项时，无法找到它们.

要生成两个可以用作依赖项的工件和一个可执行的工件，必须指定一个分类器.此分类器应用于可执行归档的名称，保留默认归档以供使用作为依赖.

要在Maven中配置 `exec` 的分类器，可以使用以下配置：

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

## 91.6当可执行文件运行时提取特定库

可执行jar中的大多数嵌套库不需要解压缩才能运行.但是，某些库可能存在问题.例如，JRuby包含自己的嵌套jar支持，它假定 `jruby-complete.jar` 总是直接作为文件直接使用.

要处理任何有问题的库，您可以标记当可执行jar首次运行时，应该将特定的嵌套jar自动解压缩到“temp文件夹”.

例如，要指示应使用Maven插件标记JRuby以进行解包，您将添加以下配置：

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

## 91.7使用排除项创建不可执行的JAR

通常，如果您将可执行文件和非可执行jar作为两个单独的构建产品，则可执行版本具有库jar中不需要的其他配置文件.例如， `application.yml` 配置文件可能会从非可执行JAR中排除.

在Maven中，可执行jar必须是主要工件，您可以为库添加一个分类jar，如下所示：

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

## 91.8从Maven开始远程调试Spring Boot应用程序

要将远程调试器附加到使用Maven启动的Spring Boot应用程序，可以使用[maven plugin](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin)的 `jvmArguments` 属性.

有关详细信息，请参阅[this example](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/examples/run-debug.html).

## 91.9从Ant构建可执行文件，而不使用spring-boot-antlib

要使用Ant构建，您需要获取依赖项，编译，然后创建jar或war存档.要使其可执行，您可以使用 `spring-boot-antlib` 模块，也可以按照以下说明操作：

如果要构建jar，请将应用程序的类和资源打包到嵌套的BOOT-INF / classes目录中.如果要构建war，请像往常一样将应用程序的类打包在嵌套的WEB-INF / classes目录中.在嵌套的BOOT-INF / lib目录中为jar或WEB-INF / lib添加运行时依赖项以进行war.切记不要压缩存档中的条目.将提供的（嵌入式容器）依赖项添加到嵌套的BOOT-INF / lib目录中，以便为战争提供jar或WEB-INF / lib.切记不要压缩存档中的条目.在归档的根目录中添加spring-boot-loader类（以便Main-Class可用）.使用适当的启动程序（例如jar文件的JarLauncher）作为清单中的Main-Class属性，并指定其作为清单条目所需的其他属性 - 主要通过设置Start-Class属性.

以下示例显示如何使用Ant构建可执行存档：

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

[Ant Sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-ant)有一个带有 `manual` 任务的 `build.xml` 文件，如果使用以下命令运行它，该任务应该有效：

```xml
$ ant -lib <folder containing ivy-2.2.jar> clean manual
```

然后，您可以使用以下命令运行该应用程序：

```java
$ java -jar target/*.jar
```


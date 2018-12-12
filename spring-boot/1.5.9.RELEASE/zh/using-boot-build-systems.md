## 13.构建系统

强烈建议您选择支持[dependency management](using-boot-build-systems.html#using-boot-dependency-management)的构建系统，并且可以使用发布到“Maven Central”存储库的工件.我们建议您选择Maven或Gradle.可以让Spring Boot与其他构建系统（例如Ant）一起工作，但它们并没有得到特别好的支持.

## 13.1依赖管理

每个版本的Spring Boot都提供了它支持的依赖项的精选列表.实际上，您不需要为构建配置中的任何这些依赖项提供版本，因为Spring Boot会为您管理这些依赖项.当您升级Spring Boot时，这些依赖项也会以一致的方式升级.

> 如果需要，您仍然可以指定版本并覆盖Spring Boot的建议.

精选列表包含可以与Spring Boot一起使用的所有spring模块以及精确的第三方库列表.该列表以标准[Bills of Materials (spring-boot-dependencies)](using-boot-build-systems.html#using-boot-maven-without-a-parent)的形式提供，可与[Maven](using-boot-build-systems.html#using-boot-maven-parent-pom)和[Gradle](using-boot-build-systems.html#using-boot-gradle)一起使用.

>  Spring Boot的每个版本都与Spring Framework的基本版本相关联.我们 **highly** 建议您不要指定其版本.

## 13.2 Maven

Maven用户可以从 `spring-boot-starter-parent` 项目继承以获得合理的默认值.父项目提供以下功能：

- Java 1.8作为默认编译器级别.

- UTF-8源编码.

- A [Dependency Management section](using-boot-build-systems.html#using-boot-dependency-management)，继承自spring-boot-dependencies pom，管理公共依赖项的版本.此依赖关系管理允许您在自己的pom中使用时省略这些依赖项的<version>标记.

- 使用 `repackage` 执行ID执行[repackage goal](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/repackage-mojo.html).

- Sensible [resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html).

- Sensible插件配置（[exec plugin](http://www.mojohaus.org/exec-maven-plugin/)，[Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin)和[shade](https://maven.apache.org/plugins/maven-shade-plugin/)）.
_0009_841_和 `application.yml` 的_Sensible资源过滤，包括特定于配置文件的文件（例如， `application-dev.properties` 和 `application-dev.yml` ）

请注意，由于 `application.properties` 和 `application.yml` 文件接受Spring样式占位符（ `${…}` ），因此Maven过滤更改为使用 `@[email protected]` 占位符. （您可以通过设置名为 `resource.delimiter` 的Maven属性来覆盖它.）

### 13.2.1继承Starter Parent

要将项目配置为从 `spring-boot-starter-parent` 继承，请按如下所示设置 `parent` ：

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.1.0.RELEASE</version>
</parent>
```

> 您应该只需要在此依赖项上指定Spring Boot版本号.如果导入其他启动器，则可以安全地省略版本号.

通过该设置，您还可以通过覆盖自己项目中的属性来覆盖单个依赖项.例如，要升级到另一个Spring Data版本系列，您需要将以下内容添加到 `pom.xml` ：

```xml
<properties>
	<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

> 检查[spring-boot-dependencies pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml)以获取支持的属性列表.

### 13.2.2在没有父POM的情况下使用Spring Boot

不是每个人都喜欢从 `spring-boot-starter-parent`  POM继承.您可能拥有自己需要使用的公司标准父级，或者您可能更愿意明确声明所有Maven配置.

如果您不想使用 `spring-boot-starter-parent` ，您仍然可以通过使用 `scope=import` 依赖项来保持依赖项管理（但不是插件管理）的好处，如下所示：

```xml
<dependencyManagement>
		<dependencies>
		<dependency>
			<!-- Import dependency management from Spring Boot -->
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.1.0.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

如上所述，前面的示例设置不允许您使用属性覆盖单个依赖项.要获得相同的结果，您需要在项目 **before**   `spring-boot-dependencies` 条目的 `dependencyManagement` 中添加一个条目.例如，要升级到另一个Spring Data版本系列，可以将以下元素添加到 `pom.xml` ：

```xml
<dependencyManagement>
	<dependencies>
		<!-- Override Spring Data release train provided by Spring Boot -->
		<dependency>
			<groupId>org.springframework.data</groupId>
			<artifactId>spring-data-releasetrain</artifactId>
			<version>Fowler-SR2</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-dependencies</artifactId>
			<version>2.1.0.RELEASE</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

> 在前面的示例中，我们指定了BOM，但是可以以相同的方式覆盖任何依赖关系类型.

### 13.2.3使用Spring Boot Maven插件

Spring Boot包含一个[Maven plugin](build-tool-plugins-maven-plugin.html)，它可以将项目打包为可执行jar.如果要使用插件，请将插件添加到 `<plugins>` 部分，如以下示例所示：

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

> 如果您使用Spring Boot启动程序父pom，则只需添加插件.除非您要更改父级中定义的设置，否则无需对其进行配置.

## 13.3 Gradle

要了解如何将Spring Boot与Gradle一起使用，请参阅Spring Boot的Gradle插件的文档：

- Reference（[HTML](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html)和[PDF](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf)）

- [API](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/api)

## 13.4Ant

可以使用Apache Ant Ivy构建Spring Boot项目.  `spring-boot-antlib` “AntLib”模块也可用于帮助Ant创建可执行jar.

要声明依赖项，典型的 `ivy.xml` 文件类似于以下示例：

```xml
<ivy-module version="2.0">
	<info organisation="org.springframework.boot" module="spring-boot-sample-ant" />
	<configurations>
		<conf name="compile" description="everything needed to compile this module" />
		<conf name="runtime" extends="compile" description="everything needed to run this module" />
	</configurations>
	<dependencies>
		<dependency org="org.springframework.boot" name="spring-boot-starter"
			rev="${spring-boot.version}" conf="compile" />
	</dependencies>
</ivy-module>
```

典型的 `build.xml` 类似于以下示例：

```xml
<project
	xmlns:ivy="antlib:org.apache.ivy.ant"
	xmlns:spring-boot="antlib:org.springframework.boot.ant"
	name="myapp" default="build">

	<property name="spring-boot.version" value="2.1.0.RELEASE" />

	<target name="resolve" description="--> retrieve dependencies with ivy">
		<ivy:retrieve pattern="lib/[conf]/[artifact]-[type]-[revision].[ext]" />
	</target>

	<target name="classpaths" depends="resolve">
		<path id="compile.classpath">
			<fileset dir="lib/compile" includes="*.jar" />
		</path>
	</target>

	<target name="init" depends="classpaths">
		<mkdir dir="build/classes" />
	</target>

	<target name="compile" depends="init" description="compile">
		<javac srcdir="src/main/java" destdir="build/classes" classpathref="compile.classpath" />
	</target>

	<target name="build" depends="compile">
		<spring-boot:exejar destfile="build/myapp.jar" classes="build/classes">
			<spring-boot:lib>
				<fileset dir="lib/runtime" />
			</spring-boot:lib>
		</spring-boot:exejar>
	</target>
</project>
```

> 如果您不想使用 `spring-boot-antlib` 模块，请参阅[Section 91.9, “Build an Executable Archive from Ant without Using spring-boot-antlib”](howto-build.html#howto-build-an-executable-archive-with-ant)“操作方法”.

## 13.5初学者

启动器是一组方便的依赖关系描述符，您可以将其包含在应用程序中.您可以获得所需的所有Spring和相关技术的一站式服务，而无需查看示例代码和复制粘贴依赖描述符的负载.例如，如果要开始使用Spring和JPA进行数据库访问，请在项目中包含 `spring-boot-starter-data-jpa` 依赖项.

该starters包含许多依赖项，这些依赖项是使项目快速启动和运行所需的依赖项，以及一组受支持的托管传递依赖项.

----
**What’s in a name** 

所有 **official** 初学者都遵循类似的命名模式;  `spring-boot-starter-*` ，其中 `*` 是一种特殊类型的应用程序.此命名结构旨在帮助您找到启动器.许多IDE中的Maven集成允许您按名称搜索依赖项.例如，安装了相应的Eclipse或STS插件后，您可以在POM编辑器中按 `ctrl-space` 并键入“spring-boot-starter”以获取完整列表.

正如“[Creating Your Own Starter](boot-features-developing-auto-configuration.html#boot-features-custom-starter)”部分所述，第三方启动器不应该以 `spring-boot` 开头，因为它是为官方Spring Boot工件保留的.相反，第三方启动器通常以项目名称开头.例如，名为 `thirdpartyproject` 的第三方入门项目通常名为 `thirdpartyproject-spring-boot-starter` .

----

以下应用程序启动程序由Spring Boot在 `org.springframework.boot` 组下提供：

**Table 13.1. Spring Boot application starters** 

|名称|说明|双龙|
| ---- | ---- | ---- |
|  `spring-boot-starter`  |核心启动器，包括自动配置支持，日志记录和YAML | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
|  `spring-boot-starter-activemq`  |使用Apache ActiveMQ启动JMS消息传递| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
|  `spring-boot-starter-amqp`  |使用Spring AMQP和Rabbit MQ的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
|  `spring-boot-starter-aop`  |使用Spring AOP和AspectJ | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |进行面向方面编程的初学者
|  `spring-boot-starter-artemis`  |使用Apache Artemis进行JMS消息传递的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml) |
|  `spring-boot-starter-batch`  |使用Spring Batch | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml) |的初学者
|  `spring-boot-starter-cache`  |使用Spring Framework缓存支持的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
|  `spring-boot-starter-cloud-connectors`  |使用Spring Cloud Connectors的初学者，它简化了Cloud Foundry和Heroku等Cloud平台中服务的连接| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml) |
|  `spring-boot-starter-data-cassandra`  |使用Cassandra分布式数据库和Spring Data Cassandra的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml) |
|  `spring-boot-starter-data-cassandra-reactive`  |使用Cassandra分布式数据库和Spring Data的初学者Cassandra Reactive | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
|  `spring-boot-starter-data-couchbase`  |使用Couchbase面向文档的数据库和Spring Data Couchbase的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml) |
|  `spring-boot-starter-data-couchbase-reactive`  |使用Couchbase面向文档的数据库和Spring Data Couchbase Reactive的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml) |
|  `spring-boot-starter-data-elasticsearch`  |使用Elasticsearch搜索和分析引擎和Spring Data Elasticsearch的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
|  `spring-boot-starter-data-jdbc`  |使用Spring Data JDBC的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jdbc/pom.xml) |
|  `spring-boot-starter-data-jpa`  |使用Spring Data JPA和Hibernate的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
|  `spring-boot-starter-data-ldap`  |使用Spring Data LDAP的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml) |
|  `spring-boot-starter-data-mongodb`  |使用MongoDB面向文档的数据库和Spring Data MongoDB的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
|  `spring-boot-starter-data-mongodb-reactive`  |使用MongoDB面向文档的数据库和Spring Data MongoDB Reactive的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml) |
|  `spring-boot-starter-data-neo4j`  |使用Neo4j图形数据库和Spring Data Neo4j的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml) |
|  `spring-boot-starter-data-redis`  |使用Spring Data Redis和Lettuce客户端使用Redis键值数据存储的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
|  `spring-boot-starter-data-redis-reactive`  |使用带有Spring Data Redis的Redis键值数据存储和Lettuce客户端的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml) |
|  `spring-boot-starter-data-rest`  |使用Spring Data REST通过REST公开Spring Data存储库的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml) |
|  `spring-boot-starter-data-solr`  |使用带有Spring Data Solr | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml) |的Apache Solr搜索平台的初学者
|  `spring-boot-starter-freemarker`  |使用FreeMarker视图构建MVC Web应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml) |
|  `spring-boot-starter-groovy-templates`  |使用Groovy模板视图构建MVC Web应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml) |
|  `spring-boot-starter-hateoas`  |使用Spring MVC和Spring HATEOAS构建基于超媒体的RESTful Web应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml) |
|  `spring-boot-starter-integration`  |使用Spring Integration的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml) |
|  `spring-boot-starter-jdbc`  |将JDBC与HikariCP连接池一起使用的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml) |
|  `spring-boot-starter-jersey`  |使用JAX-RS和Jersey构建RESTful Web应用程序的初学者. [spring-boot-starter-web](using-boot-build-systems.html#spring-boot-starter-web) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml) |的替代方案
|  `spring-boot-starter-jooq`  |使用jOOQ访问SQL数据库的初学者. [spring-boot-starter-data-jpa](using-boot-build-systems.html#spring-boot-starter-data-jpa)或[spring-boot-starter-jdbc](using-boot-build-systems.html#spring-boot-starter-jdbc) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml) |的替代方案
|  `spring-boot-starter-json`  |用于读写json | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml) |的入门
|  `spring-boot-starter-jta-atomikos`  |使用Atomikos | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |的JTA事务的入门者
|  `spring-boot-starter-jta-bitronix`  |使用Bitronix | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml) |的JTA事务的入门者
|  `spring-boot-starter-mail`  |使用Java Mail和Spring Framework的电子邮件发送支持的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
|  `spring-boot-starter-mustache`  |使用Mustache视图构建Web应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml) |
|  `spring-boot-starter-oauth2-client`  |使用Spring Security的OAuth2 / OpenID Connect客户端功能的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-client/pom.xml) |
|  `spring-boot-starter-oauth2-resource-server`  |使用Spring Security的OAuth2资源服务器功能的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-resource-server/pom.xml) |
|  `spring-boot-starter-quartz`  |使用Quartz调度程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
|  `spring-boot-starter-security`  |使用Spring Security | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |的初学者
|  `spring-boot-starter-test`  |使用JUnit，Hamcrest和Mockito等库测试Spring Boot应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
|  `spring-boot-starter-thymeleaf`  |使用Thymeleaf视图构建MVC Web应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
|  `spring-boot-starter-validation`  |使用Hibernate Validator进行Java Bean验证的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
|  `spring-boot-starter-web`  |使用Spring MVC构建Web（包括RESTful）应用程序的初学者.使用Tomcat作为默认嵌入式容器| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
|  `spring-boot-starter-web-services`  |使用Spring Web Services的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml) |
|  `spring-boot-starter-webflux`  |使用Spring Framework的Reactive Web支持构建WebFlux应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml) |
|  `spring-boot-starter-websocket`  |使用Spring Framework的WebSocket支持构建WebSocket应用程序的初学者| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |

在除了应用程序启动器，以下启动器可用于添加[production ready](production-ready.html)功能：

**Table 13.2. Spring Boot production starters** 

|名称|说明|双龙|
| ---- | ---- | ---- |
|  `spring-boot-starter-actuator`  |使用Spring Boot的Actuator的初学者，它提供生产环境就绪功能，帮助您监控和管理您的应用程序| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) |

最后，如果要排除或交换特定的技术方面，Spring Boot还包括以下启动器：

**Table 13.3. Spring Boot technical starters** 

|名称|说明|双龙|
| ---- | ---- | ---- |
|  `spring-boot-starter-jetty`  |使用Jetty作为嵌入式servlet容器的初学者. [spring-boot-starter-tomcat](using-boot-build-systems.html#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml) |的替代方案
|  `spring-boot-starter-log4j2`  |使用Log4j2进行日志记录的入门者. [spring-boot-starter-logging](using-boot-build-systems.html#spring-boot-starter-logging) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml) |的替代方案
|  `spring-boot-starter-logging`  |使用Logback进行日志记录的Starter.默认日志记录启动| [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml) |
|  `spring-boot-starter-reactor-netty`  |使用Reactor Netty作为嵌入式响应式HTTP服务器的初学者. | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
|  `spring-boot-starter-tomcat`  |使用Tomcat作为嵌入式servlet容器的初学者. [spring-boot-starter-web](using-boot-build-systems.html#spring-boot-starter-web) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml) |使用的默认servlet容器启动器
|  `spring-boot-starter-undertow`  |使用Undertow作为嵌入式servlet容器的初学者. [spring-boot-starter-tomcat](using-boot-build-systems.html#spring-boot-starter-tomcat) | [Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml) |的替代方案

> 有关其他社区贡献的启动器的列表，请参阅GitHub上 `spring-boot-starters` 模块中的[README file](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc).


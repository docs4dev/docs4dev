## 13. Build Systems

It is strongly recommended that you choose a build system that supports [dependency management](using-boot-build-systems.html#using-boot-dependency-management) and that can consume artifacts published to the “Maven Central” repository. We would recommend that you choose Maven or Gradle. It is possible to get Spring Boot to work with other build systems (Ant, for example), but they are not particularly well supported.

## 13.1 Dependency Management

Each release of Spring Boot provides a curated list of dependencies that it supports. In practice, you do not need to provide a version for any of these dependencies in your build configuration, as Spring Boot manages that for you. When you upgrade Spring Boot itself, these dependencies are upgraded as well in a consistent way.

> You can still specify a version and override Spring Boot’s recommendations if you need to do so.

The curated list contains all the spring modules that you can use with Spring Boot as well as a refined list of third party libraries. The list is available as a standard [Bills of Materials (spring-boot-dependencies)](using-boot-build-systems.html#using-boot-maven-without-a-parent) that can be used with both [Maven](using-boot-build-systems.html#using-boot-maven-parent-pom) and [Gradle](using-boot-build-systems.html#using-boot-gradle).

> Each release of Spring Boot is associated with a base version of the Spring Framework. We  **highly**  recommend that you not specify its version.

## 13.2 Maven

Maven users can inherit from the  `spring-boot-starter-parent`  project to obtain sensible defaults. The parent project provides the following features:

- Java 1.8 as the default compiler level.

- UTF-8 source encoding.

- A [Dependency Management section](using-boot-build-systems.html#using-boot-dependency-management), inherited from the spring-boot-dependencies pom, that manages the versions of common dependencies. This dependency management lets you omit <version> tags for those dependencies when used in your own pom.

- An execution of the [repackage goal](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/maven-plugin/repackage-mojo.html) with a  `repackage`  execution id.

- Sensible [resource filtering](https://maven.apache.org/plugins/maven-resources-plugin/examples/filter.html).

- Sensible plugin configuration ([exec plugin](http://www.mojohaus.org/exec-maven-plugin/), [Git commit ID](https://github.com/ktoso/maven-git-commit-id-plugin), and [shade](https://maven.apache.org/plugins/maven-shade-plugin/)).

- Sensible resource filtering for  `application.properties`  and  `application.yml`  including profile-specific files (for example,  `application-dev.properties`  and  `application-dev.yml` )

Note that, since the  `application.properties`  and  `application.yml`  files accept Spring style placeholders ( `${…}` ), the Maven filtering is changed to use  `@[email protected]`  placeholders. (You can override that by setting a Maven property called  `resource.delimiter` .)

### 13.2.1 Inheriting the Starter Parent

To configure your project to inherit from the  `spring-boot-starter-parent` , set the  `parent`  as follows:

```xml
<!-- Inherit defaults from Spring Boot -->
<parent>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-parent</artifactId>
	<version>2.1.0.RELEASE</version>
</parent>
```

> You should need to specify only the Spring Boot version number on this dependency. If you import additional starters, you can safely omit the version number.

With that setup, you can also override individual dependencies by overriding a property in your own project. For instance, to upgrade to another Spring Data release train, you would add the following to your  `pom.xml` :

```xml
<properties>
	<spring-data-releasetrain.version>Fowler-SR2</spring-data-releasetrain.version>
</properties>
```

> Check the [spring-boot-dependencies pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-dependencies/pom.xml) for a list of supported properties.

### 13.2.2 Using Spring Boot without the Parent POM

Not everyone likes inheriting from the  `spring-boot-starter-parent`  POM. You may have your own corporate standard parent that you need to use or you may prefer to explicitly declare all your Maven configuration.

If you do not want to use the  `spring-boot-starter-parent` , you can still keep the benefit of the dependency management (but not the plugin management) by using a  `scope=import`  dependency, as follows:

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

The preceding sample setup does not let you override individual dependencies by using a property, as explained above. To achieve the same result, you need to add an entry in the  `dependencyManagement`  of your project  **before**  the  `spring-boot-dependencies`  entry. For instance, to upgrade to another Spring Data release train, you could add the following element to your  `pom.xml` :

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

> In the preceding example, we specify a BOM, but any dependency type can be overridden in the same way.

### 13.2.3 Using the Spring Boot Maven Plugin

Spring Boot includes a [Maven plugin](build-tool-plugins-maven-plugin.html) that can package the project as an executable jar. Add the plugin to your  `<plugins>`  section if you want to use it, as shown in the following example:

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

> If you use the Spring Boot starter parent pom, you need to add only the plugin. There is no need to configure it unless you want to change the settings defined in the parent.

## 13.3 Gradle

To learn about using Spring Boot with Gradle, please refer to the documentation for Spring Boot’s Gradle plugin:

- Reference ([HTML](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/html) and [PDF](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/reference/pdf/spring-boot-gradle-plugin-reference.pdf))

- [API](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/gradle-plugin/api)

## 13.4 Ant

It is possible to build a Spring Boot project using Apache Ant+Ivy. The  `spring-boot-antlib`  “AntLib” module is also available to help Ant create executable jars.

To declare dependencies, a typical  `ivy.xml`  file looks something like the following example:

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

A typical  `build.xml`  looks like the following example:

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

> If you do not want to use the  `spring-boot-antlib`  module, see the [Section 91.9, “Build an Executable Archive from Ant without Using spring-boot-antlib”](howto-build.html#howto-build-an-executable-archive-with-ant) “How-to” .

## 13.5 Starters

Starters are a set of convenient dependency descriptors that you can include in your application. You get a one-stop shop for all the Spring and related technologies that you need without having to hunt through sample code and copy-paste loads of dependency descriptors. For example, if you want to get started using Spring and JPA for database access, include the  `spring-boot-starter-data-jpa`  dependency in your project.

The starters contain a lot of the dependencies that you need to get a project up and running quickly and with a consistent, supported set of managed transitive dependencies.

----
**What’s in a name** 

All  **official**  starters follow a similar naming pattern;  `spring-boot-starter-*` , where  `*`  is a particular type of application. This naming structure is intended to help when you need to find a starter. The Maven integration in many IDEs lets you search dependencies by name. For example, with the appropriate Eclipse or STS plugin installed, you can press  `ctrl-space`  in the POM editor and type “spring-boot-starter” for a complete list.

As explained in the “[Creating Your Own Starter](boot-features-developing-auto-configuration.html#boot-features-custom-starter)” section, third party starters should not start with  `spring-boot` , as it is reserved for official Spring Boot artifacts. Rather, a third-party starter typically starts with the name of the project. For example, a third-party starter project called  `thirdpartyproject`  would typically be named  `thirdpartyproject-spring-boot-starter` .

----

The following application starters are provided by Spring Boot under the  `org.springframework.boot`  group:

**Table 13.1. Spring Boot application starters** 

|Name|Description|Pom|
|----|----|----|
| `spring-boot-starter`  |Core starter, including auto-configuration support, logging and YAML |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter/pom.xml) |
| `spring-boot-starter-activemq`  |Starter for JMS messaging using Apache ActiveMQ |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-activemq/pom.xml) |
| `spring-boot-starter-amqp`  |Starter for using Spring AMQP and Rabbit MQ |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-amqp/pom.xml) |
| `spring-boot-starter-aop`  |Starter for aspect-oriented programming with Spring AOP and AspectJ |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-aop/pom.xml) |
| `spring-boot-starter-artemis`  |Starter for JMS messaging using Apache Artemis |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-artemis/pom.xml) |
| `spring-boot-starter-batch`  |Starter for using Spring Batch |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-batch/pom.xml) |
| `spring-boot-starter-cache`  |Starter for using Spring Framework’s caching support |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cache/pom.xml) |
| `spring-boot-starter-cloud-connectors`  |Starter for using Spring Cloud Connectors which simplifies connecting to services in cloud platforms like Cloud Foundry and Heroku |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-cloud-connectors/pom.xml) |
| `spring-boot-starter-data-cassandra`  |Starter for using Cassandra distributed database and Spring Data Cassandra |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra/pom.xml) |
| `spring-boot-starter-data-cassandra-reactive`  |Starter for using Cassandra distributed database and Spring Data Cassandra Reactive |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-cassandra-reactive/pom.xml) |
| `spring-boot-starter-data-couchbase`  |Starter for using Couchbase document-oriented database and Spring Data Couchbase |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase/pom.xml) |
| `spring-boot-starter-data-couchbase-reactive`  |Starter for using Couchbase document-oriented database and Spring Data Couchbase Reactive |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-couchbase-reactive/pom.xml) |
| `spring-boot-starter-data-elasticsearch`  |Starter for using Elasticsearch search and analytics engine and Spring Data Elasticsearch |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-elasticsearch/pom.xml) |
| `spring-boot-starter-data-jdbc`  |Starter for using Spring Data JDBC |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jdbc/pom.xml) |
| `spring-boot-starter-data-jpa`  |Starter for using Spring Data JPA with Hibernate |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-jpa/pom.xml) |
| `spring-boot-starter-data-ldap`  |Starter for using Spring Data LDAP |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-ldap/pom.xml) |
| `spring-boot-starter-data-mongodb`  |Starter for using MongoDB document-oriented database and Spring Data MongoDB |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb/pom.xml) |
| `spring-boot-starter-data-mongodb-reactive`  |Starter for using MongoDB document-oriented database and Spring Data MongoDB Reactive |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-mongodb-reactive/pom.xml) |
| `spring-boot-starter-data-neo4j`  |Starter for using Neo4j graph database and Spring Data Neo4j |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-neo4j/pom.xml) |
| `spring-boot-starter-data-redis`  |Starter for using Redis key-value data store with Spring Data Redis and the Lettuce client |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis/pom.xml) |
| `spring-boot-starter-data-redis-reactive`  |Starter for using Redis key-value data store with Spring Data Redis reactive and the Lettuce client |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-redis-reactive/pom.xml) |
| `spring-boot-starter-data-rest`  |Starter for exposing Spring Data repositories over REST using Spring Data REST |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-rest/pom.xml) |
| `spring-boot-starter-data-solr`  |Starter for using the Apache Solr search platform with Spring Data Solr |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-data-solr/pom.xml) |
| `spring-boot-starter-freemarker`  |Starter for building MVC web applications using FreeMarker views |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-freemarker/pom.xml) |
| `spring-boot-starter-groovy-templates`  |Starter for building MVC web applications using Groovy Templates views |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-groovy-templates/pom.xml) |
| `spring-boot-starter-hateoas`  |Starter for building hypermedia-based RESTful web application with Spring MVC and Spring HATEOAS |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-hateoas/pom.xml) |
| `spring-boot-starter-integration`  |Starter for using Spring Integration |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-integration/pom.xml) |
| `spring-boot-starter-jdbc`  |Starter for using JDBC with the HikariCP connection pool |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jdbc/pom.xml) |
| `spring-boot-starter-jersey`  |Starter for building RESTful web applications using JAX-RS and Jersey. An alternative to [spring-boot-starter-web](using-boot-build-systems.html#spring-boot-starter-web) |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jersey/pom.xml) |
| `spring-boot-starter-jooq`  |Starter for using jOOQ to access SQL databases. An alternative to [spring-boot-starter-data-jpa](using-boot-build-systems.html#spring-boot-starter-data-jpa) or [spring-boot-starter-jdbc](using-boot-build-systems.html#spring-boot-starter-jdbc) |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jooq/pom.xml) |
| `spring-boot-starter-json`  |Starter for reading and writing json |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-json/pom.xml) |
| `spring-boot-starter-jta-atomikos`  |Starter for JTA transactions using Atomikos |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-atomikos/pom.xml) |
| `spring-boot-starter-jta-bitronix`  |Starter for JTA transactions using Bitronix |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jta-bitronix/pom.xml) |
| `spring-boot-starter-mail`  |Starter for using Java Mail and Spring Framework’s email sending support |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mail/pom.xml) |
| `spring-boot-starter-mustache`  |Starter for building web applications using Mustache views |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-mustache/pom.xml) |
| `spring-boot-starter-oauth2-client`  |Starter for using Spring Security’s OAuth2/OpenID Connect client features |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-client/pom.xml) |
| `spring-boot-starter-oauth2-resource-server`  |Starter for using Spring Security’s OAuth2 resource server features |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-oauth2-resource-server/pom.xml) |
| `spring-boot-starter-quartz`  |Starter for using the Quartz scheduler |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-quartz/pom.xml) |
| `spring-boot-starter-security`  |Starter for using Spring Security |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-security/pom.xml) |
| `spring-boot-starter-test`  |Starter for testing Spring Boot applications with libraries including JUnit, Hamcrest and Mockito |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-test/pom.xml) |
| `spring-boot-starter-thymeleaf`  |Starter for building MVC web applications using Thymeleaf views |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-thymeleaf/pom.xml) |
| `spring-boot-starter-validation`  |Starter for using Java Bean Validation with Hibernate Validator |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-validation/pom.xml) |
| `spring-boot-starter-web`  |Starter for building web, including RESTful, applications using Spring MVC. Uses Tomcat as the default embedded container |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web/pom.xml) |
| `spring-boot-starter-web-services`  |Starter for using Spring Web Services |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-web-services/pom.xml) |
| `spring-boot-starter-webflux`  |Starter for building WebFlux applications using Spring Framework’s Reactive Web support |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-webflux/pom.xml) |
| `spring-boot-starter-websocket`  |Starter for building WebSocket applications using Spring Framework’s WebSocket support |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-websocket/pom.xml) |

In addition to the application starters, the following starters can be used to add [production ready](production-ready.html) features:

**Table 13.2. Spring Boot production starters** 

|Name|Description|Pom|
|----|----|----|
| `spring-boot-starter-actuator`  |Starter for using Spring Boot’s Actuator which provides production ready features to help you monitor and manage your application |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-actuator/pom.xml) |

Finally, Spring Boot also includes the following starters that can be used if you want to exclude or swap specific technical facets:

**Table 13.3. Spring Boot technical starters** 

|Name|Description|Pom|
|----|----|----|
| `spring-boot-starter-jetty`  |Starter for using Jetty as the embedded servlet container. An alternative to [spring-boot-starter-tomcat](using-boot-build-systems.html#spring-boot-starter-tomcat) |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-jetty/pom.xml) |
| `spring-boot-starter-log4j2`  |Starter for using Log4j2 for logging. An alternative to [spring-boot-starter-logging](using-boot-build-systems.html#spring-boot-starter-logging) |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-log4j2/pom.xml) |
| `spring-boot-starter-logging`  |Starter for logging using Logback. Default logging starter |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-logging/pom.xml) |
| `spring-boot-starter-reactor-netty`  |Starter for using Reactor Netty as the embedded reactive HTTP server. |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-reactor-netty/pom.xml) |
| `spring-boot-starter-tomcat`  |Starter for using Tomcat as the embedded servlet container. Default servlet container starter used by [spring-boot-starter-web](using-boot-build-systems.html#spring-boot-starter-web) |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-tomcat/pom.xml) |
| `spring-boot-starter-undertow`  |Starter for using Undertow as the embedded servlet container. An alternative to [spring-boot-starter-tomcat](using-boot-build-systems.html#spring-boot-starter-tomcat) |[Pom](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-starters/spring-boot-starter-undertow/pom.xml) |

> For a list of additional community contributed starters, see the [README file](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters/README.adoc) in the  `spring-boot-starters`  module on GitHub.


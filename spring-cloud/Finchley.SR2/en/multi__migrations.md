## 99. Migrations

> For up to date migration guides please visit the project’s [wiki page](https://github.com/spring-cloud/spring-cloud-contract/wiki/).

This section covers migrating from one version of Spring Cloud Contract Verifier to the next version. It covers the following versions upgrade paths:

## 99.1 1.0.x → 1.1.x

This section covers upgrading from version 1.0 to version 1.1.

### 99.1.1 New structure of generated stubs

In  `1.1.x`  we have introduced a change to the structure of generated stubs. If you have been using the  `@AutoConfigureWireMock`  notation to use the stubs from the classpath, it no longer works. The following example shows how the  `@AutoConfigureWireMock`  notation used to work:

```java
@AutoConfigureWireMock(stubs = "classpath:/customer-stubs/mappings", port = 8084)
```

You must either change the location of the stubs to:  `classpath:…/META-INF/groupId/artifactId/version/mappings`  or use the new classpath-based  `@AutoConfigureStubRunner` , as shown in the following example:

```java
@AutoConfigureWireMock(stubs = "classpath:customer-stubs/META-INF/travel.components/customer-contract/1.0.2-SNAPSHOT/mappings/", port = 8084)
```

If you do not want to use  `@AutoConfigureStubRunner`  and you want to remain with the old structure, set your plugin tasks accordingly. The following example would work for the structure presented in the previous snippet.

**Maven.**  

```xml
<!-- start of pom.xml -->

<properties>
<!-- we don't want the verifier to do a jar for us -->
<spring.cloud.contract.verifier.skip>true</spring.cloud.contract.verifier.skip>
</properties>

<!-- ... -->

<!-- You need to set up the assembly plugin -->
<build>
<plugins>
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-assembly-plugin</artifactId>
<executions>
<execution>
<id>stub</id>
<phase>prepare-package</phase>
<goals>
<goal>single</goal>
</goals>
<inherited>false</inherited>
<configuration>
<attach>true</attach>
<descriptor>$../../../../src/assembly/stub.xml</descriptor>
</configuration>
</execution>
</executions>
</plugin>
</plugins>
</build>
<!-- end of pom.xml -->

<!-- start of stub.xml-->

<assembly
	xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3 http://maven.apache.org/xsd/assembly-1.1.3.xsd">
	<id>stubs</id>
	<formats>
		<format>jar</format>
	</formats>
	<includeBaseDirectory>false</includeBaseDirectory>
	<fileSets>
		<fileSet>
			<directory>${project.build.directory}/snippets/stubs</directory>
			<outputDirectory>customer-stubs/mappings</outputDirectory>
			<includes>
				<include>**/*</include>
			</includes>
		</fileSet>
		<fileSet>
			<directory>$../../../../src/test/resources/contracts</directory>
			<outputDirectory>customer-stubs/contracts</outputDirectory>
			<includes>
				<include>**/*.groovy</include>
			</includes>
		</fileSet>
	</fileSets>
</assembly>

<!-- end of stub.xml-->
```

**Gradle.**  

```java
task copyStubs(type: Copy, dependsOn: 'generateWireMockClientStubs') {
//    Preserve directory structure from 1.0.X of spring-cloud-contract
from "${project.buildDir}/resources/main/customer-stubs/META-INF/${project.group}/${project.name}/${project.version}"
into "${project.buildDir}/resources/main/customer-stubs"
}
```

## 99.2 1.1.x → 1.2.x

This section covers upgrading from version 1.1 to version 1.2.

### 99.2.1 Custom HttpServerStub

`HttpServerStub`  includes a method that was not in version 1.1. The method is  `String registeredMappings()`  If you have classes that implement  `HttpServerStub` , you now have to implement the  `registeredMappings()`  method. It should return a  `String`  representing all mappings available in a single  `HttpServerStub` .

See [issue 355](https://github.com/spring-cloud/spring-cloud-contract/issues/355) for more detail.

### 99.2.2 New packages for generated tests

The flow for setting the generated tests package name will look like this:

- Set  `basePackageForTests` 

- If  `basePackageForTests`  was not set, pick the package from  `baseClassForTests` 

- If  `baseClassForTests`  was not set, pick  `packageWithBaseClasses` 

- If nothing got set, pick the default value:  `org.springframework.cloud.contract.verifier.tests` 

See [issue 260](https://github.com/spring-cloud/spring-cloud-contract/issues/260) for more detail.

### 99.2.3 New Methods in TemplateProcessor

In order to add support for  `fromRequest.path` , the following methods had to be added to the  `TemplateProcessor`  interface:

-  `path()` 

-  `path(int index)` 

See [issue 388](https://github.com/spring-cloud/spring-cloud-contract/issues/388) for more detail.

### 99.2.4 RestAssured 3.0

Rest Assured, used in the generated test classes, got bumped to  `3.0` . If you manually set versions of Spring Cloud Contract and the release train you might see the following exception:

```java
Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile (default-testCompile) on project some-project: Compilation failure: Compilation failure:
[ERROR] /some/path/SomeClass.java:[4,39] package com.jayway.restassured.response does not exist
```

This exception will occur due to the fact that the tests got generated with an old version of plugin and at test execution time you have an incompatible version of the release train (and vice versa).

Done via [issue 267](https://github.com/spring-cloud/spring-cloud-contract/issues/267)

## 99.3 1.2.x → 2.0.x

### 99.3.1 No Camel support

We will add back Apache Camel support only after this [issue](https://issues.apache.org/jira/browse/CAMEL-11430) gets fixed


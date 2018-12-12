## 99.迁移

> 有关最新的迁移指南，请访问项目的[wiki page](https://github.com/spring-cloud/spring-cloud-contract/wiki/).

本节介绍从一个版本的Spring Cloud Contract Verifier迁移到下一个版本.它涵盖了以下版本的升级路径：

## 99.1 1.0.x→1.1.x

本节介绍从1.0版升级到1.1版.

### 99.1.1生成的存根的新结构

在 `1.1.x` 中，我们引入了对生成的存根结构的更改.如果您一直使用 `@AutoConfigureWireMock` 表示法来使用类路径中的存根，则它不再有效.以下示例显示 `@AutoConfigureWireMock` 符号如何工作：

```java
@AutoConfigureWireMock(stubs = "classpath:/customer-stubs/mappings", port = 8084)
```

您必须将存根的位置更改为： `classpath:…/META-INF/groupId/artifactId/version/mappings` 或使用新的基于类路径的 `@AutoConfigureStubRunner` ，如以下示例所示：

```java
@AutoConfigureWireMock(stubs = "classpath:customer-stubs/META-INF/travel.components/customer-contract/1.0.2-SNAPSHOT/mappings/", port = 8084)
```

如果您不想使用 `@AutoConfigureStubRunner` 并且希望保留旧结构，请相应地设置插件任务.以下示例适用于上一个代码段中显示的结构.

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

## 99.2 1.1.x→1.2.x

本节介绍从1.1版升级到1.2版.

### 99.2.1自定义HttpServerStub

`HttpServerStub` 包含一个不在1.1版中的方法.方法是 `String registeredMappings()` 如果您有实现 `HttpServerStub` 的类，则现在必须实现 `registeredMappings()` 方法.它应该返回 `String` ，表示单个 `HttpServerStub` 中可用的所有映射.

有关详细信息，请参阅[issue 355](https://github.com/spring-cloud/spring-cloud-contract/issues/355).

### 99.2.2生成测试的新包

设置生成的测试包名称的流程如下所示：

- Set  `basePackageForTests` 

- 如果 `basePackageForTests` 未设置，请从 `baseClassForTests` 中选择包

- 如果 `baseClassForTests` 未设置，请选择 `packageWithBaseClasses` 

- 如果没有设置，请选择默认值： `org.springframework.cloud.contract.verifier.tests` 

有关详细信息，请参阅[issue 260](https://github.com/spring-cloud/spring-cloud-contract/issues/260).

### 99.2.3 TemplateProcessor中的新方法

为了添加对 `fromRequest.path` 的支持，必须在 `TemplateProcessor` 接口中添加以下方法：

-  `path()` 

-  `path(int index)` 

有关详细信息，请参阅[issue 388](https://github.com/spring-cloud/spring-cloud-contract/issues/388).

### 99.2.4 RestAssured 3.0

在生成的测试类中使用的Rest Assured被撞到 `3.0` .如果手动设置Spring Cloud Contract和版本系列的版本，您可能会看到以下异常：

```java
Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:testCompile (default-testCompile) on project some-project: Compilation failure: Compilation failure:
[ERROR] /some/path/SomeClass.java:[4,39] package com.jayway.restassured.response does not exist
```

发生此异常的原因是测试是使用旧版本的插件生成的，并且在测试执行时您具有版本系列的不兼容版本（反之亦然）.

通过[issue 267](https://github.com/spring-cloud/spring-cloud-contract/issues/267)完成

## 99.3 1.2.x→2.0.x

### 99.3.1没有骆驼支持

只有在[issue](https://issues.apache.org/jira/browse/CAMEL-11430)修复后才会添加Apache Camel支持


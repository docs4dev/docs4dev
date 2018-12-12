## 90. Spring Cloud Contract FAQ

## 90.1为什么要使用Spring Cloud Contract Verifier而不是X？

目前，Spring Cloud Contract是一个基于JVM的工具.因此，当您已经为JVM创建软件时，它可能是您的第一选择.该项目有许多非常有趣的功能，但尤其是其中相当一部分确实使Spring Cloud Contract Verifier在消费者驱动Contract（CDC）工具的“市场”中脱颖而出.在许多最有趣的是：

_0009可以通过消息传递进行CDC

- 清晰易用，静态类型的DSL
_0009可以将当前JSON文件粘贴到Contract中，并仅编辑其元素

- 从定义的Contract中自动生成测试

- Stub Runner功能 - 存根在运行时从Nexus / Artifactory自动下载

- Spring Cloud集成 - 没有发现服务集成测试需要

- Spring Cloud Contract与Pact开箱即用，并提供简单的钩子来扩展其功能

- Via Docker增加了对所用语言和框架的支持

## 90.2我不想在Groovy写一份Contract！

没问题.你可以在YAML写一份Contract！

## 90.3这个值是什么（consumer（），producer（））？

与存根相关的最大挑战之一是它们的可重用性.只有它们可以被广泛使用，它们才能满足它们的目的.通常使这种困难的是请求/响应元素的硬编码值.例如日期或ID.想象一下以下JSON请求

```java
{
"time" : "2016-10-10 20:10:15",
"id" : "9febab1c-6f36-4a0b-88d6-3b6a6d81cd4a",
"body" : "foo"
}
```

和JSON响应

```java
{
"time" : "2016-10-10 21:10:15",
"id" : "c4231e1f-3ca9-48d3-b7e7-567d55f0d051",
"body" : "bar"
}
```

想象一下，通过更改系统中的时钟或提供数据提供程序的存根实现，设置 `time` 字段的正确值（让我们假设此内容由数据库生成）所需的痛苦.这与称为 `id` 的字段有关.你会创建一个UUID生成器的存根实现吗？没什么意义......

因此，作为消费者，您希望发送与任何形式的时间或任何UUID匹配的请求.这样你的系统将照常工作 - 将生成数据，你不必存根.让我们假设在上述JSON的情况下，最重要的部分是 `body` 字段.您可以专注于此并为其他字段提供匹配.换句话说，你希望存根像这样工作：

```java
{
"time" : "SOMETHING THAT MATCHES TIME",
"id" : "SOMETHING THAT MATCHES UUID",
"body" : "foo"
}
```

就响应作为消费者而言，您需要一个可以操作的具体值.所以这样的JSON是有效的

```java
{
"time" : "2016-10-10 21:10:15",
"id" : "c4231e1f-3ca9-48d3-b7e7-567d55f0d051",
"body" : "bar"
}
```

正如您在前面部分中看到的，我们从Contract中生成测试.因此，从生产环境者的角度来看，情况看起来大不相同.我们正在解析提供的Contract，在测试中我们希望向您的endpoints发送实际请求.因此，对于请求的生产环境者，我们不能进行任何匹配.我们需要生产环境者后端可以使用的具体值.这样的JSON是有效的：

```java
{
"time" : "2016-10-10 20:10:15",
"id" : "9febab1c-6f36-4a0b-88d6-3b6a6d81cd4a",
"body" : "foo"
}
```

另一方面，从Contract有效性的角度来看，响应不一定必须包含 `time` 或 `id` 的具体值.假设您在生产环境者方面生成了那些 - 再次，您必须进行大量的存根以确保始终返回相同的值.这就是为什么从制作人那里你可能想要的是以下响应：

```java
{
"time" : "SOMETHING THAT MATCHES TIME",
"id" : "SOMETHING THAT MATCHES UUID",
"body" : "bar"
}
```

那么你如何为消费者提供一次匹配器，为生产环境者提供具体Value，反之亦然？在Spring Cloud Contract中，我们允许您提供 **dynamic value** .这意味着通信双方可能会有所不同.您可以传递值：

通过 `value` 方法

```java
value(consumer(...), producer(...))
value(stub(...), test(...))
value(client(...), server(...))
```

或使用 `$()` 方法

```java
$(consumer(...), producer(...))
$(stub(...), test(...))
$(client(...), server(...))
```

您可以在[Chapter 95, Contract DSL](multi_contract-dsl.html)部分阅读更多相关信息.

调用 `value()` 或 `$()` 告诉Spring Cloud Contract您将传递动态值.在 `consumer()` 方法中，您传递应在消费者端（在生成的存根中）使用的值.在 `producer()` 方法中，您传递应该在生产环境者端使用的值（在生成的测试中）.

> 如果你已经传递了正则表达式并且没有通过另一方，那么另一方将自动生成.

通常，您将使用该方法和 `regex` 辅助方法.例如.  `consumer(regex('[0-9]{10}'))` .

总结一下，前面提到的场景的Contract看起来或多或少就像这样（时间和UUID的正则表达式被简化，很可能无效，但我们希望在这个例子中保持简单）：

```java
org.springframework.cloud.contract.spec.Contract.make {
				request {
					method 'GET'
					url '/someUrl'
					body([
					    time : value(consumer(regex('[0-9]{4}-[0-9]{2}-[0-9]{2} [0-2][0-9]-[0-5][0-9]-[0-5][0-9]')),
					    id: value(consumer(regex('[0-9a-zA-z]{8}-[0-9a-zA-z]{4}-[0-9a-zA-z]{4}-[0-9a-zA-z]{12}'))
					    body: "foo"
					])
				}
			response {
				status OK()
				body([
					    time : value(producer(regex('[0-9]{4}-[0-9]{2}-[0-9]{2} [0-2][0-9]-[0-5][0-9]-[0-5][0-9]')),
					    id: value([producer(regex('[0-9a-zA-z]{8}-[0-9a-zA-z]{4}-[0-9a-zA-z]{4}-[0-9a-zA-z]{12}'))
					    body: "bar"
					])
			}
}
```

|图片/ important.png |重要|
| ---- | ---- |
|请阅读[Groovy docs related to JSON](http://groovy-lang.org/json.html)以了解如何正确构建请求/响应正文. |

## 90.4如何进行Stubs版本控制？

### 90.4.1 API版本控制

让我们试着回答一个问题版本真正意味着什么.如果您指的是API版本，则有不同的方法.

- use超媒体，链接，不要以任何方式对您的API进行版本控制

- pass版本通过headers / urls

我不会试着回答哪个方法更好的问题.无论什么适合您的需求，并允许您产生业务Value应该被选中.

我们假设您对API进行了版本控制.在这种情况下，您应该提供与您支持的许多版本一样多的Contract.您可以为每个版本创建一个子文件夹，或将其附加到Contract名称 - 任何适合您的版本.

### 90.4.2 JAR版本控制

如果版本控制是指包含存根的JAR版本，那么基本上有两种主要方法.

让我们假设您正在进行持续交付/部署，这意味着每次通过管道时都会生成新版本的jar，并且该jar可以随时进入生产环境阶段.例如，您的jar版本看起来像这样（它Build于2016年10月20日20:15:21）：

```java
1.0.0.20161020-201521-RELEASE
```

在这种情况下，您生成的存根jar将如下所示.

```java
1.0.0.20161020-201521-RELEASE-stubs.jar
```

在这如果引用存根提供最新版本的存根，则应该在 `application.yml` 或 `@AutoConfigureStubRunner` 内.你可以通过传递 `+` 标志来做到这一点.例

```java
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:stubs:8080"})
```

如果版本控制是固定的（例如 `1.0.4.RELEASE` 或 `2.1.1` ），则必须设置jar版本的具体值.例2.1.1.

```java
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:2.1.1:stubs:8080"})
```

### 90.4.3 Dev或prod存根

您可以操作分类器以针对其他服务的存根的当前开发版本或部署到生产环境的服务的存根执行测试.如果在达到生产环境部署后更改构建以使用 `prod-stubs` 分类器部署存根，则可以在一个案例中使用dev存根和一个使用prod存根运行测试.

使用存根的开发版本的测试示例

```java
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:stubs:8080"})
```

使用存根的生产环境版本的测试示例

```java
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:prod-stubs:8080"})
```

您也可以通过部署管道中的属性传递这些值.

## 90.5与Contract的共同回购

存储Contract的另一种方式是将它们与生产环境者一起存储，这是将它们保存在一个共同的位置.它可能与消费者无法克隆生产环境者代码的安全问题有关.此外，如果您将Contract保留在一个地方，那么作为生产环境者，您将知道您拥有多少消费者以及您将使用当地更改打破哪些消费者.

### 90.5.1回购结构

假设我们有一个坐标为 `com.example:server` 且有3个消费者的生产环境者： `client1` ， `client2` ， `client3` .然后在具有常见Contract的存储库中，您将进行以下设置（您可以将其签出[here](https://github.com/spring-cloud/spring-cloud-contract/tree/2.0.x/samples/standalone/contracts)）：

```java
├── com
│ └── example
│     └── server
│         ├── client1
│         │ └── expectation.groovy
│         ├── client2
│         │ └── expectation.groovy
│         ├── client3
│         │ └── expectation.groovy
│         └── pom.xml
├── mvnw
├── mvnw.cmd
├── pom.xml
└── src
└── assembly
└── contracts.xml
```

如您所见，在斜线分隔的groupid  `/`  artifact id文件夹（ `com/example/server` ）下，您对3个消费者（ `client1` ， `client2` 和 `client3` ）有所期待.期望是本文档中描述的标准Groovy DSLContract文件.此存储库必须生成一个JAR文件，该文件将一对一映射到repo的内容.

`server` 文件夹中 `pom.xml` 的示例.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example</groupId>
	<artifactId>server</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<name>Server Stubs</name>
	<description>POM used to install locally stubs for consumer side</description>

	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.0.6.RELEASE</version>
		<relativePath />
	</parent>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
		<java.version>1.8</java.version>
		<spring-cloud-contract.version>2.0.3.BUILD-SNAPSHOT</spring-cloud-contract.version>
		<spring-cloud-release.version>Finchley.BUILD-SNAPSHOT</spring-cloud-release.version>
		<excludeBuildFolders>true</excludeBuildFolders>
	</properties>

	<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-dependencies</artifactId>
				<version>${spring-cloud-release.version}</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-contract-maven-plugin</artifactId>
				<version>${spring-cloud-contract.version}</version>
				<extensions>true</extensions>
				<configuration>
					<!-- By default it would search under src/test/resources/ -->
					<contractsDirectory>${project.basedir}</contractsDirectory>
				</configuration>
			</plugin>
		</plugins>
	</build>

	<repositories>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
		<repository>
			<id>spring-releases</id>
			<name>Spring Releases</name>
			<url>https://repo.spring.io/release</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</repository>
	</repositories>
	<pluginRepositories>
		<pluginRepository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</pluginRepository>
		<pluginRepository>
			<id>spring-milestones</id>
			<name>Spring Milestones</name>
			<url>https://repo.spring.io/milestone</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</pluginRepository>
		<pluginRepository>
			<id>spring-releases</id>
			<name>Spring Releases</name>
			<url>https://repo.spring.io/release</url>
			<snapshots>
				<enabled>false</enabled>
			</snapshots>
		</pluginRepository>
	</pluginRepositories>

</project>
```

正如您所看到的，除了Spring Cloud Contract Maven插件之外，没有任何依赖项.这些poms是消费者运行 `mvn clean install -DskipTests` 以在本地安装生产环境者项目的存根所必需的.

根文件夹中的 `pom.xml` 可能如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		 xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	<modelVersion>4.0.0</modelVersion>

	<groupId>com.example.standalone</groupId>
	<artifactId>contracts</artifactId>
	<version>0.0.1-SNAPSHOT</version>

	<name>Contracts</name>
	<description>Contains all the Spring Cloud Contracts, well, contracts. JAR used by the producers to generate tests and stubs</description>

	<properties>
		<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	</properties>

	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-assembly-plugin</artifactId>
				<executions>
					<execution>
						<id>contracts</id>
						<phase>prepare-package</phase>
						<goals>
							<goal>single</goal>
						</goals>
						<configuration>
							<attach>true</attach>
							<descriptor>${basedir}/src/assembly/contracts.xml</descriptor>
							<!-- If you want an explicit classifier remove the following line -->
							<appendAssemblyId>false</appendAssemblyId>
						</configuration>
					</execution>
				</executions>
			</plugin>
		</plugins>
	</build>

</project>
```

它使用程序集插件来构建包含所有Contract的JAR.此类设置的示例如下：

```xml
<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3"
		  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		  xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.3 http://maven.apache.org/xsd/assembly-1.1.3.xsd">
	<id>project</id>
	<formats>
		<format>jar</format>
	</formats>
	<includeBaseDirectory>false</includeBaseDirectory>
	<fileSets>
		<fileSet>
			<directory>${project.basedir}</directory>
			<outputDirectory>/</outputDirectory>
			<useDefaultExcludes>true</useDefaultExcludes>
			<excludes>
				<exclude>**/${project.build.directory}/**</exclude>
				<exclude>mvnw</exclude>
				<exclude>mvnw.cmd</exclude>
				<exclude>.mvn/**</exclude>
				<exclude>src/**</exclude>
			</excludes>
		</fileSet>
	</fileSets>
</assembly>
```

### 90.5.2工作流程

工作流程看起来类似于 `Step by step guide to CDC` 中提供的工作流程.唯一的区别是生产环境者不再拥有Contract.因此，消费者和生产环境者必须在公共存储库中处理常见Contract.

### 90.5.3消费者

当 **consumer** 希望脱机处理Contract时，消费者团队不是克隆生产环境者代码，而是克隆公共存储库，转到所需的生产环境者文件夹（例如 `com/example/server` ）并运行 `mvn clean install -DskipTests` 以在本地安装从Contract转换的存根.

> 你需要[Maven installed locally](https://maven.apache.org/download.cgi)

### 90.5.4制片人

作为 **producer** ，它足以改变Spring Cloud Contract Verifier以提供包含Contract的JAR的URL和依赖关系：

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<configuration>
		<contractsMode>REMOTE</contractsMode>
		<contractsRepositoryUrl>http://link/to/your/nexus/or/artifactory/or/sth</contractsRepositoryUrl>
		<contractDependency>
			<groupId>com.example.standalone</groupId>
			<artifactId>contracts</artifactId>
		</contractDependency>
	</configuration>
</plugin>
```

通过此设置，将从 `http://link/to/your/nexus/or/artifactory/or/sth` 下载具有groupid  `com.example.standalone` 和artifactid  `contracts` 的JAR.然后将其解压缩到本地临时文件夹中， `com/example/server` 下的Contract将被选为用于生成测试和存根的Contract.由于这种约定，生产环境者团队将知道在完成一些不兼容的更改时哪些消费者团队将被破坏.

流程的其余部分看起来一样.

### 90.5.5如何针对每个生产环境者定义每个主题的消息传递Contract？

为了避免公共仓库中的消息传递Contract重复，当很少有生产环境者将消息写入一个主题时，我们可以创建结构，其余的Contract将放置在每个生产环境者的文件夹中，每个主题的文件夹中包含消息Contract.

#### 对于Maven项目

为了能够在生产环境者方面工作，我们可以做以下事情（所有这些都通过Maven插件）：

- 为您的类路径添加常见的repo依赖项：

```xml
<dependency>
<groupId>com.example</groupId>
<artifactId>common-repo</artifactId>
<version>${common-repo.version}</version>
</dependency>
```

- D与Contract一起下载JAR并将JAR解压缩到目标：

```xml
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-dependency-plugin</artifactId>
<version>3.0.0</version>
<executions>
<execution>
<id>unpack-dependencies</id>
<phase>process-resources</phase>
<goals>
<goal>unpack</goal>
</goals>
<configuration>
<artifactItems>
<artifactItem>
<groupId>com.example</groupId>
<artifactId>common-repo</artifactId>
<type>jar</type>
<overWrite>false</overWrite>
<outputDirectory>${project.build.directory}/contracts</outputDirectory>
</artifactItem>
</artifactItems>
</configuration>
</execution>
</executions>
</plugin>
```

- 删除我们不感兴趣的所有文件夹：

```xml
<plugin>
<groupId>org.apache.maven.plugins</groupId>
<artifactId>maven-antrun-plugin</artifactId>
<version>1.8</version>
<executions>
<execution>
<phase>process-resources</phase>
<goals>
<goal>run</goal>
</goals>
<configuration>
<tasks>
<delete includeemptydirs="true">
<fileset dir="${project.build.directory}/contracts">
<include name="**/*" />
<!--Producer artifactId-->
<exclude name="**/${project.artifactId}/**" />
<!--List of the supported topics-->
<exclude name="**/${first-topic}/**" />
<exclude name="**/${second-topic}/**" />
</fileset>
</delete>
</tasks>
</configuration>
</execution>
</executions>
</plugin>
```

- 通过指向目标下文件夹的Contract来运行Contract插件：

```xml
<plugin>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<version>${spring-cloud-contract.version}</version>
<extensions>true</extensions>
<configuration>
<packageWithBaseClasses>com.example</packageWithBaseClasses>
<baseClassMappings>
<baseClassMapping>
<contractPackageRegex>.*intoxication.*</contractPackageRegex>
<baseClassFQN>com.example.intoxication.BeerIntoxicationBase</baseClassFQN>
</baseClassMapping>
</baseClassMappings>
<contractsDirectory>${project.build.directory}/contracts</contractsDirectory>
</configuration>
</plugin>
```

#### For Gradle项目

- 为common-repo依赖项添加自定义配置：

```java
ext {
conractsGroupId = "com.example"
contractsArtifactId = "common-repo"
contractsVersion = "1.2.3"
}

configurations {
contracts {
transitive = false
}
}
```

- 将common-repo依赖项添加到类路径：

```java
dependencies {
contracts "${conractsGroupId}:${contractsArtifactId}:${contractsVersion}"
testCompile "${conractsGroupId}:${contractsArtifactId}:${contractsVersion}"
}
```

- D将依赖项下载到适当的文件夹：

```java
task getContracts(type: Copy) {
from configurations.contracts
into new File(project.buildDir, "downloadedContracts")
}
```

- Unzip JAR：

```java
task unzipContracts(type: Copy) {
def zipFile = new File(project.buildDir, "downloadedContracts/${contractsArtifactId}-${contractsVersion}.jar")
def outputDir = file("${buildDir}/unpackedContracts")

from zipTree(zipFile)
into outputDir
}
```

- Cleanup未使用的Contract：

```java
task deleteUnwantedContracts(type: Delete) {
delete fileTree(dir: "${buildDir}/unpackedContracts",
include: "**/*",
excludes: [
"**/${project.name}/**"",
"**/${first-topic}/**",
"**/${second-topic}/**"])
}
```

- 创建任务依赖项：

```java
unzipContracts.dependsOn("getContracts")
deleteUnwantedContracts.dependsOn("unzipContracts")
build.dependsOn("deleteUnwantedContracts")
```

- Configure插件通过使用 `contractsDslDir` 属性指定包含Contract的目录

```java
contracts {
contractsDslDir = new File("${buildDir}/unpackedContracts")
}
```

## 90.6我需要二进制存储吗？我不能用Git吗？

在多语言世界中，有些语言不使用像Artifactory或Nexus这样的二进制存储.从我们提供的Spring Cloud Contract 2.0.0版开始在SCM存储库中存储Contract和存根的机制.目前唯一支持的SCM是Git.

存储库必须进行以下设置（您可以将其签出[here](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/master/contracts_git/)）：

```java
.
└── META-INF
└── com.example
└── beer-api-producer-git
└── 0.0.1-SNAPSHOT
├── contracts
│ └── beer-api-consumer
│     ├── messaging
│     │ ├── shouldSendAcceptedVerification.groovy
│     │ └── shouldSendRejectedVerification.groovy
│     └── rest
│         ├── shouldGrantABeerIfOldEnough.groovy
│         └── shouldRejectABeerIfTooYoung.groovy
└── mappings
└── beer-api-consumer
└── rest
├── shouldGrantABeerIfOldEnough.json
└── shouldRejectABeerIfTooYoung.json
```

在 `META-INF` 文件夹下：

- we通过 `groupId` 分组申请（例如 `com.example` ）

- 然后通过 `artifactId` （例如 `beer-api-producer-git` ）表示每个应用程序

- next，应用程序的版本.该版本是强制性的！ （例如 `0.0.1-SNAPSHOT` ）

- 最后，有两个文件夹：

-  `contracts`   - 最佳做法是将每个消费者所需的Contract存储在具有消费者名称的文件夹中（例如 `beer-api-consumer` ）.这样您就可以使用 `stubs-per-consumer` 功能.进一步的目录结构是任意的

-  `mappings`   - 在此文件夹中，Maven / Gradle Spring Cloud Contract插件将推送存根服务器映射.在消费者方面，Stub Runner将扫描此文件夹以启动存根定义的存根服务器.文件夹结构将是 `contracts` 子文件夹中创建的文件夹的副本.

### 90.6.1议定书公约

为了控制Contract源的类型和位置（无论是二进制存储还是SCM存储库），您可以在存储库的URL中使用该协议. Spring Cloud Contract迭代注册协议解析器并尝试获取Contract（通过插件）或存根（通过Stub Runner）.

对于SCM功能，目前我们支持Git存储库.要使用它，在需要放置存储库URL的属性中，您只需使用 `git://` 作为连接URL的前缀.在这里你可以找到几个例子：

```java
git://file:///foo/bar
git://https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs-contracts-git.git
git://[emailprotected]:spring-cloud-samples/spring-cloud-contract-nodejs-contracts-git.git
```

### 90.6.2制片人

对于生产环境者来说，使用SCM方法，我们可以重用我们用于外部Contract的相同机制.我们通过包含 `git://` 协议的URL来路由Spring Cloud Contract以使用SCM实现.

|图片/ important.png |重要|
| ---- | ---- |
|您必须在Maven中手动添加 `pushStubsToScm` 目标或在Gradle中执行（绑定） `pushStubsToScm` 任务.我们不会将存根推送到您的git存储库的 `origin` 开箱即用. |

**Maven.** 

```xml
<plugin>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<version>${spring-cloud-contract.version}</version>
<extensions>true</extensions>
<configuration>
<!-- Base class mappings etc. -->

<!-- We want to pick contracts from a Git repository -->
<contractsRepositoryUrl>git://https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs-contracts-git.git</contractsRepositoryUrl>

<!-- We reuse the contract dependency section to set up the path
to the folder that contains the contract definitions. In our case the
path will be /groupId/artifactId/version/contracts -->
<contractDependency>
<groupId>${project.groupId}</groupId>
<artifactId>${project.artifactId}</artifactId>
<version>${project.version}</version>
</contractDependency>

<!-- The contracts mode can't be classpath -->
<contractsMode>REMOTE</contractsMode>
</configuration>
<executions>
<execution>
<phase>package</phase>
<goals>
<!-- By default we will not push the stubs back to SCM,
you have to explicitly add it as a goal -->
<goal>pushStubsToScm</goal>
</goals>
</execution>
</executions>
</plugin>
```

**Gradle.** 

```java
contracts {
	// We want to pick contracts from a Git repository
	contractDependency {
		stringNotation = "${project.group}:${project.name}:${project.version}"
	}
	/*
	We reuse the contract dependency section to set up the path
	to the folder that contains the contract definitions. In our case the
	path will be /groupId/artifactId/version/contracts
	 */
	contractRepository {
		repositoryUrl = "git://https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs-contracts-git.git"
	}
	// The mode can't be classpath
	contractsMode = "REMOTE"
	// Base class mappings etc.
}

/*
In this scenario we want to publish stubs to SCM whenever
the `publish` task is executed
*/
publish.dependsOn("publishStubsToScm")
```

有了这样的设置：

- Git项目将被克隆到一个临时目录

-  SCM存根下载器将转到 `META-INF/groupId/artifactId/version/contracts` 文件夹以查找Contract.例如.对于 `com.example:foo:1.0.0` ，路径将是 `META-INF/com.example/foo/1.0.0/contracts` 

- Tests将从Contract中生成

- Stubs将根据Contract创建

- 一旦测试通过，存根将在克隆的存储库中提交

- 最后，将对该回购邮件进行推送 `origin` 

#### 与外部存储库中的生产环境者和存根保持Contract

也可以将Contract保留在生产环境者存储库中，但将存根保留在外部git仓库中.当您想要使用基本使用者 - 生产环境者协作流时，这是最有用的，但是不可能使用工件库来存储存根.

为此，使用常用的生成器设置，然后添加 `pushStubsToScm` 目标并将 `contractsRepositoryUrl` 设置为要保留存根的存储库.

### 90.6.3消费者

在消费者方面传递 `repositoryRoot` 参数（来自 `@AutoConfigureStubRunner` 注释，JUnit规则或属性）时，足以传递SCM存储库的URL，前缀为协议.例如

```java
@AutoConfigureStubRunner(
stubsMode="REMOTE",
repositoryRoot="git://https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs-contracts-git.git",
ids="com.example:bookstore:0.0.1.RELEASE"
)
```

有了这样的设置：

- Git项目将被克隆到一个临时目录

-  SCM存根下载器将转到 `META-INF/groupId/artifactId/version/` 文件夹以查找存根定义和Contract.例如.  `com.example:foo:1.0.0` 的路径是 `META-INF/com.example/foo/1.0.0/` 

- Stub服务器将启动并提供映射
将在消息传递测试中读取和使用
- Messaging定义

## 90.7我可以使用Pact Broker吗？

使用[Pact](http://pact.io/)时，您可以使用[Pact Broker](https://github.com/pact-foundation/pact_broker)来存储和共享Pact定义.从Spring Cloud Contract 2.0.0开始，可以从Pact Broker获取Pact文件以生成测试和存根.

作为先决条件，需要Pact Converter和Pact Stub Downloader.您必须通过 `spring-cloud-contract-pact` 依赖项添加它.您可以在[Section 97.1.1, “Pact Converter”](multi__using_the_pluggable_architecture.html#pact-converter)部分阅读更多相关信息.

|图片/ important.png |重要|
| ---- | ---- |
| Pact遵循消费者Contract约定.这意味着Consumer首先创建Pact定义，然后与Producer共享文件.这些期望来自消费者的代码，如果不满足预期，可能会破坏生产环境者. |

### 90.7.1Contract消费者

消费者使用Pact框架生成Pact文件. Pact文件被发送到Pact Broker.可以找到这样的设置的示例[here](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/master/consumer_pact).

### 90.7.2制片人

对于生产环境者来说，要使用Pact Broker中的Pact文件，我们可以重用我们用于外部Contract的相同机制.我们路由Spring Cloud Contract以通过包含 `pact://` 协议的URL使用Pact实现.将URL传递给Pact Broker就足够了.可以找到这种设置的一个例子[here](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/master/producer_pact).

**Maven.** 

```xml
<plugin>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<version>${spring-cloud-contract.version}</version>
<extensions>true</extensions>
<configuration>
<!-- Base class mappings etc. -->

<!-- We want to pick contracts from a Git repository -->
<contractsRepositoryUrl>pact://http://localhost:8085</contractsRepositoryUrl>

<!-- We reuse the contract dependency section to set up the path
to the folder that contains the contract definitions. In our case the
path will be /groupId/artifactId/version/contracts -->
<contractDependency>
<groupId>${project.groupId}</groupId>
<artifactId>${project.artifactId}</artifactId>
<!-- When + is passed, a latest tag will be applied when fetching pacts -->
<version>+</version>
</contractDependency>

<!-- The contracts mode can't be classpath -->
<contractsMode>REMOTE</contractsMode>
</configuration>
<!-- Don't forget to add spring-cloud-contract-pact to the classpath! -->
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-pact</artifactId>
<version>${spring-cloud-contract.version}</version>
</dependency>
</dependencies>
</plugin>
```

**Gradle.** 

```java
buildscript {
	repositories {
		//...
	}

	dependencies {
		// ...
		// Don't forget to add spring-cloud-contract-pact to the classpath!
		classpath "org.springframework.cloud:spring-cloud-contract-pact:${contractVersion}"
	}
}

contracts {
	// When + is passed, a latest tag will be applied when fetching pacts
	contractDependency {
		stringNotation = "${project.group}:${project.name}:+"
	}
	contractRepository {
		repositoryUrl = "pact://http://localhost:8085"
	}
	// The mode can't be classpath
	contractsMode = "REMOTE"
	// Base class mappings etc.
}
```

有了这样的设置：

- Pact文件将从Pact下载经纪人

- Spring Cloud Contract会将Pact文件转换为测试和存根

- 带有存根的JAR会像往常一样自动创建

### 90.7.3Contract消费者（生产环境者Contract法）

在您不希望采用消费者Contract方法的情况下（针对每个消费者定义期望）但您更愿意做生产环境者Contract（生产环境者提供Contract并发布存根），这足以使用Spring CloudContract使用Stub Runner选项.可以找到这种设置的一个例子[here](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/master/consumer_pact_stubrunner).

首先，请记住将Stub Runner和Spring Cloud Contract Pact模块添加为测试依赖项.

**Maven.** 

```xml
<dependencyManagement>
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-dependencies</artifactId>
<version>${spring-cloud.version}</version>
<type>pom</type>
<scope>import</scope>
</dependency>
</dependencies>
</dependencyManagement>

<!-- Don't forget to add spring-cloud-contract-pact to the classpath! -->
<dependencies>
<!-- ... -->
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
<scope>test</scope>
</dependency>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-pact</artifactId>
<scope>test</scope>
</dependency>
</dependencies>
```

**Gradle.** 

```java
dependencyManagement {
imports {
mavenBom "org.springframework.cloud:spring-cloud-dependencies:${springCloudVersion}"
}
}

dependencies {
//...
testCompile("org.springframework.cloud:spring-cloud-starter-contract-stub-runner")
// Don't forget to add spring-cloud-contract-pact to the classpath!
testCompile("org.springframework.cloud:spring-cloud-contract-pact")
}
```

接下来，只需将Pact Broker的URL传递给 `repositoryRoot` ，前缀为 `pact://`  protocol.例如.  `pact://http://localhost:8085` 

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureStubRunner(stubsMode = StubRunnerProperties.StubsMode.REMOTE,
		ids = "com.example:beer-api-producer-pact",
		repositoryRoot = "pact://http://localhost:8085")
public class BeerControllerTest {
//Inject the port of the running stub
@StubRunnerPort("beer-api-producer-pact") int producerPort;
//...
}
```

有了这样的设置：

- Pact文件将从Pact Broker下载

- Spring Cloud Contract会将Pact文件转换为存根定义

- 将启动存根服务器并使用存根提供

有关Pact支持的更多信息，您可以转到[Section 97.7, “Using the Pact Stub Downloader”](multi__using_the_pluggable_architecture.html#pact-stub-downloader)部分.

## 90.8如何调试生成的测试客户端发送的请求/响应？

生成的测试都以某种形式或方式归结为RestAssured，它依赖于[Apache HttpClient](https://hc.apache.org/httpcomponents-client-ga/). HttpClient有一个名为[wire logging](https://hc.apache.org/httpcomponents-client-ga/logging.html#Wire_Logging)的工具，它将整个请求和响应记录到HttpClient. Spring Boot有一个日志记录[common application property](https://docs.spring.io/spring-boot/docs/current/reference/html/common-application-properties.html)用于执行此类操作，只需将其添加到您的应用程序属性即可

```java
logging.level.org.apache.http.wire=DEBUG
```

### 90.8.1如何调试WireMock发送的映射/请求/响应？

从版本 `1.2.0` 开始，我们将WireMock日志记录打开为info，将WireMock通知程序设置为详细.现在，您将准确了解WireMock服务器收到的请求以及选择了哪个匹配的响应定义.

要关闭此功能，只需将WireMock日志记录到 `ERROR` 

```java
logging.level.com.github.tomakehurst.wiremock=ERROR
```

### 90.8.2如何查看在HTTP服务器存根中注册的内容？

您可以使用 `@AutoConfigureStubRunner` 或 `StubRunnerRule` 上的 `mappingsOutputFolder` 属性转储每个工件ID的所有映射.此外，还将附加启动给定存根服务器的端口.

### 90.8.3我可以从文件中引用文字吗？

是!在版本1.2.0中，我们添加了这种可能性.在DSL中调用 `file(…)` 方法并提供相对于Contract所在位置的路径就足够了.如果您正在使用YAML，请使用 `bodyFromFile` 属性.


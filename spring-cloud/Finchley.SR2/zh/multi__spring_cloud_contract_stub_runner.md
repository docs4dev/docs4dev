## 93. Spring Cloud Contract Stub Runner

使用Spring Cloud Contract Verifier时可能遇到的一个问题是将生成的WireMock JSON存根从服务器端传递到客户端（或传递给各种客户端）.在客户端生成消息传递方面也是如此.

复制JSON文件并手动设置客户端以进行消息传递是不可能的.这就是我们推出Spring Cloud Contract Stub Runner的原因.它可以自动为您下载和运行存根.

## 93.1快照版本

将其他快照存储库添加到 `build.gradle` 文件以使用快照版本，这些版本会在每次成功构建后自动上载：

**Maven.** 

```xml
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
```

**Gradle.** 

```java
buildscript {
	repositories {
		mavenCentral()
		mavenLocal()
		maven { url "http://repo.spring.io/snapshot" }
		maven { url "http://repo.spring.io/milestone" }
		maven { url "http://repo.spring.io/release" }
	}
```

## 93.2将Stubs发布为JAR

最简单的方法是集中存根的方式.例如，您可以将它们保存为Maven存储库中的jar.

> 对于Maven和Gradle，设置准备就绪.但是，您可以根据需要自定义它.

**Maven.** 

```xml
<!-- First disable the default jar setup in the properties section -->
<!-- we don't want the verifier to do a jar for us -->
<spring.cloud.contract.verifier.skip>true</spring.cloud.contract.verifier.skip>

<!-- Next add the assembly plugin to your build -->
<!-- we want the assembly plugin to generate the JAR -->
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
				<descriptors>
					$../../../../src/assembly/stub.xml
				</descriptors>
			</configuration>
		</execution>
	</executions>
</plugin>

<!-- Finally setup your assembly. Below you can find the contents of src/main/assembly/stub.xml -->
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
			<directory>src/main/java</directory>
			<outputDirectory>/</outputDirectory>
			<includes>
				<include>**com/example/model/*.*</include>
			</includes>
		</fileSet>
		<fileSet>
			<directory>${project.build.directory}/classes</directory>
			<outputDirectory>/</outputDirectory>
			<includes>
				<include>**com/example/model/*.*</include>
			</includes>
		</fileSet>
		<fileSet>
			<directory>${project.build.directory}/snippets/stubs</directory>
			<outputDirectory>META-INF/${project.groupId}/${project.artifactId}/${project.version}/mappings</outputDirectory>
			<includes>
				<include>**/*</include>
			</includes>
		</fileSet>
		<fileSet>
			<directory>$../../../../src/test/resources/contracts</directory>
			<outputDirectory>META-INF/${project.groupId}/${project.artifactId}/${project.version}/contracts</outputDirectory>
			<includes>
				<include>**/*.groovy</include>
			</includes>
		</fileSet>
	</fileSets>
</assembly>
```

**Gradle.** 

```java
ext {
	contractsDir = file("mappings")
	stubsOutputDirRoot = file("${project.buildDir}/production/${project.name}-stubs/")
}

// Automatically added by plugin:
// copyContracts - copies contracts to the output folder from which JAR will be created
// verifierStubsJar - JAR with a provided stub suffix
// the presented publication is also added by the plugin but you can modify it as you wish

publishing {
	publications {
		stubs(MavenPublication) {
			artifactId "${project.name}-stubs"
			artifact verifierStubsJar
		}
	}
}
```

## 93.3 Stub Runner Core

运行服务协作者的存根.将存根作为服务Contract处理允许使用存根运行器作为[Consumer Driven Contracts](http://martinfowler.com/articles/consumerDrivenContracts.html)的实现.

Stub Runner允许您自动下载所提供的依赖项的存根（或从类路径中选择那些存根），为它们启动WireMock服务器并使用适当的存根定义提供它们.对于消息传递，定义了特殊的存根路由.

### 93.3.1正在检索存根

您可以选择以下获取存根的选项

- Aether based解决方案，从Artifactory / Nexus下载带有存根的JAR

- Classpath扫描解决方案，通过模式搜索类路径以检索存根

- 编写自己的 `org.springframework.cloud.contract.stubrunner.StubDownloaderBuilder` 实现以进行完全自定义

后一个例子在[Custom Stub Runner]()部分中描述.

#### Stub下载

您可以通过 `stubsMode` 开关控制存根下载.它从 `StubRunnerProperties.StubsMode` 枚举中选择值.您可以使用以下选项

-  `StubRunnerProperties.StubsMode.CLASSPATH` （默认值） - 将从类路径中选取存根

-  `StubRunnerProperties.StubsMode.LOCAL`   - 将从本地存储中挑选存根（例如 `.m2` ）

-  `StubRunnerProperties.StubsMode.REMOTE`   - 将从远程位置挑选存根

例：

```java
@AutoConfigureStubRunner(repositoryRoot="http://foo.bar", ids = "com.example:beer-api-producer:+:stubs:8095", stubsMode = StubRunnerProperties.StubsMode.LOCAL)
```

#### Classpath扫描

如果将 `stubsMode` 属性设置为 `StubRunnerProperties.StubsMode.CLASSPATH` （或者在 `CLASSPATH` 为默认值时未设置任何内容），则会扫描类路径.我们来看下面的例子：

```java
@AutoConfigureStubRunner(ids = {
"com.example:beer-api-producer:+:stubs:8095",
"com.example.foo:bar:1.0.0:superstubs:8096"
})
```

如果已将依赖项添加到类路径中

**Maven.** 

```xml
<dependency>
<groupId>com.example</groupId>
<artifactId>beer-api-producer-restdocs</artifactId>
<classifier>stubs</classifier>
<version>0.0.1-SNAPSHOT</version>
<scope>test</scope>
<exclusions>
<exclusion>
<groupId>*</groupId>
<artifactId>*</artifactId>
</exclusion>
</exclusions>
</dependency>
<dependency>
<groupId>com.example.foo</groupId>
<artifactId>bar</artifactId>
<classifier>superstubs</classifier>
<version>1.0.0</version>
<scope>test</scope>
<exclusions>
<exclusion>
<groupId>*</groupId>
<artifactId>*</artifactId>
</exclusion>
</exclusions>
</dependency>
```

**Gradle.** 

```java
testCompile("com.example:beer-api-producer-restdocs:0.0.1-SNAPSHOT:stubs") {
transitive = false
}
testCompile("com.example.foo:bar:1.0.0:superstubs") {
transitive = false
}
```

然后将扫描类路径中的以下位置.对于 `com.example:beer-api-producer-restdocs` 

-  / META-INF / com.示例/啤酒-API的生产环境者restdocs /  ***/**  *.

-  /Contract/ com.示例/啤酒-API的生产环境者restdocs /  ***/**  *.

-  /映射/ com.示例/啤酒-API的生产环境者restdocs /  ***/**  *.

和 `com.example.foo:bar` 

-  / META-INF / com.example.foo /酒吧/  ***/**  *.

-  /Contract/ com.example.foo /酒吧/  ***/**  *.

-  /映射/ com.example.foo /酒吧/  ***/**  *.

> 如您所见，在打包生产环境者存根时必须明确提供组和工件ID.

制片人会的像这样设置Contract：

```java
└── src
└── test
└── resources
└── contracts
└── com.example
└── beer-api-producer-restdocs
└── nested
└── contract3.groovy
```

实现适当的短截包装.

或者使用[Maven assembly plugin](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/blob/2.0.x/producer_with_restdocs/pom.xml)或[Gradle Jar](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/blob/2.0.x/producer_with_restdocs/build.gradle)任务，您必须在存根jar中创建以下结构.

```java
└── META-INF
└── com.example
└── beer-api-producer-restdocs
└── 2.0.0
├── contracts
│ └── nested
│       └── contract2.groovy
└── mappings
└── mapping.json
```

通过维护此结构，可以扫描类路径，您可以从消息传递/ HTTP存根中获益，而无需下载工件.

### 93.3.2正在运行的存根

#### 使用主应用程序运行

您可以为主类设置以下选项：

```java
-c, --classifier                Suffix for the jar containing stubs (e.
g. 'stubs' if the stub jar would
have a 'stubs' classifier for stubs:
foobar-stubs ). Defaults to 'stubs'
(default: stubs)
--maxPort, --maxp <Integer>     Maximum port value to be assigned to
the WireMock instance. Defaults to
15000 (default: 15000)
--minPort, --minp <Integer>     Minimum port value to be assigned to
the WireMock instance. Defaults to
10000 (default: 10000)
-p, --password                  Password to user when connecting to
repository
--phost, --proxyHost            Proxy host to use for repository
requests
--pport, --proxyPort [Integer]  Proxy port to use for repository
requests
-r, --root                      Location of a Jar containing server
where you keep your stubs (e.g. http:
//nexus.
net/content/repositories/repository)
-s, --stubs                     Comma separated list of Ivy
representation of jars with stubs.
Eg. groupid:artifactid1,groupid2:
artifactid2:classifier
--sm, --stubsMode               Stubs mode to be used. Acceptable values
[CLASSPATH, LOCAL, REMOTE]
-u, --username                  Username to user when connecting to
repository
```

#### HTTP存根

存根在JSON文档中定义，其语法在[WireMock documentation](http://wiremock.org/stubbing.html)中定义

例：

```java
{
"request": {
"method": "GET",
"url": "/ping"
},
"response": {
"status": 200,
"body": "pong",
"headers": {
"Content-Type": "text/plain"
}
}
}
```

#### 查看已注册的映射

每个存根协作者都会在 `__/admin/` endpoints下公开已定义映射的列表.

您还可以使用 `mappingsOutputFolder` 属性将映射转储到文件.对于基于注释的方法，它看起来像这样

```java
@AutoConfigureStubRunner(ids="a.b.c:loanIssuance,a.b.c:fraudDetectionServer",
mappingsOutputFolder = "target/outputmappings/")
```

对于像这样的JUnit方法：

```java
@ClassRule @Shared StubRunnerRule rule = new StubRunnerRule()
			.repoRoot("http://some_url")
			.downloadStub("a.b.c", "loanIssuance")
			.downloadStub("a.b.c:fraudDetectionServer")
			.withMappingsOutputFolder("target/outputmappings")
```

然后，如果您签出文件夹 `target/outputmappings` ，您将看到以下结构

```java
.
├── fraudDetectionServer_13705
└── loanIssuance_12255
```

这意味着有两个存根注册.  `fraudDetectionServer` 在端口 `13705` 注册， `loanIssuance` 在端口 `12255` 注册.如果我们看一下我们将看到的一个文件（对于WireMock）可用于给定服务器的映射：

```java
[{
"id" : "f9152eb9-bf77-4c38-8289-90be7d10d0d7",
"request" : {
"url" : "/name",
"method" : "GET"
},
"response" : {
"status" : 200,
"body" : "fraudDetectionServer"
},
"uuid" : "f9152eb9-bf77-4c38-8289-90be7d10d0d7"
},
...
]
```

#### Messaging Stubs

根据提供的Stub Runner依赖关系和DSL，将自动设置消息传递路由.

## 93.4 Stub Runner JUnit规则

Stub Runner附带了一个JUnit规则，因此您可以非常轻松地下载和运行给定组和工件ID的存根：

```java
@ClassRule public static StubRunnerRule rule = new StubRunnerRule()
		.repoRoot(repoRoot())
		.downloadStub("org.springframework.cloud.contract.verifier.stubs", "loanIssuance")
		.downloadStub("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer");
```

执行该规则后，Stub Runner将连接到您的Maven存储库，并且对于给定的依赖项列表，尝试：

- download他们

- cache他们在本地

- unzip他们到一个临时文件夹

- 启动一个WireMock服务器，用于从提供的端口/提供端口范围内的随机端口上的每个Maven依赖项

- 给WireMock服务器提供了有效的WireMock定义的所有JSON文件

- can也可以发送消息（记得传递 `MessageVerifier` 接口的实现）

Stub Runner使用[Eclipse Aether](https://wiki.eclipse.org/Aether)机制下载Maven依赖项.查看他们的[docs](https://wiki.eclipse.org/Aether)了解更多信息.

由于 `StubRunnerRule` 实现了 `StubFinder` ，它允许您查找已启动的存根：

```java
package org.springframework.cloud.contract.stubrunner;

import java.net.URL;
import java.util.Collection;
import java.util.Map;

import org.springframework.cloud.contract.spec.Contract;

public interface StubFinder extends StubTrigger {
	/**
	 * For the given groupId and artifactId tries to find the matching
	 * URL of the running stub.
	 *
	 * @param groupId - might be null. In that case a search only via artifactId takes place
	 * @return URL of a running stub or throws exception if not found
	 */
	URL findStubUrl(String groupId, String artifactId) throws StubNotFoundException;

	/**
	 * For the given Ivy notation {@code [groupId]:artifactId:[version]:[classifier]} tries to
	 * find the matching URL of the running stub. You can also pass only {@code artifactId}.
	 *
	 * @param ivyNotation - Ivy representation of the Maven artifact
	 * @return URL of a running stub or throws exception if not found
	 */
	URL findStubUrl(String ivyNotation) throws StubNotFoundException;

	/**
	 * Returns all running stubs
	 */
	RunningStubs findAllRunningStubs();

	/**
	 * Returns the list of Contracts
	 */
	Map<StubConfiguration, Collection<Contract>> getContracts();
}
```

Spock测试中的使用示例：

```java
@ClassRule @Shared StubRunnerRule rule = new StubRunnerRule()
		.stubsMode(StubRunnerProperties.StubsMode.REMOTE)
		.repoRoot(StubRunnerRuleSpec.getResource("/m2repo/repository").toURI().toString())
		.downloadStub("org.springframework.cloud.contract.verifier.stubs", "loanIssuance")
		.downloadStub("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer")
		.withMappingsOutputFolder("target/outputmappingsforrule")

def 'should start WireMock servers'() {
	expect: 'WireMocks are running'
		rule.findStubUrl('org.springframework.cloud.contract.verifier.stubs', 'loanIssuance') != null
		rule.findStubUrl('loanIssuance') != null
		rule.findStubUrl('loanIssuance') == rule.findStubUrl('org.springframework.cloud.contract.verifier.stubs', 'loanIssuance')
		rule.findStubUrl('org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer') != null
	and:
		rule.findAllRunningStubs().isPresent('loanIssuance')
		rule.findAllRunningStubs().isPresent('org.springframework.cloud.contract.verifier.stubs', 'fraudDetectionServer')
		rule.findAllRunningStubs().isPresent('org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer')
	and: 'Stubs were registered'
		"${rule.findStubUrl('loanIssuance').toString()}/name".toURL().text == 'loanIssuance'
		"${rule.findStubUrl('fraudDetectionServer').toString()}/name".toURL().text == 'fraudDetectionServer'
}

def 'should output mappings to output folder'() {
	when:
		def url = rule.findStubUrl('fraudDetectionServer')
	then:
		new File("target/outputmappingsforrule", "fraudDetectionServer_${url.port}").exists()
}
```

JUnit测试中的用法示例：

```java
@Test
public void should_start_wiremock_servers() throws Exception {
	// expect: 'WireMocks are running'
		then(rule.findStubUrl("org.springframework.cloud.contract.verifier.stubs", "loanIssuance")).isNotNull();
		then(rule.findStubUrl("loanIssuance")).isNotNull();
		then(rule.findStubUrl("loanIssuance")).isEqualTo(rule.findStubUrl("org.springframework.cloud.contract.verifier.stubs", "loanIssuance"));
		then(rule.findStubUrl("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer")).isNotNull();
	// and:
		then(rule.findAllRunningStubs().isPresent("loanIssuance")).isTrue();
		then(rule.findAllRunningStubs().isPresent("org.springframework.cloud.contract.verifier.stubs", "fraudDetectionServer")).isTrue();
		then(rule.findAllRunningStubs().isPresent("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer")).isTrue();
	// and: 'Stubs were registered'
		then(httpGet(rule.findStubUrl("loanIssuance").toString() + "/name")).isEqualTo("loanIssuance");
		then(httpGet(rule.findStubUrl("fraudDetectionServer").toString() + "/name")).isEqualTo("fraudDetectionServer");
}
```

有关如何应用Stub Runner全局配置的更多信息，请查看 **Common properties for JUnit and Spring** .

|图片/ important.png |重要|
| ---- | ---- |
|要将JUnit规则与消息传递一起使用，必须为规则构建器提供 `MessageVerifier` 接口的实现（例如 `rule.messageVerifier(new MyMessageVerifier())` ）.如果您不这样做，那么每当您尝试发送消息时，都会抛出异常. |

### 93.4.1 Maven设置

存根下载程序为不同的本地存储库文件夹提供Maven设置.目前未考虑存储库和配置文件的身份验证详细信息，因此您需要使用上述属性指定它.

### 93.4.2提供固定端口

您还可以在固定端口上运行存根.你可以用两种不同的方式做到这一点.一种是通过属性传递它，另一种是通过JUnit规则的流畅API.

### 93.4.3流利的API

使用 `StubRunnerRule` 时，您可以添加要下载的存根，然后传递最后下载的存根的端口.

```java
@ClassRule public static StubRunnerRule rule = new StubRunnerRule()
		.repoRoot(repoRoot())
		.downloadStub("org.springframework.cloud.contract.verifier.stubs", "loanIssuance")
		.withPort(12345)
		.downloadStub("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer:12346");
```

您可以看到，对于此示例，以下测试有效：

```java
then(rule.findStubUrl("loanIssuance")).isEqualTo(URI.create("http://localhost:12345").toURL());
then(rule.findStubUrl("fraudDetectionServer")).isEqualTo(URI.create("http://localhost:12346").toURL());
```

### 93.4.4 Stub Runner with Spring

设置Stub Runner项目的Spring配置.

通过在配置文件中提供存根列表，Stub Runner自动下载并在WireMock中注册选定的存根.

如果要查找存根依赖关系的URL，可以自动装配 `StubFinder` 接口并使用其方法，如下所示：

```java
@ContextConfiguration(classes = Config, loader = SpringBootContextLoader)
@SpringBootTest(properties = [" stubrunner.cloud.enabled=false",
		'foo=${stubrunner.runningstubs.fraudDetectionServer.port}',
		'fooWithGroup=${stubrunner.runningstubs.org.springframework.cloud.contract.verifier.stubs.fraudDetectionServer.port}'])
@AutoConfigureStubRunner(mappingsOutputFolder = "target/outputmappings/")
@ActiveProfiles("test")
class StubRunnerConfigurationSpec extends Specification {

	@Autowired StubFinder stubFinder
	@Autowired Environment environment
	@StubRunnerPort("fraudDetectionServer") int fraudDetectionServerPort
	@StubRunnerPort("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer") int fraudDetectionServerPortWithGroupId
	@Value('${foo}') Integer foo

	@BeforeClass
	@AfterClass
	void setupProps() {
		System.clearProperty("stubrunner.repository.root")
		System.clearProperty("stubrunner.classifier")
	}

	def 'should start WireMock servers'() {
		expect: 'WireMocks are running'
			stubFinder.findStubUrl('org.springframework.cloud.contract.verifier.stubs', 'loanIssuance') != null
			stubFinder.findStubUrl('loanIssuance') != null
			stubFinder.findStubUrl('loanIssuance') == stubFinder.findStubUrl('org.springframework.cloud.contract.verifier.stubs', 'loanIssuance')
			stubFinder.findStubUrl('loanIssuance') == stubFinder.findStubUrl('org.springframework.cloud.contract.verifier.stubs:loanIssuance')
			stubFinder.findStubUrl('org.springframework.cloud.contract.verifier.stubs:loanIssuance:0.0.1-SNAPSHOT') == stubFinder.findStubUrl('org.springframework.cloud.contract.verifier.stubs:loanIssuance:0.0.1-SNAPSHOT:stubs')
			stubFinder.findStubUrl('org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer') != null
		and:
			stubFinder.findAllRunningStubs().isPresent('loanIssuance')
			stubFinder.findAllRunningStubs().isPresent('org.springframework.cloud.contract.verifier.stubs', 'fraudDetectionServer')
			stubFinder.findAllRunningStubs().isPresent('org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer')
		and: 'Stubs were registered'
			"${stubFinder.findStubUrl('loanIssuance').toString()}/name".toURL().text == 'loanIssuance'
			"${stubFinder.findStubUrl('fraudDetectionServer').toString()}/name".toURL().text == 'fraudDetectionServer'
	}

	def 'should throw an exception when stub is not found'() {
		when:
			stubFinder.findStubUrl('nonExistingService')
		then:
			thrown(StubNotFoundException)
		when:
			stubFinder.findStubUrl('nonExistingGroupId', 'nonExistingArtifactId')
		then:
			thrown(StubNotFoundException)
	}

	def 'should register started servers as environment variables'() {
		expect:
			environment.getProperty("stubrunner.runningstubs.loanIssuance.port") != null
			stubFinder.findAllRunningStubs().getPort("loanIssuance") == (environment.getProperty("stubrunner.runningstubs.loanIssuance.port") as Integer)
		and:
			environment.getProperty("stubrunner.runningstubs.fraudDetectionServer.port") != null
			stubFinder.findAllRunningStubs().getPort("fraudDetectionServer") == (environment.getProperty("stubrunner.runningstubs.fraudDetectionServer.port") as Integer)
		and:
			environment.getProperty("stubrunner.runningstubs.fraudDetectionServer.port") != null
			stubFinder.findAllRunningStubs().getPort("fraudDetectionServer") == (environment.getProperty("stubrunner.runningstubs.org.springframework.cloud.contract.verifier.stubs.fraudDetectionServer.port") as Integer)
	}

	def 'should be able to interpolate a running stub in the passed test property'() {
		given:
			int fraudPort = stubFinder.findAllRunningStubs().getPort("fraudDetectionServer")
		expect:
			fraudPort > 0
			environment.getProperty("foo", Integer) == fraudPort
			environment.getProperty("fooWithGroup", Integer) == fraudPort
			foo == fraudPort
	}

	@Issue("#573")
	def 'should be able to retrieve the port of a running stub via an annotation'() {
		given:
			int fraudPort = stubFinder.findAllRunningStubs().getPort("fraudDetectionServer")
		expect:
			fraudPort > 0
			fraudDetectionServerPort == fraudPort
			fraudDetectionServerPortWithGroupId == fraudPort
	}

	def 'should dump all mappings to a file'() {
		when:
			def url = stubFinder.findStubUrl("fraudDetectionServer")
		then:
			new File("target/outputmappings/", "fraudDetectionServer_${url.port}").exists()
	}

	@Configuration
	@EnableAutoConfiguration
	static class Config {}
}
```

对于以下配置文件：

```java
stubrunner:
repositoryRoot: classpath:m2repo/repository/
ids:
- org.springframework.cloud.contract.verifier.stubs:loanIssuance
- org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer
- org.springframework.cloud.contract.verifier.stubs:bootService
stubs-mode: remote
```

您也可以使用 `@AutoConfigureStubRunner` 中的属性，而不是使用属性.您可以在下面找到通过在注释上设置值来获得相同结果的示例.

```java
@AutoConfigureStubRunner(
		ids = ["org.springframework.cloud.contract.verifier.stubs:loanIssuance",
		"org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer",
		"org.springframework.cloud.contract.verifier.stubs:bootService"],
		stubsMode = StubRunnerProperties.StubsMode.REMOTE,
		repositoryRoot = "classpath:m2repo/repository/")
```

Stub Runner Spring为每个注册的WireMock服务器按以下方式注册环境变量. Stub Runner ID的示例 `com.example:foo` ， `com.example:bar` .

-  `stubrunner.runningstubs.foo.port` 

-  `stubrunner.runningstubs.com.example.foo.port` 

-  `stubrunner.runningstubs.bar.port` 

-  `stubrunner.runningstubs.com.example.bar.port` 

您可以在代码中引用它.

您还可以使用 `@StubRunnerPort` 注释注入正在运行的存根的端口.注释的值可以是 `groupid:artifactid` 或只是 `artifactid` . Stub Runner ID的示例 `com.example:foo` ， `com.example:bar` .

```java
@StubRunnerPort("foo")
int fooPort;
@StubRunnerPort("com.example:bar")
int barPort;
```

## 93.5 Stub Runner Spring Cloud

Stub Runner可以与Spring Cloud集成.

对于现实生活中的例子，你可以查看

- [producer app sample](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/2.0.x/producer)

- [consumer app sample](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/2.0.x/consumer_with_discovery)

### 93.5.1 Stubbing Service Discovery

`Stub Runner Spring Cloud` 最重要的特征是它的存在

-  `DiscoveryClient` 

-  `Ribbon`   `ServerList` 

这意味着无论您使用的是Zookeeper，Consul，Eureka还是其他任何东西，您都不需要在测试中使用它.我们正在启动依赖项的WireMock实例，并且只要您使用 `Feign` ，负载均衡 `RestTemplate` 或我们告诉您的应用程序 `DiscoveryClient` 直接调用那些存根服务器而不是调用真正的服务发现工具.

例如，此测试将通过

```java
def 'should make service discovery work'() {
	expect: 'WireMocks are running'
		"${stubFinder.findStubUrl('loanIssuance').toString()}/name".toURL().text == 'loanIssuance'
		"${stubFinder.findStubUrl('fraudDetectionServer').toString()}/name".toURL().text == 'fraudDetectionServer'
	and: 'Stubs can be reached via load service discovery'
		restTemplate.getForObject('http://loanIssuance/name', String) == 'loanIssuance'
		restTemplate.getForObject('http://someNameThatShouldMapFraudDetectionServer/name', String) == 'fraudDetectionServer'
}
```

对于以下配置文件

```java
stubrunner:
idsToServiceIds:
ivyNotation: someValueInsideYourCode
fraudDetectionServer: someNameThatShouldMapFraudDetectionServer
```

#### Test配置文件和服务发现

在集成测试中，您通常不希望既不调用发现服务（例如Eureka）也不调用Config Server.这就是您创建要禁用这些功能的其他测试配置的原因.

由于[spring-cloud-commons](https://github.com/spring-cloud/spring-cloud-commons/issues/156)的某些限制要实现此目的，您可以通过如下所示的静态块禁用这些属性（Eureka的示例）

```java
//Hack to work around https://github.com/spring-cloud/spring-cloud-commons/issues/156
static {
System.setProperty("eureka.client.enabled", "false");
System.setProperty("spring.cloud.config.failFast", "false");
}
```

### 93.5.2其他配置

您可以使用 `stubrunner.idsToServiceIds:` 映射将存根的artifactId与应用程序的名称进行匹配.您可以通过以下方式禁用Stub Runner功能区支持： `stubrunner.cloud.ribbon.enabled` 等于 `false` 您可以通过提供以下内容来禁用Stub Runner支持： `stubrunner.cloud.enabled` 等于 `false` 

> By默认所有服务发现都将被存根.这意味着，无论您是否拥有现有的 `DiscoveryClient` ，其结果都将被忽略.但是，如果要重复使用它，只需将 `stubrunner.cloud.delegate.enabled` 设置为 `true` ，然后将现有的 `DiscoveryClient` 结果与存根的结果合并.

Stub Runner使用的默认Maven配置可以通过以下系统属性或环境变量进行调整

-  `maven.repo.local`   - 自定义maven本地存储库位置的路径

-  `org.apache.maven.user-settings`   - 自定义maven用户设置位置的路径

-  `org.apache.maven.global-settings`   -  maven全局设置位置的路径

## 93.6 Stub Runner Boot应用程序

Spring Cloud Contract Stub Runner Boot是一个Spring Boot应用程序，它公开RESTendpoints以触发消息标签并访问已启动的WireMock服务器.

其中一个用例是在已部署的应用程序上运行一些冒烟（端到端）测试.您可以查看[Spring Cloud Pipelines](https://github.com/spring-cloud/spring-cloud-pipelines)项目以获取更多信息.

### 93.6.1如何使用它？

#### Stub Runner Server

只需添加

```java
compile "org.springframework.cloud:spring-cloud-starter-stub-runner"
```

使用 `@EnableStubRunnerServer` 注释一个类，构建一个胖jar，你准备好了！

对于属性，请检查 **Stub Runner Spring** 部分.

#### Stub Runner Server Fat Jar

您可以从Maven下载独立的JAR（例如，版本1.2.3.RELEASE），如下所示：

```java
$ wget -O stub-runner.jar 'https://search.maven.org/remote_content?g=org.springframework.cloud&a=spring-cloud-contract-stub-runner-boot&v=1.2.3.RELEASE'
$ java -jar stub-runner.jar --stubrunner.ids=... --stubrunner.repositoryRoot=...
```

#### Spring Cloud CLI

从 `1.4.0.RELEASE` 项目的 `1.4.0.RELEASE` 版本开始，您可以通过执行 `spring cloud stubrunner` 来启动Stub Runner Boot.

要传递配置，只需在当前工作目录或名为 `config` 或 `~/.spring-cloud` 的子目录中创建 `stubrunner.yml` 文件.该文件可能如下所示（本地安装运行存根的示例）

**stubrunner.yml.** 

```java
stubrunner:
stubsMode: LOCAL
ids:
- com.example:beer-api-producer:+:9876
```

然后从终端窗口调用 `spring cloud stubrunner` 以启动Stub Runner服务器.它将在 `8750` 端口提供.

### 93.6.2endpoints

#### HTTP

- GET  `/stubs`   - 以 `ivy:integer` 表示法返回所有正在运行的存根的列表

- GET  `/stubs/{ivy}`   - 返回给定 `ivy` 表示法的端口（当调用endpoints `ivy` 时也可以只 `artifactId` ）

#### Messaging

对于消息传递

- GET  `/triggers`   - 以 `ivy : [ label1, label2 …]` 表示法返回所有正在运行的标签的列表

- POST  `/triggers/{label}`   - 使用 `label` 执行触发器

- POST  `/triggers/{ivy}/{label}`   - 对于给定的 `ivy` 表示法执行 `label` 的触发器（当调用endpoints `ivy` 时也可以只 `artifactId` ）

### 93.6.3示例

```java
@ContextConfiguration(classes = StubRunnerBoot, loader = SpringBootContextLoader)
@SpringBootTest(properties = "spring.cloud.zookeeper.enabled=false")
@ActiveProfiles("test")
class StubRunnerBootSpec extends Specification {

	@Autowired StubRunning stubRunning

	def setup() {
		RestAssuredMockMvc.standaloneSetup(new HttpStubsController(stubRunning),
				new TriggerController(stubRunning))
	}

	def 'should return a list of running stub servers in "full ivy:port" notation'() {
		when:
			String response = RestAssuredMockMvc.get('/stubs').body.asString()
		then:
			def root = new JsonSlurper().parseText(response)
			root.'org.springframework.cloud.contract.verifier.stubs:bootService:0.0.1-SNAPSHOT:stubs' instanceof Integer
	}

	def 'should return a port on which a [#stubId] stub is running'() {
		when:
			def response = RestAssuredMockMvc.get("/stubs/${stubId}")
		then:
			response.statusCode == 200
			Integer.valueOf(response.body.asString()) > 0
		where:
			stubId << ['org.springframework.cloud.contract.verifier.stubs:bootService:+:stubs',
					   'org.springframework.cloud.contract.verifier.stubs:bootService:0.0.1-SNAPSHOT:stubs',
					   'org.springframework.cloud.contract.verifier.stubs:bootService:+',
					   'org.springframework.cloud.contract.verifier.stubs:bootService',
					   'bootService']
	}

	def 'should return 404 when missing stub was called'() {
		when:
			def response = RestAssuredMockMvc.get("/stubs/a:b:c:d")
		then:
			response.statusCode == 404
	}

	def 'should return a list of messaging labels that can be triggered when version and classifier are passed'() {
		when:
			String response = RestAssuredMockMvc.get('/triggers').body.asString()
		then:
			def root = new JsonSlurper().parseText(response)
			root.'org.springframework.cloud.contract.verifier.stubs:bootService:0.0.1-SNAPSHOT:stubs'?.containsAll(["delete_book","return_book_1","return_book_2"])
	}

	def 'should trigger a messaging label'() {
		given:
			StubRunning stubRunning = Mock()
			RestAssuredMockMvc.standaloneSetup(new HttpStubsController(stubRunning), new TriggerController(stubRunning))
		when:
			def response = RestAssuredMockMvc.post("/triggers/delete_book")
		then:
			response.statusCode == 200
		and:
			1 * stubRunning.trigger('delete_book')
	}

	def 'should trigger a messaging label for a stub with [#stubId] ivy notation'() {
		given:
			StubRunning stubRunning = Mock()
			RestAssuredMockMvc.standaloneSetup(new HttpStubsController(stubRunning), new TriggerController(stubRunning))
		when:
			def response = RestAssuredMockMvc.post("/triggers/$stubId/delete_book")
		then:
			response.statusCode == 200
		and:
			1 * stubRunning.trigger(stubId, 'delete_book')
		where:
			stubId << ['org.springframework.cloud.contract.verifier.stubs:bootService:stubs', 'org.springframework.cloud.contract.verifier.stubs:bootService', 'bootService']
	}

	def 'should throw exception when trigger is missing'() {
		when:
			RestAssuredMockMvc.post("/triggers/missing_label")
		then:
			Exception e = thrown(Exception)
			e.message.contains("Exception occurred while trying to return [missing_label] label.")
			e.message.contains("Available labels are")
			e.message.contains("org.springframework.cloud.contract.verifier.stubs:loanIssuance:0.0.1-SNAPSHOT:stubs=[]")
			e.message.contains("org.springframework.cloud.contract.verifier.stubs:bootService:0.0.1-SNAPSHOT:stubs=")
	}

}
```

### 93.6.4 Stub Runner Boot with Service Discovery

使用Stub Runner Boot的一种可能性是将其用作“烟雾测试”存根的馈送.这是什么意思？假设您不希望将50个微服务部署到测试环境中，以检查您的应用程序是否正常工作.您已经在构建过程中执行了一系列测试，但您还希望确保应用程序的包装正常.你可以做的是将你的应用程序部署到一个环境，启动它并对它运行几个测试，看看它是否正常工作.我们可以将这些测试称为烟雾测试，因为他们的想法是只检查少数测试场景.

这种方法的问题在于，如果您正在使用微服务，那么您很可能正在使用服务发现工具. Stub Runner Boot允许您通过启动所需的存根并在服务发现工具中注册它们来解决此问题.让我们来看看Eureka这样一个设置的例子.让我们假设尤里卡已经在运行.

```java
@SpringBootApplication
@EnableStubRunnerServer
@EnableEurekaClient
@AutoConfigureStubRunner
public class StubRunnerBootEurekaExample {

	public static void main(String[] args) {
		SpringApplication.run(StubRunnerBootEurekaExample.class, args);
	}

}
```

正如您所看到的，我们想要启动Stub Runner Boot服务器 `@EnableStubRunnerServer` ，启用Eureka客户端 `@EnableEurekaClient` ，我们希望启用存根运行器功能 `@AutoConfigureStubRunner` .

现在让我们假设我们想要启动这个应用程序，以便自动注册存根.我们可以通过运行app  `java -jar ${SYSTEM_PROPS} stub-runner-boot-eureka-example.jar` 来实现，其中 `${SYSTEM_PROPS}` 将包含以下属性列表

```java
-Dstubrunner.repositoryRoot=http://repo.spring.io/snapshots (1)
-Dstubrunner.cloud.stubbed.discovery.enabled=false (2)
-Dstubrunner.ids=org.springframework.cloud.contract.verifier.stubs:loanIssuance,org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer,org.springframework.cloud.contract.verifier.stubs:bootService (3)
-Dstubrunner.idsToServiceIds.fraudDetectionServer=someNameThatShouldMapFraudDetectionServer (4)

(1) - we tell Stub Runner where all the stubs reside
(2) - we don't want the default behaviour where the discovery service is stubbed. That's why the stub registration will be picked
(3) - we provide a list of stubs to download
(4) - we provide a list of artifactId to serviceId mapping
```

这样，您部署的应用程序可以通过服务发现向已启动的WireMock服务器发送请求.最可能的点1-3可以在 `application.yml` 中默认设置，因为它们不太可能改变.这样，您只能提供要随时下载的存根列表你启动Stub Runner Boot.

## 93.7每消费者存根

在某些情况下，同一endpoints的2个消费者希望有2个不同的响应.

> This方法还允许您立即知道哪个消费者正在使用您的API的哪个部分.您可以删除API生成的部分响应，并且可以看到哪些自动生成的测试失败.如果没有失败，那么您可以安全地删除响应的那部分因为没有人使用它.

让我们看一下为生产环境者定义的Contract `producer` 的以下示例.有2个消费者： `foo-consumer` 和 `bar-consumer` .

**Consumer foo-service** 

```java
request {
url '/foo'
method GET()
}
response {
status OK()
body(
foo: "foo"
}
}
```

**Consumer bar-service** 

```java
request {
url '/foo'
method GET()
}
response {
status OK()
body(
bar: "bar"
}
}
```

您不能为同一请求生成2个不同的响应.这就是为什么你可以正确打包Contract然后从 `stubsPerConsumer` 功能中获利的原因.

在生产环境者方面，消费者可以拥有一个包含仅与它们相关的Contract的文件夹.通过将 `stubrunner.stubs-per-consumer` 标志设置为 `true` ，我们不再注册所有存根，而只注册与消费者应用程序名称对应的存根.换句话说，我们将扫描每个存根的路径，如果它只包含路径中具有消费者名称的子文件夹，那么它是否会被注册.

在 `foo` 生产环境者方面，Contract将如下所示

```java
.
└── contracts
├── bar-consumer
│ ├── bookReturnedForBar.groovy
│ └── shouldCallBar.groovy
└── foo-consumer
├── bookReturnedForFoo.groovy
└── shouldCallFoo.groovy
```

作为 `bar-consumer` 使用者，您可以将 `spring.application.name` 或 `stubrunner.consumer-name` 设置为 `bar-consumer` 或者按如下方式设置测试：

```java
@ContextConfiguration(classes = Config, loader = SpringBootContextLoader)
@SpringBootTest(properties = ["spring.application.name=bar-consumer"])
@AutoConfigureStubRunner(ids = "org.springframework.cloud.contract.verifier.stubs:producerWithMultipleConsumers",
		repositoryRoot = "classpath:m2repo/repository/",
		stubsMode = StubRunnerProperties.StubsMode.REMOTE,
		stubsPerConsumer = true)
class StubRunnerStubsPerConsumerSpec extends Specification {
...
}
```

然后，只允许引用在其名称中包含 `bar-consumer` 的路径下注册的存根（即来自 `src/test/resources/contracts/bar-consumer/some/contracts/…` 文件夹的存根）.

或者明确设置消费者名称

```java
@ContextConfiguration(classes = Config, loader = SpringBootContextLoader)
@SpringBootTest
@AutoConfigureStubRunner(ids = "org.springframework.cloud.contract.verifier.stubs:producerWithMultipleConsumers",
		repositoryRoot = "classpath:m2repo/repository/",
		consumerName = "foo-consumer",
		stubsMode = StubRunnerProperties.StubsMode.REMOTE,
		stubsPerConsumer = true)
class StubRunnerStubsPerConsumerWithConsumerNameSpec extends Specification {
...
}
```

然后，只允许引用在其名称中包含 `foo-consumer` 的路径下注册的存根（即来自 `src/test/resources/contracts/foo-consumer/some/contracts/…` 文件夹的存根）.

您可以查看[issue 224](https://github.com/spring-cloud/spring-cloud-contract/issues/224)，了解有关此更改背后原因的更多信息.

## 93.8常见

本节简要介绍常见属性，包括：

- [Section 93.8.1, “Common Properties for JUnit and Spring”](multi__spring_cloud_contract_stub_runner.html#common-properties-junit-spring)

- [Section 93.8.2, “Stub Runner Stubs IDs”](multi__spring_cloud_contract_stub_runner.html#stub-runner-stub-ids)

### 93.8.1 JUnit和Spring的公共属性

您可以使用系统属性或Spring配置属性设置重复属性.以下是其名称及其默认值：

|属性名称|默认值|描述|
| ---- | ---- | ---- |
| stubrunner.minPort | 10000 |已启动的带有存根的WireMock的端口的最小值. |
| stubrunner.maxPort | 15000 |具有存根的已启动WireMock的端口的最大值. |
| stubrunner.repositoryRoot | | Maven repo URL.如果为空，则调用本地maven仓库. |
| stubrunner.classifier | stubs |存根工件的默认分类器. |
| stubrunner.stubsMode | CLASSPATH |您想要获取和注册存根的方式
| stubrunner.ids | |要下载的常Spring藤符号存根的数组. |
| stubrunner.username | |可选用户名，用于访问存储带存根的JAR的工具. |
| stubrunner.password | |可选密码，用于访问存储带存根的JAR的工具. |
| stubrunner.stubsPerConsumer | false |如果要为每个使用者使用不同的存根，而不是为每个使用者注册所有存根，则设置为 `true` . |
| stubrunner.consumerName | |如果要为每个使用者使用存根，并希望覆盖使用者名称，只需更改此值. |

### 93.8.2 Stub Runner存根ID

您可以通过 `stubrunner.ids` 系统属性提供要下载的存根.他们遵循这种模式：

```java
groupId:artifactId:version:classifier:port
```

请注意 `version` ， `classifier` 和 `port` 是可选的.

- 如果您没有提供 `port` ，将会选择一个随机的.

- 如果您未提供 `classifier` ，则使用默认值. （请注意，您可以通过这种方式传递空分类器： `groupId:artifactId:version:` ）.

- 如果您未提供 `version` ，则将传递 `+` 并下载最新的 `+` .

`port` 表示WireMock服务器的端口.

|图片/ important.png |重要|
| ---- | ---- |
|从版本1.0.4开始，您可以提供一系列您希望Stub Runner考虑的版本.您可以阅读有关[Aether versioning ranges here](https://wiki.eclipse.org/Aether/New_and_Noteworthy#Version_Ranges)的更多信息. |

## 93.9 Stub Runner Docker

我们正在发布一个 `spring-cloud/spring-cloud-contract-stub-runner`  Docker镜像，它将启动Stub Runner的独立版本.

如果您想了解有关Maven，工件ID，组ID，分类器和工件管理器的基础知识的更多信息，请单击此处[Section 91.6, “Docker Project”](multi__spring_cloud_contract_verifier_setup.html#docker-project).

### 93.9.1如何使用它

只需执行泊坞窗图像.您可以传递任何[Section 93.8.1, “Common Properties for JUnit and Spring”](multi__spring_cloud_contract_stub_runner.html#common-properties-junit-spring)作为环境变量.惯例是所有字母都应该是大写的.驼峰表示法应该和点（ `.` ）应该通过下划线（ `_` ）分开.例如. `stubrunner.repositoryRoot` 属性应表示为 `STUBRUNNER_REPOSITORY_ROOT` 环境变量.

### 93.9.2非JVM项目中客户端使用的示例

我们想使用在[Section 91.6.4, “Server side (nodejs)”](multi__spring_cloud_contract_verifier_setup.html#docker-server-side)步骤中创建的存根.假设我们想在端口 `9876` 上运行存根. NodeJS代码可在此处获得：

```java
$ git clone https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs
$ cd bookstore
```

让我们使用存根运行Stub Runner Boot应用程序.

```java
# Provide the Spring Cloud Contract Docker version
$ SC_CONTRACT_DOCKER_VERSION="..."
# The IP at which the app is running and Docker container can reach it
$ APP_IP="192.168.0.100"
# Spring Cloud Contract Stub Runner properties
$ STUBRUNNER_PORT="8083"
# Stub coordinates 'groupId:artifactId:version:classifier:port'
$ STUBRUNNER_IDS="com.example:bookstore:0.0.1.RELEASE:stubs:9876"
$ STUBRUNNER_REPOSITORY_ROOT="http://${APP_IP}:8081/artifactory/libs-release-local"
# Run the docker with Stub Runner Boot
$ docker run  --rm -e "STUBRUNNER_IDS=${STUBRUNNER_IDS}" -e "STUBRUNNER_REPOSITORY_ROOT=${STUBRUNNER_REPOSITORY_ROOT}" -e "STUBRUNNER_STUBS_MODE=REMOTE" -p "${STUBRUNNER_PORT}:${STUBRUNNER_PORT}" -p "9876:9876" springcloud/spring-cloud-contract-stub-runner:"${SC_CONTRACT_DOCKER_VERSION}"
```

发生了什么事

- a独立的Stub Runner应用程序启动了

- it在端口上下载了坐标为 `com.example:bookstore:0.0.1.RELEASE:stubs` 的存根 `9876` 

- it从运行于 `http://192.168.0.100:8081/artifactory/libs-release-local` 的Artifactory下载

- 暂时Stub Runner将在端口 `8083` 上运行

- 并且存根将在端口 `9876` 上运行

在服务器端，我们构建了一个有状态存根.让我们使用curl断言存根设置正确.

```java
# let's execute the first request (no response is returned)
$ curl -H "Content-Type:application/json" -X POST --data '{ "title" : "Title", "genre" : "Genre", "description" : "Description", "author" : "Author", "publisher" : "Publisher", "pages" : 100, "image_url" : "https://d213dhlpdb53mu.cloudfront.net/assets/pivotal-square-logo-41418bd391196c3022f3cd9f3959b3f6d7764c47873d858583384e759c7db435.svg", "buy_url" : "https://pivotal.io" }' http://localhost:9876/api/books
# Now time for the second request
$ curl -X GET http://localhost:9876/api/books
# You will receive contents of the JSON
```

|图片/ important.png |重要|
| ---- | ---- |
|如果要在主机上使用本地构建的存根，则应传递环境变量 `-e STUBRUNNER_STUBS_MODE=LOCAL` 并装入本地m2的音量 `-v "${HOME}/.m2/:/root/.m2:ro"`  |


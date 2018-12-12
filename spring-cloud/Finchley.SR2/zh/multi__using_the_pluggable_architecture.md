## 97.使用可插拔架构

您可能会遇到以下其他格式定义Contract的情况，例如YAML，RAML或PACT.在这些情况下，您仍然希望从自动生成测试和存根中受益.您可以添加自己的实现来生成测试和存根.此外，您可以自定义生成测试的方式（例如，您可以为其他语言生成测试）以及生成存根的方式（例如，您可以为其他HTTP服务器实现生成存根）.

## 97.1自定义Contract转换器

`ContractConverter` 接口允许您注册自己的Contract结构转换器实现.以下代码清单显示了 `ContractConverter` 接口：

```java
package org.springframework.cloud.contract.spec

/**
* Converter to be used to convert FROM {@link File} TO {@link Contract}
* and from {@link Contract} to {@code T}
*
* @param <T> - type to which we want to convert the contract
*
* @author Marcin Grzejszczak
* @since 1.1.0
*/
interface ContractConverter<T> {

	/**
	 * Should this file be accepted by the converter. Can use the file extension
	 * to check if the conversion is possible.
	 *
	 * @param file - file to be considered for conversion
	 * @return - {@code true} if the given implementation can convert the file
	 */
	boolean isAccepted(File file)

	/**
	 * Converts the given {@link File} to its {@link Contract} representation
	 *
	 * @param file - file to convert
	 * @return - {@link Contract} representation of the file
	 */
	Collection<Contract> convertFrom(File file)

	/**
	 * Converts the given {@link Contract} to a {@link T} representation
	 *
	 * @param contract - the parsed contract
	 * @return - {@link T} the type to which we do the conversion
	 */
	T convertTo(Collection<Contract> contract)
}
```

您的实现必须定义它应该开始转换的条件.此外，您必须定义如何在两个方向上执行该转换.

|图片/ important.png |重要|
| ---- | ---- |
|创建实现后，必须创建一个 `/META-INF/spring.factories` 文件，在该文件中提供实现的完全限定名称. |

以下示例显示了典型的 `spring.factories` 文件：

```java
org.springframework.cloud.contract.spec.ContractConverter=\
org.springframework.cloud.contract.verifier.converter.YamlContractConverter
```

### 97.1.1 Pact Converter

Spring Cloud Contract包括对v4之前Contract的[Pact](https://docs.pact.io/)表示的支持.您可以使用Pact文件，而不是使用Groovy DSL.在本节中，我们将介绍如何为项目添加Pact支持.但请注意，并非所有功能都受支持.从v3开始，您可以为同一元素组合多个匹配器;你可以使用匹配器作为正文，Headers，请求和路径;你可以使用Value生成器. Spring Cloud Contract目前仅支持使用AND规则逻辑组合的多个匹配器.在此之后，转换期间将跳过请求和路径匹配器.当使用具有给定格式的日期，时间或日期时间值生成器时，将跳过给定格式并使用ISO格式.

### 97.1.2ContractContract

请考虑下面的ContractContract示例，该Contract是 `src/test/resources/contracts` 文件夹下的文件.

```java
{
"provider": {
"name": "Provider"
},
"consumer": {
"name": "Consumer"
},
"interactions": [
{
"description": "",
"request": {
"method": "PUT",
"path": "/fraudcheck",
"headers": {
"Content-Type": "application/vnd.fraud.v1+json"
},
"body": {
"clientId": "1234567890",
"loanAmount": 99999
},
"generators": {
"body": {
"$.clientId": {
"type": "Regex",
"regex": "[0-9]{10}"
}
}
},
"matchingRules": {
"header": {
"Content-Type": {
"matchers": [
{
"match": "regex",
"regex": "application/vnd\\.fraud\\.v1\\+json.*"
}
],
"combine": "AND"
}
},
"body" : {
"$.clientId": {
"matchers": [
{
"match": "regex",
"regex": "[0-9]{10}"
}
],
"combine": "AND"
}
}
}
},
"response": {
"status": 200,
"headers": {
"Content-Type": "application/vnd.fraud.v1+json;charset=UTF-8"
},
"body": {
"fraudCheckStatus": "FRAUD",
"rejectionReason": "Amount too high"
},
"matchingRules": {
"header": {
"Content-Type": {
"matchers": [
{
"match": "regex",
"regex": "application/vnd\\.fraud\\.v1\\+json.*"
}
],
"combine": "AND"
}
},
"body": {
"$.fraudCheckStatus": {
"matchers": [
{
"match": "regex",
"regex": "FRAUD"
}
],
"combine": "AND"
}
}
}
}
}
],
"metadata": {
"pact-specification": {
"version": "3.0.0"
},
"pact-jvm": {
"version": "3.5.13"
}
}
}
```

关于使用Pact的本节的其余部分是指前面的文件.

### 97.1.3生产环境者协议

在生产环境者方面，您必须为插件配置添加两个额外的依赖项.一个是Spring Cloud Contract Pact支持，另一个代表您使用的当前Pact版本.

**Maven.** 

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<version>${spring-cloud-contract.version}</version>
	<extensions>true</extensions>
	<configuration>
		<packageWithBaseClasses>com.example.fraud</packageWithBaseClasses>
	</configuration>
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
classpath "org.springframework.cloud:spring-cloud-contract-pact:${findProperty('verifierVersion') ?: verifierVersion}"
```

执行应用程序的构建时，将生成测试.生成的测试可能如下：

```java
@Test
public void validate_shouldMarkClientAsFraud() throws Exception {
	// given:
		MockMvcRequestSpecification request = given()
				.header("Content-Type", "application/vnd.fraud.v1+json")
				.body("{\"clientId\":\"1234567890\",\"loanAmount\":99999}");

	// when:
		ResponseOptions response = given().spec(request)
				.put("/fraudcheck");

	// then:
		assertThat(response.statusCode()).isEqualTo(200);
		assertThat(response.header("Content-Type")).matches("application/vnd\\.fraud\\.v1\\+json.*");
	// and:
		DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
		assertThatJson(parsedJson).field("['rejectionReason']").isEqualTo("Amount too high");
	// and:
		assertThat(parsedJson.read("$.fraudCheckStatus", String.class)).matches("FRAUD");
}
```

相应的生成存根可能如下：

```java
{
"id" : "996ae5ae-6834-4db6-8fac-358ca187ab62",
"uuid" : "996ae5ae-6834-4db6-8fac-358ca187ab62",
"request" : {
"url" : "/fraudcheck",
"method" : "PUT",
"headers" : {
"Content-Type" : {
"matches" : "application/vnd\\.fraud\\.v1\\+json.*"
}
},
"bodyPatterns" : [ {
"matchesJsonPath" : "$[?(@.['loanAmount'] == 99999)]"
}, {
"matchesJsonPath" : "$[?(@.clientId =~ /([0-9]{10})/)]"
} ]
},
"response" : {
"status" : 200,
"body" : "{\"fraudCheckStatus\":\"FRAUD\",\"rejectionReason\":\"Amount too high\"}",
"headers" : {
"Content-Type" : "application/vnd.fraud.v1+json;charset=UTF-8"
},
"transformers" : [ "response-template" ]
},
}
```

### 97.1.4消费者协议

在生产环境者方面，您必须为项目依赖项添加两个额外的依赖项.一个是Spring Cloud Contract Pact支持，另一个代表您使用的当前Pact版本.

**Maven.** 

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-pact</artifactId>
	<scope>test</scope>
</dependency>
```

**Gradle.** 

```java
testCompile "org.springframework.cloud:spring-cloud-contract-pact"
```

## 97.2使用自定义测试生成器

如果要生成测试除了Java以外的语言，或者您对验证程序构建Java测试的方式不满意，您可以注册自己的实现.

`SingleTestGenerator` 界面允许您注册自己的实现.以下代码清单显示了 `SingleTestGenerator` 接口：

```java
package org.springframework.cloud.contract.verifier.builder

import org.springframework.cloud.contract.verifier.config.ContractVerifierConfigProperties
import org.springframework.cloud.contract.verifier.file.ContractMetadata
/**
* Builds a single test.
*
* @since 1.1.0
*/
interface SingleTestGenerator {

	/**
	 * Creates contents of a single test class in which all test scenarios from
	 * the contract metadata should be placed.
	 *
	 * @param properties - properties passed to the plugin
	 * @param listOfFiles - list of parsed contracts with additional metadata
	 * @param className - the name of the generated test class
	 * @param classPackage - the name of the package in which the test class should be stored
	 * @param includedDirectoryRelativePath - relative path to the included directory
	 * @return contents of a single test class
	 */
	String buildClass(ContractVerifierConfigProperties properties, Collection<ContractMetadata> listOfFiles,
					  String className, String classPackage, String includedDirectoryRelativePath)

	/**
	 * Extension that should be appended to the generated test class. E.g. {@code .java} or {@code .php}
	 *
	 * @param properties - properties passed to the plugin
	 */
	String fileExtension(ContractVerifierConfigProperties properties)
}
```

同样，您必须提供 `spring.factories` 文件，例如以下示例中显示的文件：

```java
org.springframework.cloud.contract.verifier.builder.SingleTestGenerator=/
com.example.MyGenerator
```

## 97.3使用自定义存根生成器

如果要为WireMock以外的存根服务器生成存根，可以插入自己的 `StubGenerator` 接口实现.以下代码清单显示了 `StubGenerator` 接口：

```java
package org.springframework.cloud.contract.verifier.converter

import groovy.transform.CompileStatic
import org.springframework.cloud.contract.spec.Contract
import org.springframework.cloud.contract.verifier.file.ContractMetadata

/**
* Converts contracts into their stub representation.
*
* @since 1.1.0
*/
@CompileStatic
interface StubGenerator {

	/**
	 * Returns {@code true} if the converter can handle the file to convert it into a stub.
	 */
	boolean canHandleFileName(String fileName)

	/**
	 * Returns the collection of converted contracts into stubs. One contract can
	 * result in multiple stubs.
	 */
	Map<Contract, String> convertContents(String rootName, ContractMetadata content)

	/**
	 * Returns the name of the converted stub file. If you have multiple contracts
	 * in a single file then a prefix will be added to the generated file. If you
	 * provide the {@link Contract#name} field then that field will override the
	 * generated file name.
	 *
	 * Example: name of file with 2 contracts is {@code foo.groovy}, it will be
	 * converted by the implementation to {@code foo.json}. The recursive file
	 * converter will create two files {@code 0_foo.json} and {@code 1_foo.json}
	 */
	String generateOutputFileNameForInput(String inputFileName)
}
```

同样，您必须提供 `spring.factories` 文件，例如以下示例中显示的文件：

```java
# Stub converters
org.springframework.cloud.contract.verifier.converter.StubGenerator=\
org.springframework.cloud.contract.verifier.wiremock.DslToWireMockClientConverter
```

默认实现是WireMock存根生成.

> 您可以提供多个存根生成器实现.例如，从单个DSL，您可以生成WireMock存根和Pact文件.

## 97.4使用自定义存根运行器

如果您决定使用自定义存根生成，则还需要使用不同存根提供程序自定义运行存根的方法.

假设您使用[Moco](https://github.com/dreamhead/moco)构建存根，并且已编写存根生成器并将存根放在JAR文件中.

为了让Stub Runner知道如何运行存根，您必须定义自定义HTTP Stub服务器实现，这可能类似于以下示例：

```java
package org.springframework.cloud.contract.stubrunner.provider.moco

import com.github.dreamhead.moco.bootstrap.arg.HttpArgs
import com.github.dreamhead.moco.runner.JsonRunner
import com.github.dreamhead.moco.runner.RunnerSetting
import groovy.util.logging.Commons

import org.springframework.cloud.contract.stubrunner.HttpServerStub
import org.springframework.util.SocketUtils

@Commons
class MocoHttpServerStub implements HttpServerStub {

	private boolean started
	private JsonRunner runner
	private int port

	@Override
	int port() {
		if (!isRunning()) {
			return -1
		}
		return port
	}

	@Override
	boolean isRunning() {
		return started
	}

	@Override
	HttpServerStub start() {
		return start(SocketUtils.findAvailableTcpPort())
	}

	@Override
	HttpServerStub start(int port) {
		this.port = port
		return this
	}

	@Override
	HttpServerStub stop() {
		if (!isRunning()) {
			return this
		}
		this.runner.stop()
		return this
	}

	@Override
	HttpServerStub registerMappings(Collection<File> stubFiles) {
		List<RunnerSetting> settings = stubFiles.findAll { it.name.endsWith("json") }
				.collect {
			log.info("Trying to parse [${it.name}]")
			try {
				return RunnerSetting.aRunnerSetting().withStream(it.newInputStream()).build()
			} catch (Exception e) {
				log.warn("Exception occurred while trying to parse file [${it.name}]", e)
				return null
			}
		}.findAll { it }
		this.runner = JsonRunner.newJsonRunnerWithSetting(settings,
				HttpArgs.httpArgs().withPort(this.port).build())
		this.runner.run()
		this.started = true
		return this
	}

	@Override
	String registeredMappings() {
		return ""
	}

	@Override
	boolean isAccepted(File file) {
		return file.name.endsWith(".json")
	}
}
```

然后，您可以在 `spring.factories` 文件中注册它，如以下示例所示：

```java
org.springframework.cloud.contract.stubrunner.HttpServerStub=\
org.springframework.cloud.contract.stubrunner.provider.moco.MocoHttpServerStub
```

现在您可以使用Moco运行存根.

|图片/ important.png |重要|
| ---- | ---- |
|如果您未提供任何实现，则使用默认（WireMock）实现.如果提供多个，则使用列表中的第一个. |

## 97.5使用自定义存根下载程序

您可以通过创建 `StubDownloaderBuilder` 接口的实现来自定义存根的下载方式，如以下示例所示：

```java
package com.example;

class CustomStubDownloaderBuilder implements StubDownloaderBuilder {

	@Override
	public StubDownloader build(final StubRunnerOptions stubRunnerOptions) {
		return new StubDownloader() {
			@Override
			public Map.Entry<StubConfiguration, File> downloadAndUnpackStubJar(
					StubConfiguration config) {
				File unpackedStubs = retrieveStubs();
				return new AbstractMap.SimpleEntry<>(
						new StubConfiguration(config.getGroupId(), config.getArtifactId(), version,
								config.getClassifier()), unpackedStubs);
			}

			File retrieveStubs() {
			    // here goes your custom logic to provide a folder where all the stubs reside
			}
}
```

然后，您可以在 `spring.factories` 文件中注册它，如以下示例所示：

```java
# Example of a custom Stub Downloader Provider
org.springframework.cloud.contract.stubrunner.StubDownloaderBuilder=\
com.example.CustomStubDownloaderBuilder
```

现在，您可以选择包含存根源的文件夹.

|图片/ important.png |重要|
| ---- | ---- |
|如果未提供任何实现，则使用默认值（扫描类路径）.如果您提供 `stubsMode = StubRunnerProperties.StubsMode.LOCAL` 或 `, stubsMode = StubRunnerProperties.StubsMode.REMOTE` ，那么将使用Aether实现如果您提供多个，则使用列表中的第一个. |

## 97.6使用SCM Stub Downloader

每当 `repositoryRoot` 以SCM协议（目前我们仅支持 `git://` ）启动时，存根下载器将尝试克隆存储库并将其用作生成测试或存根的Contract源.

通过环境变量，系统属性，插件内部设置的属性或Contract存储库配置，您可以调整下载程序的行为.您可以在下面找到属性列表

**Table 97.1. SCM Stub Downloader properties** 

|属性类型|属性名称|描述|
| ---- | ---- | ---- |
| *  `git.branch` （插件道具）*  `stubrunner.properties.git.branch` （系统道具）*  `STUBRUNNER_PROPERTIES_GIT_BRANCH` （env prop）| master |哪个分支结帐|
| *  `git.username` （插件道具）*  `stubrunner.properties.git.username` （系统道具）*  `STUBRUNNER_PROPERTIES_GIT_USERNAME` （env prop）| | Git克隆用户名|
| *  `git.password` （插件道具）*  `stubrunner.properties.git.password` （系统道具）*  `STUBRUNNER_PROPERTIES_GIT_PASSWORD` （env prop）| | Git克隆密码|
| *  `git.no-of-attempts` （插件道具）*  `stubrunner.properties.git.no-of-attempts` （系统道具）*  `STUBRUNNER_PROPERTIES_GIT_NO_OF_ATTEMPTS` （env prop）| 10 |尝试将提交推送到 `origin`  |
| *  `git.wait-between-attempts` （插件道具）*  `stubrunner.properties.git.wait-between-attempts` （系统道具）*  `STUBRUNNER_PROPERTIES_GIT_WAIT_BETWEEN_ATTEMPTS` （env prop）| 1000 |尝试将提交推送到 `origin` 之间等待的毫秒数

## 97.7使用Pact Stub Downloader

每当 `repositoryRoot` 以Pact协议（以 `pact://` 开头）启动时，存根下载器将尝试从Pact Broker获取Pact合约定义.在 `pact://` 之后设置的任何内容都将被解析为Pact Broker URL.

通过环境变量，系统属性，插件内部设置的属性或Contract存储库配置，您可以调整下载程序的行为.您可以在下面找到属性列表

**Table 97.2. SCM Stub Downloader properties** 

|属性名称|默认|描述|
| ---- | ---- | ---- |
| *  `pactbroker.host` （插件道具）*  `stubrunner.properties.pactbroker.host` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_HOST` （env prop）|来自URL的主机传递给 `repositoryRoot`  | Pact Broker的URL是什么？
| *  `pactbroker.port` （插件道具）*  `stubrunner.properties.pactbroker.port` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_PORT` （env prop）|从URL传递到 `repositoryRoot` 的端口| Pact Broker的端口是什么？
| *  `pactbroker.protocol` （插件道具）*  `stubrunner.properties.pactbroker.protocol` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_PROTOCOL` （env prop）|来自URL的协议传递给 `repositoryRoot`  | Pact Broker的协议是什么？
| *  `pactbroker.tags` （插件道具）*  `stubrunner.properties.pactbroker.tags` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_TAGS` （env prop）|存根的版本，如果版本是 `+` 则为 `latest`  |应该使用什么标签来获取存根|
| *  `pactbroker.auth.scheme` （插件道具）*  `stubrunner.properties.pactbroker.auth.scheme` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_AUTH_SCHEME` （env prop）|  `Basic`  |应使用哪种身份验证来连接Pact Broker.
| *  `pactbroker.auth.username` （插件道具）*  `stubrunner.properties.pactbroker.auth.username` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_AUTH_USERNAME` （env prop）|传递给 `contractsRepositoryUsername` （maven）或 `contractRepository.username` （gradle）的用户名|用于连接Pact Broker的用户名|
| *  `pactbroker.auth.password` （插件道具）*  `stubrunner.properties.pactbroker.auth.password` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_AUTH_PASSWORD` （env prop）|传递给 `contractsRepositoryPassword` （maven）或 `contractRepository.password` （gradle）的密码|用于连接Pact Broker的密码
| *  `pactbroker.provider-name-with-group-id` （插件道具）*  `stubrunner.properties.pactbroker.provider-name-with-group-id` （系统道具）*  `STUBRUNNER_PROPERTIES_PACTBROKER_PROVIDER_NAME_WITH_GROUP_ID` （env prop）| false |当 `true` 时，提供者名称将是 `groupId:artifactId` 的组合.如果 `false` ，则使用 `artifactId` 


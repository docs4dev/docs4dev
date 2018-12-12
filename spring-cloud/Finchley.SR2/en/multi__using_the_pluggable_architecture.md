## 97. Using the Pluggable Architecture

You may encounter cases where you have your contracts have been defined in other formats, such as YAML, RAML or PACT. In those cases, you still want to benefit from the automatic generation of tests and stubs. You can add your own implementation for generating both tests and stubs. Also, you can customize the way tests are generated (for example, you can generate tests for other languages) and the way stubs are generated (for example, you can generate stubs for other HTTP server implementations).

## 97.1 Custom Contract Converter

The  `ContractConverter`  interface lets you register your own implementation of a contract structure converter. The following code listing shows the  `ContractConverter`  interface:

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

Your implementation must define the condition on which it should start the conversion. Also, you must define how to perform that conversion in both directions.

|images/important.png|Important|
|----|----|
|Once you create your implementation, you must create a  `/META-INF/spring.factories`  file in which you provide the fully qualified name of your implementation. |

The following example shows a typical  `spring.factories`  file:

```java
org.springframework.cloud.contract.spec.ContractConverter=\
org.springframework.cloud.contract.verifier.converter.YamlContractConverter
```

### 97.1.1 Pact Converter

Spring Cloud Contract includes support for [Pact](https://docs.pact.io/) representation of contracts up until v4. Instead of using the Groovy DSL, you can use Pact files. In this section, we present how to add Pact support for your project. Note however that not all functionality is supported. Starting with v3 you can combine multiple matcher for the same element; you can use matchers for the body, headers, request and path; and you can use value generators. Spring Cloud Contract currently only supports multiple matchers that are combined using the AND rule logic. Next to that the request and path matchers are skipped during the conversion. When using a date, time or datetime value generator with a given format, the given format will be skipped and the ISO format will be used.

### 97.1.2 Pact Contract

Consider following example of a Pact contract, which is a file under the  `src/test/resources/contracts`  folder.

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

The remainder of this section about using Pact refers to the preceding file.

### 97.1.3 Pact for Producers

On the producer side, you must add two additional dependencies to your plugin configuration. One is the Spring Cloud Contract Pact support, and the other represents the current Pact version that you use.

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

When you execute the build of your application, a test will be generated. The generated test might be as follows:

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

The corresponding generated stub might be as follows:

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

### 97.1.4 Pact for Consumers

On the producer side, you must add two additional dependencies to your project dependencies. One is the Spring Cloud Contract Pact support, and the other represents the current Pact version that you use.

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

## 97.2 Using the Custom Test Generator

If you want to generate tests for languages other than Java or you are not happy with the way the verifier builds Java tests, you can register your own implementation.

The  `SingleTestGenerator`  interface lets you register your own implementation. The following code listing shows the  `SingleTestGenerator`  interface:

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

Again, you must provide a  `spring.factories`  file, such as the one shown in the following example:

```java
org.springframework.cloud.contract.verifier.builder.SingleTestGenerator=/
com.example.MyGenerator
```

## 97.3 Using the Custom Stub Generator

If you want to generate stubs for stub servers other than WireMock, you can plug in your own implementation of the  `StubGenerator`  interface. The following code listing shows the  `StubGenerator`  interface:

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

Again, you must provide a  `spring.factories`  file, such as the one shown in the following example:

```java
# Stub converters
org.springframework.cloud.contract.verifier.converter.StubGenerator=\
org.springframework.cloud.contract.verifier.wiremock.DslToWireMockClientConverter
```

The default implementation is the WireMock stub generation.

> You can provide multiple stub generator implementations. For example, from a single DSL, you can produce both WireMock stubs and Pact files.

## 97.4 Using the Custom Stub Runner

If you decide to use a custom stub generation, you also need a custom way of running stubs with your different stub provider.

Assume that you use [Moco](https://github.com/dreamhead/moco) to build your stubs and that you have written a stub generator and placed your stubs in a JAR file.

In order for Stub Runner to know how to run your stubs, you have to define a custom HTTP Stub server implementation, which might resemble the following example:

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

Then, you can register it in your  `spring.factories`  file, as shown in the following example:

```java
org.springframework.cloud.contract.stubrunner.HttpServerStub=\
org.springframework.cloud.contract.stubrunner.provider.moco.MocoHttpServerStub
```

Now you can run stubs with Moco.

|images/important.png|Important|
|----|----|
|If you do not provide any implementation, then the default (WireMock) implementation is used. If you provide more than one, the first one on the list is used. |

## 97.5 Using the Custom Stub Downloader

You can customize the way your stubs are downloaded by creating an implementation of the  `StubDownloaderBuilder`  interface, as shown in the following example:

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

Then you can register it in your  `spring.factories`  file, as shown in the following example:

```java
# Example of a custom Stub Downloader Provider
org.springframework.cloud.contract.stubrunner.StubDownloaderBuilder=\
com.example.CustomStubDownloaderBuilder
```

Now you can pick a folder with the source of your stubs.

|images/important.png|Important|
|----|----|
|If you do not provide any implementation, then the default is used (scan classpath). If you provide the  `stubsMode = StubRunnerProperties.StubsMode.LOCAL`  or  `, stubsMode = StubRunnerProperties.StubsMode.REMOTE`  then the Aether implementation will be used If you provide more than one, then the first one on the list is used. |

## 97.6 Using the SCM Stub Downloader

Whenever the  `repositoryRoot`  starts with a SCM protocol (currently we support only  `git://` ), the stub downloader will try to clone the repository and use it as a source of contracts to generate tests or stubs.

Either via environment variables, system properties, properties set inside the plugin or contracts repository configuration you can tweak the downloader’s behaviour. Below you can find the list of properties

**Table 97.1. SCM Stub Downloader properties** 

|Type of a property |Name of the property |Description |
|----|----|----|
|*  `git.branch`  (plugin prop)   *  `stubrunner.properties.git.branch`  (system prop)   *  `STUBRUNNER_PROPERTIES_GIT_BRANCH`  (env prop) |master |Which branch to checkout |
|*  `git.username`  (plugin prop)   *  `stubrunner.properties.git.username`  (system prop)   *  `STUBRUNNER_PROPERTIES_GIT_USERNAME`  (env prop) | |Git clone username |
|*  `git.password`  (plugin prop)   *  `stubrunner.properties.git.password`  (system prop)   *  `STUBRUNNER_PROPERTIES_GIT_PASSWORD`  (env prop) | |Git clone password |
|*  `git.no-of-attempts`  (plugin prop)   *  `stubrunner.properties.git.no-of-attempts`  (system prop)   *  `STUBRUNNER_PROPERTIES_GIT_NO_OF_ATTEMPTS`  (env prop) |10 |Number of attempts to push the commits to  `origin`  |
|*  `git.wait-between-attempts`  (Plugin prop)   *  `stubrunner.properties.git.wait-between-attempts`  (system prop)   *  `STUBRUNNER_PROPERTIES_GIT_WAIT_BETWEEN_ATTEMPTS`  (env prop) |1000 |Number of millis to wait between attempts to push the commits to  `origin`  |

## 97.7 Using the Pact Stub Downloader

Whenever the  `repositoryRoot`  starts with a Pact protocol (starts with  `pact://` ), the stub downloader will try to fetch the Pact contract definitions from the Pact Broker. Whatever is set after  `pact://`  will be parsed as the Pact Broker URL.

Either via environment variables, system properties, properties set inside the plugin or contracts repository configuration you can tweak the downloader’s behaviour. Below you can find the list of properties

**Table 97.2. SCM Stub Downloader properties** 

|Name of a property |Default |Description |
|----|----|----|
|*  `pactbroker.host`  (plugin prop)   *  `stubrunner.properties.pactbroker.host`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_HOST`  (env prop) |Host from URL passed to  `repositoryRoot`  |What is the URL of Pact Broker |
|*  `pactbroker.port`  (plugin prop)   *  `stubrunner.properties.pactbroker.port`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_PORT`  (env prop) |Port from URL passed to  `repositoryRoot`  |What is the port of Pact Broker |
|*  `pactbroker.protocol`  (plugin prop)   *  `stubrunner.properties.pactbroker.protocol`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_PROTOCOL`  (env prop) |Protocol from URL passed to  `repositoryRoot`  |What is the protocol of Pact Broker |
|*  `pactbroker.tags`  (plugin prop)   *  `stubrunner.properties.pactbroker.tags`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_TAGS`  (env prop) |Version of the stub, or  `latest`  if version is  `+`  |What tags should be used to fetch the stub |
|*  `pactbroker.auth.scheme`  (plugin prop)   *  `stubrunner.properties.pactbroker.auth.scheme`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_AUTH_SCHEME`  (env prop) | `Basic`  |What kind of authentication should be used to connect to the Pact Broker |
|*  `pactbroker.auth.username`  (plugin prop)   *  `stubrunner.properties.pactbroker.auth.username`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_AUTH_USERNAME`  (env prop) |The username passed to  `contractsRepositoryUsername`  (maven) or  `contractRepository.username`  (gradle) |Username used to connect to the Pact Broker |
|*  `pactbroker.auth.password`  (plugin prop)   *  `stubrunner.properties.pactbroker.auth.password`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_AUTH_PASSWORD`  (env prop) |The password passed to  `contractsRepositoryPassword`  (maven) or  `contractRepository.password`  (gradle) |Password used to connect to the Pact Broker |
|*  `pactbroker.provider-name-with-group-id`  (plugin prop)   *  `stubrunner.properties.pactbroker.provider-name-with-group-id`  (system prop)   *  `STUBRUNNER_PROPERTIES_PACTBROKER_PROVIDER_NAME_WITH_GROUP_ID`  (env prop) |false |When  `true` , the provider name will be a combination of  `groupId:artifactId` . If  `false` , just  `artifactId`  is used |


## 91. Spring Cloud Contract Verifier Setup

You can set up Spring Cloud Contract Verifier in the following ways:

- [As a Gradle project](multi__spring_cloud_contract_verifier_setup.html#gradle-project)

- [As a Maven project](multi__spring_cloud_contract_verifier_setup.html#maven-project)

- [As a Docker project](multi__spring_cloud_contract_verifier_setup.html#docker-project)

## 91.1 Gradle Project

To learn how to set up the Gradle project for Spring Cloud Contract Verifier, read the following sections:

- [Section 91.1.1, “Prerequisites”](multi__spring_cloud_contract_verifier_setup.html#gradle-prerequisites)

- [Section 91.1.2, “Add Gradle Plugin with Dependencies”](multi__spring_cloud_contract_verifier_setup.html#gradle-add-gradle-plugin)

- [Section 91.1.3, “Gradle and Rest Assured 2.0”](multi__spring_cloud_contract_verifier_setup.html#gradle-and-rest-assured)

- [Section 91.1.4, “Snapshot Versions for Gradle”](multi__spring_cloud_contract_verifier_setup.html#gradle-snapshot-versions)

- [Section 91.1.5, “Add stubs”](multi__spring_cloud_contract_verifier_setup.html#gradle-add-stubs)

- [Section 91.1.7, “Default Setup”](multi__spring_cloud_contract_verifier_setup.html#gradle-default-setup)

- [Section 91.1.8, “Configure Plugin”](multi__spring_cloud_contract_verifier_setup.html#gradle-configure-plugin)

- [Section 91.1.9, “Configuration Options”](multi__spring_cloud_contract_verifier_setup.html#gradle-configuration-options)

- [Section 91.1.10, “Single Base Class for All Tests”](multi__spring_cloud_contract_verifier_setup.html#gradle-single-base-class)

- [Section 91.1.11, “Different Base Classes for Contracts”](multi__spring_cloud_contract_verifier_setup.html#gradle-different-base-classes)

- [Section 91.1.12, “Invoking Generated Tests”](multi__spring_cloud_contract_verifier_setup.html#gradle-invoking-generated-tests)

- [Section 91.1.13, “Pushing stubs to SCM”](multi__spring_cloud_contract_verifier_setup.html#gradle-pushing-stubs-to-scm)

- [Section 91.1.14, “Spring Cloud Contract Verifier on the Consumer Side”](multi__spring_cloud_contract_verifier_setup.html#gradle-consumer)

### 91.1.1 Prerequisites

In order to use Spring Cloud Contract Verifier with WireMock, you muse use either a Gradle or a Maven plugin.

> If you want to use Spock in your projects, you must add separately the  `spock-core`  and  `spock-spring`  modules. Check [Spock docs for more information](https://spockframework.github.io/)

### 91.1.2 Add Gradle Plugin with Dependencies

To add a Gradle plugin with dependencies, use code similar to this:

```java
buildscript {
	repositories {
		mavenCentral()
	}
	dependencies {
	    classpath "org.springframework.boot:spring-boot-gradle-plugin:${springboot_version}"
		classpath "org.springframework.cloud:spring-cloud-contract-gradle-plugin:${verifier_version}"
	}
}

apply plugin: 'groovy'
apply plugin: 'spring-cloud-contract'

dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-contract-dependencies:${verifier_version}"
	}
}

dependencies {
	testCompile 'org.codehaus.groovy:groovy-all:2.4.6'
	// example with adding Spock core and Spock Spring
	testCompile 'org.spockframework:spock-core:1.0-groovy-2.4'
	testCompile 'org.spockframework:spock-spring:1.0-groovy-2.4'
	testCompile 'org.springframework.cloud:spring-cloud-starter-contract-verifier'
}
```

### 91.1.3 Gradle and Rest Assured 2.0

By default, Rest Assured 3.x is added to the classpath. However, to use Rest Assured 2.x you can add it to the plugins classpath, as shown here:

```java
buildscript {
	repositories {
		mavenCentral()
	}
	dependencies {
	    classpath "org.springframework.boot:spring-boot-gradle-plugin:${springboot_version}"
		classpath "org.springframework.cloud:spring-cloud-contract-gradle-plugin:${verifier_version}"
		classpath "com.jayway.restassured:rest-assured:2.5.0"
		classpath "com.jayway.restassured:spring-mock-mvc:2.5.0"
	}
}

depenendencies {
// all dependencies
// you can exclude rest-assured from spring-cloud-contract-verifier
testCompile "com.jayway.restassured:rest-assured:2.5.0"
testCompile "com.jayway.restassured:spring-mock-mvc:2.5.0"
}
```

That way, the plugin automatically sees that Rest Assured 2.x is present on the classpath and modifies the imports accordingly.

### 91.1.4 Snapshot Versions for Gradle

Add the additional snapshot repository to your build.gradle to use snapshot versions, which are automatically uploaded after every successful build, as shown here:

```java
buildscript {
	repositories {
		mavenCentral()
		mavenLocal()
		maven { url "http://repo.spring.io/snapshot" }
		maven { url "http://repo.spring.io/milestone" }
		maven { url "http://repo.spring.io/release" }
	}
}
```

### 91.1.5 Add stubs

By default, Spring Cloud Contract Verifier is looking for stubs in the  `src/test/resources/contracts`  directory.

The directory containing stub definitions is treated as a class name, and each stub definition is treated as a single test. Spring Cloud Contract Verifier assumes that it contains at least one level of directories that are to be used as the test class name. If more than one level of nested directories is present, all except the last one is used as the package name. For example, with following structure:

```java
src/test/resources/contracts/myservice/shouldCreateUser.groovy
src/test/resources/contracts/myservice/shouldReturnUser.groovy
```

Spring Cloud Contract Verifier creates a test class named  `defaultBasePackage.MyService`  with two methods:

-  `shouldCreateUser()` 

-  `shouldReturnUser()` 

### 91.1.6 Run the Plugin

The plugin registers itself to be invoked before a  `check`  task. If you want it to be part of your build process, you need to do nothing more. If you just want to generate tests, invoke the  `generateContractTests`  task.

### 91.1.7 Default Setup

The default Gradle Plugin setup creates the following Gradle part of the build (in pseudocode):

```java
contracts {
targetFramework = 'JUNIT'
testMode = 'MockMvc'
generatedTestSourcesDir = project.file("${project.buildDir}/generated-test-sources/contracts")
contractsDslDir = "${project.rootDir}/src/test/resources/contracts"
basePackageForTests = 'org.springframework.cloud.verifier.tests'
stubsOutputDir = project.file("${project.buildDir}/stubs")

// the following properties are used when you want to provide where the JAR with contract lays
contractDependency {
stringNotation = ''
}
contractsPath = ''
contractsWorkOffline = false
contractRepository {
cacheDownloadedContracts(true)
}
}

tasks.create(type: Jar, name: 'verifierStubsJar', dependsOn: 'generateClientStubs') {
baseName = project.name
classifier = contracts.stubsSuffix
from contractVerifier.stubsOutputDir
}

project.artifacts {
archives task
}

tasks.create(type: Copy, name: 'copyContracts') {
from contracts.contractsDslDir
into contracts.stubsOutputDir
}

verifierStubsJar.dependsOn 'copyContracts'

publishing {
publications {
stubs(MavenPublication) {
artifactId project.name
artifact verifierStubsJar
}
}
}
```

### 91.1.8 Configure Plugin

To change the default configuration, add a  `contracts`  snippet to your Gradle config, as shown here:

```java
contracts {
	testMode = 'MockMvc'
	baseClassForTests = 'org.mycompany.tests'
	generatedTestSourcesDir = project.file('src/generatedContract')
}
```

### 91.1.9 Configuration Options

-  **testMode** : Defines the mode for acceptance tests. By default, the mode is MockMvc, which is based on Spring’s MockMvc. It can also be changed to  **JaxRsClient**  or to  **Explicit**  for real HTTP calls.

-  **imports** : Creates an array with imports that should be included in generated tests (for example ['org.myorg.Matchers']). By default, it creates an empty array.

-  **staticImports** : Creates an array with static imports that should be included in generated tests(for example ['org.myorg.Matchers.*']). By default, it creates an empty array.

-  **basePackageForTests** : Specifies the base package for all generated tests. If not set, the value is picked from  `baseClassForTests’s package and from `packageWithBaseClasses` . If neither of these values are set, then the value is set to  `org.springframework.cloud.contract.verifier.tests` .

-  **baseClassForTests** : Creates a base class for all generated tests. By default, if you use Spock classes, the class is  `spock.lang.Specification` .

-  **packageWithBaseClasses** : Defines a package where all the base classes reside. This setting takes precedence over  **baseClassForTests** .

-  **baseClassMappings** : Explicitly maps a contract package to a FQN of a base class. This setting takes precedence over  **packageWithBaseClasses**  and  **baseClassForTests** .

-  **ruleClassForTests** : Specifies a rule that should be added to the generated test classes.

-  **ignoredFiles** : Uses an  `Antmatcher`  to allow defining stub files for which processing should be skipped. By default, it is an empty array.

-  **contractsDslDir** : Specifies the directory containing contracts written using the GroovyDSL. By default, its value is  `$rootDir/src/test/resources/contracts` .

-  **generatedTestSourcesDir** : Specifies the test source directory where tests generated from the Groovy DSL should be placed. By default its value is  `$buildDir/generated-test-sources/contractVerifier` .

-  **stubsOutputDir** : Specifies the directory where the generated WireMock stubs from the Groovy DSL should be placed.

-  **targetFramework** : Specifies the target test framework to be used. Currently, Spock and JUnit are supported with JUnit being the default framework.

-  **contractsProperties** : a map containing properties to be passed to Spring Cloud Contract components. Those properties might be used by e.g. inbuilt or custom Stub Downloaders.

The following properties are used when you want to specify the location of the JAR containing the contracts: *  **contractDependency** : Specifies the Dependency that provides  `groupid:artifactid:version:classifier`  coordinates. You can use the  `contractDependency`  closure to set it up. *  **contractsPath** : Specifies the path to the jar. If contract dependencies are downloaded, the path defaults to  `groupid/artifactid`  where  `groupid`  is slash separated. Otherwise, it scans contracts under the provided directory. *  **contractsMode** : Specifies the mode of downloading contracts (whether the JAR is available offline, remotely etc.) *  **contractsSnapshotCheckSkip** : If set to  `true`  will not assert whether the downloaded stubs / contract JAR was downloaded from a remote location or a local one(only applicable to Maven repos, not Git or Pact). *  **deleteStubsAfterTest** : If set to  `false`  will not remove any downloaded contracts from temporary directories

### 91.1.10 Single Base Class for All Tests

When using Spring Cloud Contract Verifier in default MockMvc, you need to create a base specification for all generated acceptance tests. In this class, you need to point to an endpoint, which should be verified.

```java
abstract class BaseMockMvcSpec extends Specification {

	def setup() {
		RestAssuredMockMvc.standaloneSetup(new PairIdController())
	}

	void isProperCorrelationId(Integer correlationId) {
		assert correlationId == 123456
	}

	void isEmpty(String value) {
		assert value == null
	}

}
```

If you use  `Explicit`  mode, you can use a base class to initialize the whole tested app as you might see in regular integration tests. If you use the  `JAXRSCLIENT`  mode, this base class should also contain a  `protected WebTarget webTarget`  field. Right now, the only option to test the JAX-RS API is to start a web server.

### 91.1.11 Different Base Classes for Contracts

If your base classes differ between contracts, you can tell the Spring Cloud Contract plugin which class should get extended by the autogenerated tests. You have two options:

- Follow a convention by providing the  `packageWithBaseClasses` 

- Provide explicit mapping via  `baseClassMappings` 

**By Convention** 

The convention is such that if you have a contract under (for example)  `src/test/resources/contract/foo/bar/baz/`  and set the value of the  `packageWithBaseClasses`  property to  `com.example.base` , then Spring Cloud Contract Verifier assumes that there is a  `BarBazBase`  class under the  `com.example.base`  package. In other words, the system takes the last two parts of the package, if they exist, and forms a class with a  `Base`  suffix. This rule takes precedence over  **baseClassForTests** . Here is an example of how it works in the  `contracts`  closure:

```java
packageWithBaseClasses = 'com.example.base'
```

**By Mapping** 

You can manually map a regular expression of the contract’s package to fully qualified name of the base class for the matched contract. You have to provide a list called  `baseClassMappings`  that consists  `baseClassMapping`  objects that takes a  `contractPackageRegex`  to  `baseClassFQN`  mapping. Consider the following example:

```java
baseClassForTests = "com.example.FooBase"
baseClassMappings {
	baseClassMapping('.*/com/.*', 'com.example.ComBase')
	baseClassMapping('.*/bar/.*':'com.example.BarBase')
}
```

Let’s assume that you have contracts under -  `src/test/resources/contract/com/`  -  `src/test/resources/contract/foo/` 

By providing the  `baseClassForTests` , we have a fallback in case mapping did not succeed. (You could also provide the  `packageWithBaseClasses`  as a fallback.) That way, the tests generated from  `src/test/resources/contract/com/`  contracts extend the  `com.example.ComBase` , whereas the rest of the tests extend  `com.example.FooBase` .

### 91.1.12 Invoking Generated Tests

To ensure that the provider side is compliant with defined contracts, you need to invoke:

```java
./gradlew generateContractTests test
```

### 91.1.13 Pushing stubs to SCM

If you’re using the SCM repository to keep the contracts and stubs, you might want to automate the step of pushing stubs to the repository. To do that, it’s enough to call the  `pushStubsToScm`  task. Example:

```java
$ ./gradlew pushStubsToScm
```

Under [Section 97.6, “Using the SCM Stub Downloader”](multi__using_the_pluggable_architecture.html#scm-stub-downloader) you can find all possible configuration options that you can pass either via the  `contractsProperties`  field e.g.  `contracts { contractsProperties = [foo:"bar"] }` , via  `contractsProperties`  method e.g.  `contracts { contractsProperties([foo:"bar"]) }` , a system property or an environment variable.

### 91.1.14 Spring Cloud Contract Verifier on the Consumer Side

In a consuming service, you need to configure the Spring Cloud Contract Verifier plugin in exactly the same way as in case of provider. If you do not want to use Stub Runner then you need to copy contracts stored in  `src/test/resources/contracts`  and generate WireMock JSON stubs using:

```java
./gradlew generateClientStubs
```

> The  `stubsOutputDir`  option has to be set for stub generation to work.

When present, JSON stubs can be used in automated tests of consuming a service.

```java
@ContextConfiguration(loader == SpringApplicationContextLoader, classes == Application)
class LoanApplicationServiceSpec extends Specification {

@ClassRule
@Shared
WireMockClassRule wireMockRule == new WireMockClassRule()

@Autowired
LoanApplicationService sut

def 'should successfully apply for loan'() {
given:
	LoanApplication application =
			new LoanApplication(client: new Client(clientPesel: '12345678901'), amount: 123.123)
when:
	LoanApplicationResult loanApplication == sut.loanApplication(application)
then:
	loanApplication.loanApplicationStatus == LoanApplicationStatus.LOAN_APPLIED
	loanApplication.rejectionReason == null
}
}
```

`LoanApplication`  makes a call to  `FraudDetection`  service. This request is handled by a WireMock server configured with stubs generated by Spring Cloud Contract Verifier.

## 91.2 Maven Project

To learn how to set up the Maven project for Spring Cloud Contract Verifier, read the following sections:

- [Section 91.2.1, “Add maven plugin”](multi__spring_cloud_contract_verifier_setup.html#maven-add-plugin)

- [Section 91.2.2, “Maven and Rest Assured 2.0”](multi__spring_cloud_contract_verifier_setup.html#maven-rest-assured)

- [Section 91.2.3, “Snapshot versions for Maven”](multi__spring_cloud_contract_verifier_setup.html#maven-snapshot-versions)

- [Section 91.2.4, “Add stubs”](multi__spring_cloud_contract_verifier_setup.html#maven-add-stubs)

- [Section 91.2.5, “Run plugin”](multi__spring_cloud_contract_verifier_setup.html#maven-run-plugin)

- [Section 91.2.6, “Configure plugin”](multi__spring_cloud_contract_verifier_setup.html#maven-configure-plugin)

- [Section 91.2.7, “Configuration Options”](multi__spring_cloud_contract_verifier_setup.html#maven-configuration-options)

- [Section 91.2.8, “Single Base Class for All Tests”](multi__spring_cloud_contract_verifier_setup.html#maven-single-base)

- [Section 91.2.9, “Different base classes for contracts”](multi__spring_cloud_contract_verifier_setup.html#maven-different-base)

- [Section 91.2.10, “Invoking generated tests”](multi__spring_cloud_contract_verifier_setup.html#maven-invoking-generated-tests)

- [Section 91.2.11, “Pushing stubs to SCM”](multi__spring_cloud_contract_verifier_setup.html#maven-pushing-stubs-to-scm)

- [Section 91.2.12, “Maven Plugin and STS”](multi__spring_cloud_contract_verifier_setup.html#maven-sts)

### 91.2.1 Add maven plugin

Add the Spring Cloud Contract BOM in a fashion similar to this:

```xml
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
```

Next, add the  `Spring Cloud Contract Verifier`  Maven plugin:

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<version>${spring-cloud-contract.version}</version>
	<extensions>true</extensions>
	<configuration>
		<packageWithBaseClasses>com.example.fraud</packageWithBaseClasses>
	</configuration>
</plugin>
```

You can read more in the [Spring Cloud Contract Maven Plugin Documentation (example for 2.0.0.RELEASE version)](https://cloud.spring.io/spring-cloud-static/spring-cloud-contract/2.0.0.RELEASE/spring-cloud-contract-maven-plugin/).

### 91.2.2 Maven and Rest Assured 2.0

By default, Rest Assured 3.x is added to the classpath. However, you can use Rest Assured 2.x by adding it to the plugins classpath, as shown here:

```xml
<plugin>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<version>${spring-cloud-contract.version}</version>
<extensions>true</extensions>
<configuration>
<packageWithBaseClasses>com.example</packageWithBaseClasses>
</configuration>
<dependencies>
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-verifier</artifactId>
<version>${spring-cloud-contract.version}</version>
</dependency>
<dependency>
<groupId>com.jayway.restassured</groupId>
<artifactId>rest-assured</artifactId>
<version>2.5.0</version>
<scope>compile</scope>
</dependency>
<dependency>
<groupId>com.jayway.restassured</groupId>
<artifactId>spring-mock-mvc</artifactId>
<version>2.5.0</version>
<scope>compile</scope>
</dependency>
</dependencies>
</plugin>

<dependencies>
<!-- all dependencies -->
<!-- you can exclude rest-assured from spring-cloud-contract-verifier -->
<dependency>
<groupId>com.jayway.restassured</groupId>
<artifactId>rest-assured</artifactId>
<version>2.5.0</version>
<scope>test</scope>
</dependency>
<dependency>
<groupId>com.jayway.restassured</groupId>
<artifactId>spring-mock-mvc</artifactId>
<version>2.5.0</version>
<scope>test</scope>
</dependency>
</dependencies>
```

That way, the plugin automatically sees that Rest Assured 3.x is present on the classpath and modifies the imports accordingly.

### 91.2.3 Snapshot versions for Maven

For Snapshot and Milestone versions, you have to add the following section to your  `pom.xml` , as shown here:

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

### 91.2.4 Add stubs

By default, Spring Cloud Contract Verifier is looking for stubs in the  `src/test/resources/contracts`  directory. The directory containing stub definitions is treated as a class name, and each stub definition is treated as a single test. We assume that it contains at least one directory to be used as test class name. If there is more than one level of nested directories, all except the last one is used as package name. For example, with following structure:

```java
src/test/resources/contracts/myservice/shouldCreateUser.groovy
src/test/resources/contracts/myservice/shouldReturnUser.groovy
```

Spring Cloud Contract Verifier creates a test class named  `defaultBasePackage.MyService`  with two methods

-  `shouldCreateUser()` 

-  `shouldReturnUser()` 

### 91.2.5 Run plugin

The plugin goal  `generateTests`  is assigned to be invoked in the phase called  `generate-test-sources` . If you want it to be part of your build process, you need not do anything. If you just want to generate tests, invoke the  `generateTests`  goal.

### 91.2.6 Configure plugin

To change the default configuration, just add a  `configuration`  section to the plugin definition or the  `execution`  definition, as shown here:

```xml
<plugin>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<executions>
<execution>
<goals>
<goal>convert</goal>
<goal>generateStubs</goal>
<goal>generateTests</goal>
</goals>
</execution>
</executions>
<configuration>
<basePackageForTests>org.springframework.cloud.verifier.twitter.place</basePackageForTests>
<baseClassForTests>org.springframework.cloud.verifier.twitter.place.BaseMockMvcSpec</baseClassForTests>
</configuration>
</plugin>
```

### 91.2.7 Configuration Options

-  **testMode** : Defines the mode for acceptance tests. By default, the mode is MockMvc, which is based on Spring’s MockMvc. It can also be changed to  **JaxRsClient**  or to  **Explicit**  for real HTTP calls.

-  **basePackageForTests** : Specifies the base package for all generated tests. If not set, the value is picked from  `baseClassForTests’s package and from `packageWithBaseClasses` . If neither of these values are set, then the value is set to  `org.springframework.cloud.contract.verifier.tests` .

-  **ruleClassForTests** : Specifies a rule that should be added to the generated test classes.

-  **baseClassForTests** : Creates a base class for all generated tests. By default, if you use Spock classes, the class is  `spock.lang.Specification` .

-  **contractsDirectory** : Specifies a directory containing contracts written with the GroovyDSL. The default directory is  `/src/test/resources/contracts` .

-  **testFramework** : Specifies the target test framework to be used. Currently, Spock and JUnit are supported with JUnit being the default framework

-  **packageWithBaseClasses** : Defines a package where all the base classes reside. This setting takes precedence over  **baseClassForTests** . The convention is such that, if you have a contract under (for example)  `src/test/resources/contract/foo/bar/baz/`  and set the value of the  `packageWithBaseClasses`  property to  `com.example.base` , then Spring Cloud Contract Verifier assumes that there is a  `BarBazBase`  class under the  `com.example.base`  package. In other words, the system takes the last two parts of the package, if they exist, and forms a class with a  `Base`  suffix.

-  **baseClassMappings** : Specifies a list of base class mappings that provide  `contractPackageRegex` , which is checked against the package where the contract is located, and  `baseClassFQN` , which maps to the fully qualified name of the base class for the matched contract. For example, if you have a contract under  `src/test/resources/contract/foo/bar/baz/`  and map the property  `.* → com.example.base.BaseClass` , then the test class generated from these contracts extends  `com.example.base.BaseClass` . This setting takes precedence over  **packageWithBaseClasses**  and  **baseClassForTests** .

-  **contractsProperties** : a map containing properties to be passed to Spring Cloud Contract components. Those properties might be used by e.g. inbuilt or custom Stub Downloaders.

If you want to download your contract definitions from a Maven repository, you can use the following options:

-  **contractDependency** : The contract dependency that contains all the packaged contracts.

-  **contractsPath** : The path to the concrete contracts in the JAR with packaged contracts. Defaults to  `groupid/artifactid`  where  `gropuid`  is slash separated.

-  **contractsMode** : Picks the mode in which stubs will be found and registered

-  **contractsSnapshotCheckSkip** : If  `true`  then will not assert whether a stub / contract JAR was downloaded from local or remote location

-  **deleteStubsAfterTest** : If set to  `false`  will not remove any downloaded contracts from temporary directories

-  **contractsRepositoryUrl** : URL to a repo with the artifacts that have contracts. If it is not provided, use the current Maven ones.

-  **contractsRepositoryUsername** : The user name to be used to connect to the repo with contracts.

-  **contractsRepositoryPassword** : The password to be used to connect to the repo with contracts.

-  **contractsRepositoryProxyHost** : The proxy host to be used to connect to the repo with contracts.

-  **contractsRepositoryProxyPort** : The proxy port to be used to connect to the repo with contracts.

We cache only non-snapshot, explicitly provided versions (for example  `+`  or  `1.0.0.BUILD-SNAPSHOT`  won’t get cached). By default, this feature is turned on.

### 91.2.8 Single Base Class for All Tests

When using Spring Cloud Contract Verifier in default MockMvc, you need to create a base specification for all generated acceptance tests. In this class, you need to point to an endpoint, which should be verified.

```java
package org.mycompany.tests

import org.mycompany.ExampleSpringController
import com.jayway.restassured.module.mockmvc.RestAssuredMockMvc
import spock.lang.Specification

class MvcSpec extends Specification {
def setup() {
RestAssuredMockMvc.standaloneSetup(new ExampleSpringController())
}
}
```

You can also setup the whole context if necessary.

```java
import io.restassured.module.mockmvc.RestAssuredMockMvc;
import org.junit.Before;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.web.context.WebApplicationContext;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT, classes = SomeConfig.class, properties="some=property")
public abstract class BaseTestClass {

	@Autowired
	WebApplicationContext context;

	@Before
	public void setup() {
		RestAssuredMockMvc.webAppContextSetup(this.context);
	}
}
```

If you use  `EXPLICIT`  mode, you can use a base class to initialize the whole tested app similarly, as you might find in regular integration tests.

```java
import io.restassured.RestAssured;
import org.junit.Before;
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.web.server.LocalServerPort
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.web.context.WebApplicationContext;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT, classes = SomeConfig.class, properties="some=property")
public abstract class BaseTestClass {

	@LocalServerPort
	int port;

	@Before
	public void setup() {
		RestAssured.baseURI = "http://localhost:" + this.port;
	}
}
```

If you use the  `JAXRSCLIENT`  mode, this base class should also contain a  `protected WebTarget webTarget`  field. Right now, the only option to test the JAX-RS API is to start a web server.

### 91.2.9 Different base classes for contracts

If your base classes differ between contracts, you can tell the Spring Cloud Contract plugin which class should get extended by the autogenerated tests. You have two options:

- Follow a convention by providing the  `packageWithBaseClasses` 

- provide explicit mapping via  `baseClassMappings` 

**By Convention** 

The convention is such that if you have a contract under (for example)  `src/test/resources/contract/foo/bar/baz/`  and set the value of the  `packageWithBaseClasses`  property to  `com.example.base` , then Spring Cloud Contract Verifier assumes that there is a  `BarBazBase`  class under the  `com.example.base`  package. In other words, the system takes the last two parts of the package, if they exist, and forms a class with a  `Base`  suffix. This rule takes precedence over  **baseClassForTests** . Here is an example of how it works in the  `contracts`  closure:

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<configuration>
		<packageWithBaseClasses>hello</packageWithBaseClasses>
	</configuration>
</plugin>
```

**By Mapping** 

You can manually map a regular expression of the contract’s package to fully qualified name of the base class for the matched contract. You have to provide a list called  `baseClassMappings`  that consists  `baseClassMapping`  objects that takes a  `contractPackageRegex`  to  `baseClassFQN`  mapping. Consider the following example:

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<configuration>
		<baseClassForTests>com.example.FooBase</baseClassForTests>
		<baseClassMappings>
			<baseClassMapping>
				<contractPackageRegex>.*com.*</contractPackageRegex>
				<baseClassFQN>com.example.TestBase</baseClassFQN>
			</baseClassMapping>
		</baseClassMappings>
	</configuration>
</plugin>
```

Assume that you have contracts under these two locations: *  `src/test/resources/contract/com/`  *  `src/test/resources/contract/foo/` 

By providing the  `baseClassForTests` , we have a fallback in case mapping did not succeed. (You can also provide the  `packageWithBaseClasses`  as a fallback.) That way, the tests generated from  `src/test/resources/contract/com/`  contracts extend the  `com.example.ComBase` , whereas the rest of the tests extend  `com.example.FooBase` .

### 91.2.10 Invoking generated tests

The Spring Cloud Contract Maven Plugin generates verification code in a directory called  `/generated-test-sources/contractVerifier`  and attaches this directory to  `testCompile`  goal.

For Groovy Spock code, use the following:

```xml
<plugin>
	<groupId>org.codehaus.gmavenplus</groupId>
	<artifactId>gmavenplus-plugin</artifactId>
	<version>1.5</version>
	<executions>
		<execution>
			<goals>
				<goal>testCompile</goal>
			</goals>
		</execution>
	</executions>
	<configuration>
		<testSources>
			<testSource>
				<directory>${project.basedir}/src/test/groovy</directory>
				<includes>
					<include>**/*.groovy</include>
				</includes>
			</testSource>
			<testSource>
				<directory>${project.build.directory}/generated-test-sources/contractVerifier</directory>
				<includes>
					<include>**/*.groovy</include>
				</includes>
			</testSource>
		</testSources>
	</configuration>
</plugin>
```

To ensure that provider side is compliant with defined contracts, you need to invoke  `mvn generateTest test` .

### 91.2.11 Pushing stubs to SCM

If you’re using the SCM repository to keep the contracts and stubs, you might want to automate the step of pushing stubs to the repository. To do that, it’s enough to add the  `pushStubsToScm`  goal. Example:

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

Under [Section 97.6, “Using the SCM Stub Downloader”](multi__using_the_pluggable_architecture.html#scm-stub-downloader) you can find all possible configuration options that you can pass either via the  `<configuration><contractProperties>`  map, a system property or an environment variable.

### 91.2.12 Maven Plugin and STS

If you see the following exception while using STS:

![STS Exception](https://raw.githubusercontent.com/spring-cloud/spring-cloud-contract/2.0.x/docs/src/main/asciidoc/images/sts_exception.png)

When you click on the error marker you should see something like this:

```java
plugin:1.1.0.M1:convert:default-convert:process-test-resources) org.apache.maven.plugin.PluginExecutionException: Execution default-convert of goal org.springframework.cloud:spring-
cloud-contract-maven-plugin:1.1.0.M1:convert failed. at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo(DefaultBuildPluginManager.java:145) at
org.eclipse.m2e.core.internal.embedder.MavenImpl.execute(MavenImpl.java:331) at org.eclipse.m2e.core.internal.embedder.MavenImpl$11.call(MavenImpl.java:1362) at
...
org.eclipse.core.internal.jobs.Worker.run(Worker.java:55) Caused by: java.lang.NullPointerException at
org.eclipse.m2e.core.internal.builder.plexusbuildapi.EclipseIncrementalBuildContext.hasDelta(EclipseIncrementalBuildContext.java:53) at
org.sonatype.plexus.build.incremental.ThreadBuildContext.hasDelta(ThreadBuildContext.java:59) at
```

In order to fix this issue, provide the following section in your  `pom.xml` :

```xml
<build>
<pluginManagement>
<plugins>
<!--This plugin's configuration is used to store Eclipse m2e settings
only. It has no influence on the Maven build itself. -->
<plugin>
<groupId>org.eclipse.m2e</groupId>
<artifactId>lifecycle-mapping</artifactId>
<version>1.0.0</version>
<configuration>
<lifecycleMappingMetadata>
<pluginExecutions>
<pluginExecution>
<pluginExecutionFilter>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-contract-maven-plugin</artifactId>
<versionRange>[1.0,)</versionRange>
<goals>
<goal>convert</goal>
</goals>
</pluginExecutionFilter>
<action>
<execute />
</action>
</pluginExecution>
</pluginExecutions>
</lifecycleMappingMetadata>
</configuration>
</plugin>
</plugins>
</pluginManagement>
</build>
```

## 91.3 Stubs and Transitive Dependencies

The Maven and Gradle plugin that add the tasks that create the stubs jar for you. One problem that arises is that, when reusing the stubs, you can mistakenly import all of that stub’s dependencies. When building a Maven artifact, even though you have a couple of different jars, all of them share one pom:

```java
├── github-webhook-0.0.1.BUILD-20160903.075506-1-stubs.jar
├── github-webhook-0.0.1.BUILD-20160903.075506-1-stubs.jar.sha1
├── github-webhook-0.0.1.BUILD-20160903.075655-2-stubs.jar
├── github-webhook-0.0.1.BUILD-20160903.075655-2-stubs.jar.sha1
├── github-webhook-0.0.1.BUILD-SNAPSHOT.jar
├── github-webhook-0.0.1.BUILD-SNAPSHOT.pom
├── github-webhook-0.0.1.BUILD-SNAPSHOT-stubs.jar
├── ...
└── ...
```

There are three possibilities of working with those dependencies so as not to have any issues with transitive dependencies:

- Mark all application dependencies as optional

- Create a separate artifactid for the stubs

- Exclude dependencies on the consumer side

**Mark all application dependencies as optional** 

If, in the  `github-webhook`  application, you mark all of your dependencies as optional, when you include the  `github-webhook`  stubs in another application (or when that dependency gets downloaded by Stub Runner) then, since all of the dependencies are optional, they will not get downloaded.

**Create a separate artifactid for the stubs** 

If you create a separate  `artifactid` , then you can set it up in whatever way you wish. For example, you might decide to have no dependencies at all.

**Exclude dependencies on the consumer side** 

As a consumer, if you add the stub dependency to your classpath, you can explicitly exclude the unwanted dependencies.

## 91.4 CI Server setup

When fetching stubs / contracts in a CI, shared environment, what might happen is that both the producer and the consumer reuse the same local Maven repository. Due to this, the framework, responsible for downloading a stub JAR from remote location, can’t decide which JAR should be picked, local or remote one. That caused the  `"The artifact was found in the local repository but you have explicitly stated that it should be downloaded from a remote one"`  exception and failed the build.

For such cases we’re introducing the property and plugin setup mechanism:

- via  `stubrunner.snapshot-check-skip`  system property

- via  `STUBRUNNER_SNAPSHOT_CHECK_SKIP`  environment variable

if either of these values is set to  `true` , then the stub downloader will not verify the origin of the downloaded JAR.

For the plugins you need to set the  `contractsSnapshotCheckSkip`  property to  `true` .

## 91.5 Scenarios

You can handle scenarios with Spring Cloud Contract Verifier. All you need to do is to stick to the proper naming convention while creating your contracts. The convention requires including an order number followed by an underscore. This will work regardles of whether you’re working with YAML or Groovy. Example:

```java
my_contracts_dir\
scenario1\
1_login.groovy
2_showCart.groovy
3_logout.groovy
```

Such a tree causes Spring Cloud Contract Verifier to generate WireMock’s scenario with a name of  `scenario1`  and the three following steps:

login marked as Started pointing to… showCart marked as Step1 pointing to… logout marked as Step2 which will close the scenario.

More details about WireMock scenarios can be found at [http://wiremock.org/docs/stateful-behaviour/](http://wiremock.org/docs/stateful-behaviour/)

Spring Cloud Contract Verifier also generates tests with a guaranteed order of execution.

## 91.6 Docker Project

We’re publishing a  `springcloud/spring-cloud-contract`  Docker image that contains a project that will generate tests and execute them in  `EXPLICIT`  mode against a running application.

> The  `EXPLICIT`  mode means that the tests generated from contracts will send real requests and not the mocked ones.

### 91.6.1 Short intro to Maven, JARs and Binary storage

Since the Docker image can be used by non JVM projects, it’s good to explain the basic terms behind Spring Cloud Contract packaging defaults.

Part of the following definitions were taken from the [Maven Glossary](https://maven.apache.org/glossary.html)

-  `Project` : Maven thinks in terms of projects. Everything that you will build are projects. Those projects follow a well defined “Project Object Model”. Projects can depend on other projects, in which case the latter are called “dependencies”. A project may consistent of several subprojects, however these subprojects are still treated equally as projects.

-  `Artifact` : An artifact is something that is either produced or used by a project. Examples of artifacts produced by Maven for a project include: JARs, source and binary distributions. Each artifact is uniquely identified by a group id and an artifact ID which is unique within a group.

-  `JAR` : JAR stands for Java ARchive. It’s a format based on the ZIP file format. Spring Cloud Contract packages the contracts and generated stubs in a JAR file.

-  `GroupId` : A group ID is a universally unique identifier for a project. While this is often just the project name (eg. commons-collections), it is helpful to use a fully-qualified package name to distinguish it from other projects with a similar name (eg. org.apache.maven). Typically, when published to the Artifact Manager, the  `GroupId`  will get slash separated and form part of the URL. E.g. for group id  `com.example`  and artifact id  `application`  would be  `/com/example/application/` .

-  `Classifier` : The Maven dependency notation looks as follows:  `groupId:artifactId:version:classifier` . The classifier is additional suffix passed to the dependency. E.g.  `stubs` ,  `sources` . The same dependency e.g.  `com.example:application`  can produce multiple artifacts that differ from each other with the classifier.

-  `Artifact manager` : When you generate binaries / sources / packages, you would like them to be available for others to download / reference or reuse. In case of the JVM world those artifacts would be JARs, for Ruby these are gems and for Docker those would be Docker images. You can store those artifacts in a manager. Examples of such managers can be [Artifactory](https://jfrog.com/artifactory/) or [Nexus](http://www.sonatype.org/nexus/).

### 91.6.2 How it works

The image searches for contracts under the  `/contracts`  folder. The output from running the tests will be available under  `/spring-cloud-contract/build`  folder (it’s useful for debugging purposes).

It’s enough for you to mount your contracts, pass the environment variables and the image will:

- generate the contract tests

- execute the tests against the provided URL

- generate the [WireMock](http://wiremock.org) stubs

- (optional - turned on by default) publish the stubs to a Artifact Manager

#### Environment Variables

The Docker image requires some environment variables to point to your running application, to the Artifact manager instance etc.

-  `PROJECT_GROUP`  - your project’s group id. Defaults to  `com.example` 

-  `PROJECT_VERSION`  - your project’s version. Defaults to  `0.0.1-SNAPSHOT` 

-  `PROJECT_NAME`  - artifact id. Defaults to  `example` 

-  `REPO_WITH_BINARIES_URL`  - URL of your Artifact Manager. Defaults to  `http://localhost:8081/artifactory/libs-release-local`  which is the default URL of [Artifactory](https://jfrog.com/artifactory/) running locally

-  `REPO_WITH_BINARIES_USERNAME`  - (optional) username when the Artifact Manager is secured

-  `REPO_WITH_BINARIES_PASSWORD`  - (optional) password when the Artifact Manager is secured

-  `PUBLISH_ARTIFACTS`  - if set to  `true`  then will publish artifact to binary storage. Defaults to  `true` .

These environment variables are used when contracts lay in an external repository. To enable this feature you must set the  `EXTERNAL_CONTRACTS_ARTIFACT_ID`  environment variable.

-  `EXTERNAL_CONTRACTS_GROUP_ID`  - group id of the project with contracts. Defaults to  `com.example` 

-  `EXTERNAL_CONTRACTS_ARTIFACT_ID` - artifact id of the project with contracts.

-  `EXTERNAL_CONTRACTS_CLASSIFIER` - classifier of the project with contracts. Empty by default

-  `EXTERNAL_CONTRACTS_VERSION`  - version of the project with contracts. Defaults to  `+` , equivalent to picking the latest

-  `EXTERNAL_CONTRACTS_REPO_WITH_BINARIES_URL`  - URL of your Artifact Manager. Defaults to value of  `REPO_WITH_BINARIES_URL`  env var. If that’s not set, defaults to  `http://localhost:8081/artifactory/libs-release-local`  which is the default URL of [Artifactory](https://jfrog.com/artifactory/) running locally

-  `EXTERNAL_CONTRACTS_PATH`  - path to contracts for the given project, inside the project with contracts. Defaults to slash separated  `EXTERNAL_CONTRACTS_GROUP_ID`  concatenated with  `/`  and  `EXTERNAL_CONTRACTS_ARTIFACT_ID` . E.g. for group id  `foo.bar`  and artifact id  `baz` , would result in  `foo/bar/baz`  contracts path.

-  `EXTERNAL_CONTRACTS_WORK_OFFLINE`  - if set to  `true`  then will retrieve artifact with contracts from the container’s  `.m2` . Mount your local  `.m2`  as a volume available at the container’s  `/root/.m2`  path. You must not set both  `EXTERNAL_CONTRACTS_WORK_OFFLINE`  and  `EXTERNAL_CONTRACTS_REPO_WITH_BINARIES_URL` .

These environment variables are used when tests are executed:

-  `APPLICATION_BASE_URL`  - url against which tests should be executed. Remember that it has to be accessible from the Docker container (e.g.  `localhost`  will not work)

-  `APPLICATION_USERNAME`  - (optional) username for basic authentication to your application

-  `APPLICATION_PASSWORD`  - (optional) password for basic authentication to your application

### 91.6.3 Example of usage

Let’s take a look at a simple MVC application

```java
$ git clone https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs
$ cd bookstore
```

The contracts are available under  `/contracts`  folder.

### 91.6.4 Server side (nodejs)

Since we want to run tests, we could just execute:

```java
$ npm test
```

however, for learning purposes, let’s split it into pieces:

```java
# Stop docker infra (nodejs, artifactory)
$ ./stop_infra.sh
# Start docker infra (nodejs, artifactory)
$ ./setup_infra.sh

# Kill & Run app
$ pkill -f "node app"
$ nohup node app &

# Prepare environment variables
$ SC_CONTRACT_DOCKER_VERSION="..."
$ APP_IP="192.168.0.100"
$ APP_PORT="3000"
$ ARTIFACTORY_PORT="8081"
$ APPLICATION_BASE_URL="http://${APP_IP}:${APP_PORT}"
$ ARTIFACTORY_URL="http://${APP_IP}:${ARTIFACTORY_PORT}/artifactory/libs-release-local"
$ CURRENT_DIR="$( pwd )"
$ CURRENT_FOLDER_NAME=${PWD##*/}
$ PROJECT_VERSION="0.0.1.RELEASE"

# Execute contract tests
$ docker run  --rm -e "APPLICATION_BASE_URL=${APPLICATION_BASE_URL}" -e "PUBLISH_ARTIFACTS=true" -e "PROJECT_NAME=${CURRENT_FOLDER_NAME}" -e "REPO_WITH_BINARIES_URL=${ARTIFACTORY_URL}" -e "PROJECT_VERSION=${PROJECT_VERSION}" -v "${CURRENT_DIR}/contracts/:/contracts:ro" -v "${CURRENT_DIR}/node_modules/spring-cloud-contract/output:/spring-cloud-contract-output/" springcloud/spring-cloud-contract:"${SC_CONTRACT_DOCKER_VERSION}"

# Kill app
$ pkill -f "node app"
```

What will happen is that via bash scripts:

- infrastructure will be set up (MongoDb, Artifactory). In real life scenario you would just run the NodeJS application with mocked database. In this example we want to show how we can benefit from Spring Cloud Contract in no time.

- due to those constraints the contracts also represent the stateful situation

- first request is a  `POST`  that causes data to get inserted to the database

- second request is a  `GET`  that returns a list of data with 1 previously inserted element

- the NodeJS application will be started (on port  `3000` )

- contract tests will be generated via Docker and tests will be executed against the running application

- the contracts will be taken from  `/contracts`  folder.

- the output of the test execution is available under  `node_modules/spring-cloud-contract/output` .

- the stubs will be uploaded to Artifactory. You can check them out under [http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/](http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/) . The stubs will be here [http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/bookstore-0.0.1.RELEASE-stubs.jar](http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/bookstore-0.0.1.RELEASE-stubs.jar).

To see how the client side looks like check out the [Section 93.9, “Stub Runner Docker”](multi__spring_cloud_contract_stub_runner.html#stubrunner-docker) section.


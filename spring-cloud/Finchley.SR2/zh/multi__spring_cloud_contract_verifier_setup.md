## 91. Spring CloudContract验证程序设置

您可以通过以下方式设置Spring Cloud Contract Verifier：

- [As a Gradle project](multi__spring_cloud_contract_verifier_setup.html#gradle-project)

- [As a Maven project](multi__spring_cloud_contract_verifier_setup.html#maven-project)

- [As a Docker project](multi__spring_cloud_contract_verifier_setup.html#docker-project)

## 91.1 Gradle项目

要了解如何为Spring Cloud Contract Verifier设置Gradle项目，请阅读以下部分：

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

### 91.1.1先决条件

为了将Spring Cloud Contract Verifier与WireMock一起使用，您可以使用Gradle或Maven插件.

> 如果要在项目中使用Spock，则必须单独添加 `spock-core` 和 `spock-spring` 模块.检查[Spock docs for more information](https://spockframework.github.io/)

### 91.1.2添加具有依赖关系的Gradle插件

要添加具有依赖项的Gradle插件，请使用与此类似的代码：

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

默认情况下，Rest Assured 3.x将添加到类路径中.但是，要使用Rest Assured 2.x，您可以将其添加到插件类路径中，如下所示：

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

这样，插件会自动看到Rest Assured 2.x出现在类路径上并相应地修改导入.

### 91.1.4 Gradle的快照版本

将其他快照存储库添加到build.gradle以使用快照版本，这些版本会在每次成功构建后自动上载，如下所示：

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

### 91.1.5添加存根

默认情况下，Spring Cloud Contract Verifier在 `src/test/resources/contracts` 目录中查找存根.

包含存根定义的目录被视为类名，每个存根定义都被视为单个测试. Spring Cloud Contract Verifier假定它至少包含一个级别的目录，这些目录将用作测试类名称.如果存在多个级别的嵌套目录，则除最后一个目录之外的所有目录都将用作包名称.例如，具有以下结构：

```java
src/test/resources/contracts/myservice/shouldCreateUser.groovy
src/test/resources/contracts/myservice/shouldReturnUser.groovy
```

Spring Cloud Contract Verifier使用两种方法创建名为 `defaultBasePackage.MyService` 的测试类：

-  `shouldCreateUser()` 

-  `shouldReturnUser()` 

### 91.1.6运行插件

插件注册自己在 `check` 任务之前调用.如果您希望它成为构建过程的一部分，则无需执行任何操作.如果您只想生成测试，请调用 `generateContractTests` 任务.

### 91.1.7默认设置

默认的Gradle Plugin设置会创建构建的以下Gradle部分（伪代码）：

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

### 91.1.8配置插件

要更改默认配置，请在Gradle配置中添加 `contracts` 代码段，如下所示：

```java
contracts {
	testMode = 'MockMvc'
	baseClassForTests = 'org.mycompany.tests'
	generatedTestSourcesDir = project.file('src/generatedContract')
}
```

### 91.1.9配置选项

-  **testMode** ：定义验收测试的模式.默认情况下，模式是MockMvc，它基于Spring的MockMvc.对于真正的HTTP调用，它也可以更改为 **JaxRsClient** 或 **Explicit** .

-  **imports** ：创建一个数组，其中包含应包含在生成的测试中的导入（例如['org.myorg.Matchers']）.默认情况下，它会创建一个空数组.

-  **staticImports** ：使用应包含在生成的测试中的静态导入创建一个数组（例如['org.myorg.Matchers.*']）.默认情况下，它会创建一个空数组.

-  **basePackageForTests** ：指定所有生成的测试的基础包.如果未设置，则从 `baseClassForTests’s package and from `packageWithBaseClasses` 中选择值.如果这两个值均未设置，则该值设置为 `org.springframework.cloud.contract.verifier.tests` .

-  **baseClassForTests** ：为所有生成的测试创建基类.默认情况下，如果使用Spock类，则该类为 `spock.lang.Specification` .

-  **packageWithBaseClasses** ：定义所有基类所在的包.此设置优先于 **baseClassForTests** .

-  **baseClassMappings** ：将Contract包显式映射到基类的FQN.此设置优先于 **packageWithBaseClasses** 和 **baseClassForTests** .

-  **ruleClassForTests** ：指定应添加到生成的测试类的规则.

-  **ignoredFiles** ：使用 `Antmatcher` 允许定义应跳过处理的存根文件.默认情况下，它是一个空数组.

-  **contractsDslDir** ：指定包含使用GroovyDSL编写的协定的目录.默认情况下，其值为 `$rootDir/src/test/resources/contracts` .

-  **generatedTestSourcesDir** ：指定应放置从Groovy DSL生成的测试的测试源目录.默认情况下，其值为 `$buildDir/generated-test-sources/contractVerifier` .

-  **stubsOutputDir** ：指定应放置从Groovy DSL生成的WireMock存根的目录.

-  **targetFramework** ：指定要使用的目标测试框架.目前，支持Spock和JUnit，JUnit是默认框架.

-  **contractsProperties** ：包含要传递给Spring Cloud Contract组件的属性的映射.这些属性可以用于例如内置或自定义Stub下载器.

如果要指定包含Contract的JAR的位置，则使用以下属性：*  **contractDependency** ：指定提供 `groupid:artifactid:version:classifier` 坐标的依赖关系.您可以使用 `contractDependency` 闭包进行设置. *  **contractsPath** ：指定jar的路径.如果下载了Contract依赖项，则路径默认为 `groupid/artifactid` ，其中 `groupid` 是斜杠分隔.否则，它会扫描提供的目录下的Contract. *  **contractsMode** ：指定下载Contract的模式（JAR是否可脱机，远程等可用）*  **contractsSnapshotCheckSkip** ：如果设置为 `true` ，则不会断言下载的存根/ContractJAR是从远程位置还是从本地下载的（仅限）适用于Maven回购，而不是Git或Pact）. *  **deleteStubsAfterTest** ：如果设置为 `false` 将不会从临时目录中删除任何下载的Contract

### 91.1.10所有测试的单基类

在默认MockMvc中使用Spring Cloud Contract Verifier时，您需要为所有生成的验收测试创建基本规范.在此类中，您需要指向应验证的endpoints.

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

如果使用 `Explicit` 模式，则可以使用基类初始化整个测试的应用程序，就像在常规集成测试中看到的那样.如果使用 `JAXRSCLIENT` 模式，则此基类还应包含 `protected WebTarget webTarget` 字段.现在，测试JAX-RS API的唯一选择是启动Web服务器.

### 91.1.11Contract的不同基本类别

如果您的基类在Contract之间有所不同，您可以告诉Spring Cloud Contract插件哪个类应该通过自动生成的测试进行扩展.你有两个选择：

- 通过提供 `packageWithBaseClasses` 遵循惯例

- Provide显式映射 `baseClassMappings` 

**By Convention** 

如果您在（例如） `src/test/resources/contract/foo/bar/baz/` 下签订Contract并将 `packageWithBaseClasses` 属性的值设置为 `com.example.base` ，则Spring Cloud Contract Verifier会假定 `com.example.base` 包下有 `BarBazBase` 类.换句话说，系统采用包的最后两部分（如果存在），并形成一个带有 `Base` 后缀的类.此规则优先于 **baseClassForTests** .以下是它在 `contracts` 闭包中如何工作的示例：

```java
packageWithBaseClasses = 'com.example.base'
```

**By Mapping** 

您可以手动将Contract包的正则表达式映射到匹配Contract的基类的完全限定名称.您必须提供一个名为 `baseClassMappings` 的列表，其中包含 `baseClassMapping` 个对象，这些对象采用 `contractPackageRegex` 到 `baseClassFQN` 映射.请考虑以下示例：

```java
baseClassForTests = "com.example.FooBase"
baseClassMappings {
	baseClassMapping('.*/com/.*', 'com.example.ComBase')
	baseClassMapping('.*/bar/.*':'com.example.BarBase')
}
```

我们假设你有Contract -   `src/test/resources/contract/com/`   -   `src/test/resources/contract/foo/` 

通过提供 `baseClassForTests` ，我们在案例映射未成功的情况下有后备. （您也可以提供 `packageWithBaseClasses` 作为后备.）这样，从 `src/test/resources/contract/com/` Contract生成的测试扩展 `com.example.ComBase` ，而其余测试扩展 `com.example.FooBase` .

### 91.1.12调用生成的测试

要确保提供者方符合已定义的Contract，您需要调用：

```java
./gradlew generateContractTests test
```

### 91.1.13将存根推送到SCM

如果您正在使用SCM存储库保留Contract和存根，您可能希望自动执行将存根推送到存储库的步骤.要做到这一点，就足以调用 `pushStubsToScm` 任务了.例：

```java
$ ./gradlew pushStubsToScm
```

在[Section 97.6, “Using the SCM Stub Downloader”](multi__using_the_pluggable_architecture.html#scm-stub-downloader)下，您可以找到所有可以通过 `contractsProperties` 字段传递的配置选项，例如 `contracts { contractsProperties = [foo:"bar"] }` ，通过 `contractsProperties` 方法，例如 `contracts { contractsProperties([foo:"bar"]) }` ，系统属性或环境变量.

### 91.1.14消费者方面的Spring CloudContract验证者

在消费服务中，您需要以与提供者完全相同的方式配置Spring Cloud Contract Verifier插件.如果您不想使用Stub Runner，那么您需要复制存储在 `src/test/resources/contracts` 中的Contract并使用以下方法生成WireMock JSON存根：

```java
./gradlew generateClientStubs
```

> 必须设置 `stubsOutputDir` 选项才能使存根生成生效.

如果存在，JSON存根可用于使用服务的自动化测试.

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

`LoanApplication` 拨打 `FraudDetection` 服务电话.此请求由配置有Spring Cloud Contract Verifier生成的存根的WireMock服务器处理.

## 91.2 Maven项目

要了解如何为Spring Cloud Contract Verifier设置Maven项目，请阅读以下部分：

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

### 91.2.1添加maven插件

以类似于此的方式添加Spring Cloud Contract BOM：

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

接下来，添加 `Spring Cloud Contract Verifier`  Maven插件：

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

您可以在[Spring Cloud Contract Maven Plugin Documentation (example for 2.0.0.RELEASE version)](https://cloud.spring.io/spring-cloud-static/spring-cloud-contract/2.0.0.RELEASE/spring-cloud-contract-maven-plugin/)中阅读更多内容.

### 91.2.2 Maven and Rest Assured 2.0

默认情况下，Rest Assured 3.x将添加到类路径中.但是，您可以将Rest Assured 2.x添加到插件类路径中，如下所示：

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

这样，插件会自动看到Rest Assured 3.x出现在类路径上并相应地修改导入.

### 91.2.3 Maven的快照版本

对于Snapshot和Milestone版本，您必须将以下部分添加到 `pom.xml` ，如下所示：

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

### 91.2.4添加存根

默认情况下，Spring Cloud Contract Verifier在 `src/test/resources/contracts` 目录中查找存根.包含存根定义的目录被视为类名，每个存根定义都被视为单个测试.我们假设它至少包含一个用作测试类名的目录.如果有多个级别的嵌套目录，则除最后一个目录之外的所有目录都将用作包名称.例如，具有以下结构：

```java
src/test/resources/contracts/myservice/shouldCreateUser.groovy
src/test/resources/contracts/myservice/shouldReturnUser.groovy
```

Spring Cloud Contract Verifier使用两种方法创建名为 `defaultBasePackage.MyService` 的测试类

-  `shouldCreateUser()` 

-  `shouldReturnUser()` 

### 91.2.5运行插件

插件目标 `generateTests` 被指定在名为 `generate-test-sources` 的阶段中调用.如果您希望它成为构建过程的一部分，则无需执行任何操作.如果您只想生成测试，请调用 `generateTests` 目标.

### 91.2.6配置插件

要更改默认配置，只需将 `configuration` 部分添加到插件定义或 `execution` 定义中，如下所示：

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

### 91.2.7配置选项

-  **testMode** ：定义验收测试的模式.默认情况下，模式是MockMvc，它基于Spring的MockMvc.对于真正的HTTP调用，它也可以更改为 **JaxRsClient** 或 **Explicit** .

-  **basePackageForTests** ：指定所有生成的测试的基础包.如果未设置，则从 `baseClassForTests’s package and from `packageWithBaseClasses` 中选择该值.如果这两个值均未设置，则该值设置为 `org.springframework.cloud.contract.verifier.tests` .

-  **ruleClassForTests** ：指定应添加到生成的测试类的规则.

-  **baseClassForTests** ：为所有生成的测试创建基类.默认情况下，如果使用Spock类，则该类为 `spock.lang.Specification` .

-  **contractsDirectory** ：指定包含使用GroovyDSL编写的协定的目录.默认目录是 `/src/test/resources/contracts` .

-  **testFramework** ：指定要使用的目标测试框架.目前，支持Spock和JUnit，JUnit是默认框架

-  **packageWithBaseClasses** ：定义所有基类所在的包.此设置优先于 **baseClassForTests** .该约定是这样的，如果您在（例如） `src/test/resources/contract/foo/bar/baz/` 下签订Contract并将 `packageWithBaseClasses` 属性的值设置为 `com.example.base` ，则Spring Cloud Contract Verifier会假定 `com.example.base` 包下有 `BarBazBase` 类.换句话说，系统采用包的最后两部分（如果存在），并形成一个带有 `Base` 后缀的类.

-  **baseClassMappings** ：指定提供 `contractPackageRegex` 的基类映射列表，该列表根据Contract所在的包进行检查，并且 `baseClassFQN` ，映射到匹配Contract的基类的完全限定名称.例如，如果您在 `src/test/resources/contract/foo/bar/baz/` 下签订Contract并映射属性 `.* → com.example.base.BaseClass` ，那么从这些Contract生成的测试类将扩展 `com.example.base.BaseClass` .此设置优先于 **packageWithBaseClasses** 和 **baseClassForTests** .

-  **contractsProperties** ：包含要传递给Spring Cloud Contract组件的属性的映射.这些属性可以用于例如内置或自定义Stub下载器.

如果你想下载从Maven存储库中获取Contract定义，您可以使用以下选项：

-  **contractDependency** ：包含所有打包Contract的Contract依赖项.

-  **contractsPath** ：包装Contract中JAR具体Contract的路径.默认为 `groupid/artifactid` ，其中 `gropuid` 是斜杠分隔.

-  **contractsMode** ：选择将找到并注册存根的模式

-  **contractsSnapshotCheckSkip** ：如果 `true` 则不会断言是否从本地或远程位置下载了存根/合约JAR

-  **deleteStubsAfterTest** ：如果设置为 `false` ，则不会从临时目录中删除任何下载的Contract

-  **contractsRepositoryUrl** ：具有Contract的工件的repo的URL.如果未提供，请使用当前的Maven.

-  **contractsRepositoryUsername** ：用于通过Contract连接到仓库的用户名.

-  **contractsRepositoryPassword** ：用于通过Contract连接到仓库的密码.

-  **contractsRepositoryProxyHost** ：用于通过Contract连接到仓库的代理主机.

-  **contractsRepositoryProxyPort** ：用于通过Contract连接到仓库的代理端口.

我们只缓存非快照，显式提供的版本（例如 `+` 或 `1.0.0.BUILD-SNAPSHOT` 不会被缓存）.默认情况下，此功能已打开.

### 91.2.8所有测试的单基类

在默认MockMvc中使用Spring Cloud Contract Verifier时，您需要为所有生成的验收测试创建基本规范.在此类中，您需要指向应验证的endpoints.

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

如有必要，您还可以设置整个上下文.

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

如果使用 `EXPLICIT` 模式，则可以使用基类来初始化整个测试的应用程序，就像在常规集成测试中所看到的那样.

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

如果使用 `JAXRSCLIENT` 模式，则此基类还应包含 `protected WebTarget webTarget` 字段.现在，测试JAX-RS API的唯一选择是启动Web服务器.

### 91.2.9Contract的不同基类

如果您的基类在Contract之间有所不同，您可以告诉Spring Cloud Contract插件哪个类应该通过自动生成的测试进行扩展.你有两个选择：

- 通过提供 `packageWithBaseClasses` 遵循惯例

- 通过 `baseClassMappings` 提供显式映射

**By Convention** 

如果您在（例如） `src/test/resources/contract/foo/bar/baz/` 下签订Contract并将 `packageWithBaseClasses` 属性的值设置为 `com.example.base` ，则Spring Cloud Contract Verifier会假定 `com.example.base` 包下有 `BarBazBase` 类.换句话说，系统采用包的最后两部分（如果存在），并形成一个带有 `Base` 后缀的类.此规则优先于 **baseClassForTests** .以下是它在 `contracts` 闭包中如何工作的示例：

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

您可以手动将Contract包的正则表达式映射到匹配Contract的基类的完全限定名称.您必须提供一个名为 `baseClassMappings` 的列表，其中包含 `baseClassMapping` 个对象，这些对象采用 `contractPackageRegex` 到 `baseClassFQN` 映射.请考虑以下示例：

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

假设您在这两个地点下有Contract：*  `src/test/resources/contract/com/`  *  `src/test/resources/contract/foo/` 

通过提供 `baseClassForTests` ，我们在案例映射未成功的情况下进行了回退. （您也可以将 `packageWithBaseClasses` 作为后备提供.）这样，从 `src/test/resources/contract/com/` Contract生成的测试会扩展 `com.example.ComBase` ，而其余测试则扩展 `com.example.FooBase` .

### 91.2.10调用生成的测试

Spring Cloud Contract Maven插件在名为 `/generated-test-sources/contractVerifier` 的目录中生成验证码，并将此目录附加到 `testCompile` 目标.

对于Groovy Spock代码，请使用以下代码：

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

要确保提供者方符合已定义的Contract，您需要调用 `mvn generateTest test` .

### 91.2.11将存根推送到SCM

如果您使用SCM存储库来保留Contract和存根，则可能需要自动执行将存根推送到存储库的步骤.要做到这一点，就足以添加 `pushStubsToScm` 目标了.例：

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

在[Section 97.6, “Using the SCM Stub Downloader”](multi__using_the_pluggable_architecture.html#scm-stub-downloader)下，您可以找到所有可以通过 `<configuration><contractProperties>` 映射，系统属性或环境变量传递的配置选项.

### 91.2.12 Maven插件和STS

如果在使用STS时看到以下异常：

![STS Exception](https://raw.githubusercontent.com/spring-cloud/spring-cloud-contract/2.0.x/docs/src/main/asciidoc/images/sts_exception.png)

当您单击错误标记时，您应该看到如下内容：

```java
plugin:1.1.0.M1:convert:default-convert:process-test-resources) org.apache.maven.plugin.PluginExecutionException: Execution default-convert of goal org.springframework.cloud:spring-
cloud-contract-maven-plugin:1.1.0.M1:convert failed. at org.apache.maven.plugin.DefaultBuildPluginManager.executeMojo(DefaultBuildPluginManager.java:145) at
org.eclipse.m2e.core.internal.embedder.MavenImpl.execute(MavenImpl.java:331) at org.eclipse.m2e.core.internal.embedder.MavenImpl$11.call(MavenImpl.java:1362) at
...
org.eclipse.core.internal.jobs.Worker.run(Worker.java:55) Caused by: java.lang.NullPointerException at
org.eclipse.m2e.core.internal.builder.plexusbuildapi.EclipseIncrementalBuildContext.hasDelta(EclipseIncrementalBuildContext.java:53) at
org.sonatype.plexus.build.incremental.ThreadBuildContext.hasDelta(ThreadBuildContext.java:59) at
```

要解决此问题，请在 `pom.xml` 中提供以下部分：

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

## 91.3存根和传递依赖关系

Maven和Gradle插件，为您添加创建存根jar的任务.出现的一个问题是，在重用存根时，您可能会错误地导入所有存根的依赖关系.在构建Maven工件时，即使你有几个不同的jar，它们都共用一个pom：

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

使用这些依赖项有三种可能性，以免对传递依赖项产生任何问题：

- 将所有应用程序依赖项标记为可选

- 为存根创建单独的artifactid

- 在消费者方面排除依赖关系

**Mark all application dependencies as optional** 

如果在 `github-webhook` 应用程序中将所有依赖项标记为可选，则在包含 `github-webhook` 存根时在另一个应用程序中（或者当Stub Runner下载该依赖项时），因为所有依赖项都是可选的，所以它们不会被下载.

**Create a separate artifactid for the stubs** 

如果您创建单独的 `artifactid` ，则可以按照您希望的任何方式进行设置.例如，您可能决定根本没有依赖关系.

**Exclude dependencies on the consumer side** 

作为使用者，如果将存根依赖关系添加到类路径，则可以显式排除不需要的依赖关系.

## 91.4 CI服务器设置

在CI，共享环境中获取存根/Contract时，可能发生的情况是生产环境者和使用者都重用相同的本地Maven存储库.因此，负责从远程位置下载存根JAR的框架无法决定应该选择哪个JAR，本地或远程JAR.这导致 `"The artifact was found in the local repository but you have explicitly stated that it should be downloaded from a remote one"` 异常并且构建失败.

对于这种情况，我们引入了属性和插件设置机制：

- via  `stubrunner.snapshot-check-skip` 系统属性

- via  `STUBRUNNER_SNAPSHOT_CHECK_SKIP` 环境变量

如果这些值中的任何一个设置为 `true` ，则存根下载器将不会验证下载的JAR的来源.

对于插件，您需要将 `contractsSnapshotCheckSkip` 属性设置为 `true` .

## 91.5场景

您可以使用Spring Cloud Contract Verifier处理方案.您需要做的就是在创建Contract时坚持正确的命名约定.该惯例要求包括订单号后跟下划线.这将考虑您是否使用YAML或Groovy.例：

```java
my_contracts_dir\
scenario1\
1_login.groovy
2_showCart.groovy
3_logout.groovy
```

这样的树导致Spring Cloud Contract Verifier生成名为 `scenario1` 的WireMock场景以及以下三个步骤：

登录标记为已开始指向... showCart标记为Step1指向...注销标记为Step2将关闭方案.

有关WireMock方案的更多详细信息，请访问[http://wiremock.org/docs/stateful-behaviour/](http://wiremock.org/docs/stateful-behaviour/)

Spring Cloud Contract Verifier还生成具有保证执行顺序的测试.

## 91.6 Docker项目

我们正在发布一个包含一个项目的 `springcloud/spring-cloud-contract`  Docker镜像，该项目将生成测试并在 `EXPLICIT` 模式下针对正在运行的应用程序执行它们.

>   `EXPLICIT` 模式意味着从Contract生成的测试将发送实际请求而不是模拟的请求.

### 91.6.1 Maven，JAR和二进制存储的简介

由于非JVM项目可以使用Docker镜像，因此最好解释Spring Cloud Contract打包默认值背后的基本术语.

部分以下定义取自[Maven Glossary](https://maven.apache.org/glossary.html)

-  `Project` ：Maven考虑项目.你要Build的一切都是项目.这些项目遵循明确定义的“项目对象模型”.项目可以依赖于其他项目，在这种情况下，后者称为“依赖项”.一个项目可能与几个子项目一致，但这些子项目仍然被视为项目.

-  `Artifact` ：工件是由项目生成或使用的东西. Maven为项目生成的工件示例包括：JAR，源和二进制分发.每个工件由组ID和工件ID唯一标识，工件ID在组内是唯一的.

-  `JAR` ：JAR代表Java ARchive.这是一种基于ZIP文件格式的格式. Spring Cloud Contract将Contract和生成的存根打包在JAR文件中.

-  `GroupId` ：组ID是项目的通用唯一标识符.虽然这通常只是项目名称（例如，commons-collections），但使用完全限定的包名称将其与具有类似名称的其他项目（例如org.apache.maven）区分开是有帮助的.通常，当发布到工件管理器时， `GroupId` 将被斜杠分隔并形成URL的一部分.例如.对于组ID  `com.example` 和工件ID  `application` 将是 `/com/example/application/` .

-  `Classifier` ：Maven依赖表示法如下所示： `groupId:artifactId:version:classifier` .分类器是传递给依赖项的附加后缀.例如.  `stubs` ， `sources` .相同的依赖，例如 `com.example:application` 可以使用分类器生成多个彼此不同的工件.

-  `Artifact manager` ：生成二进制文件/源/包时，您希望其他人可以下载/引用或重用它们.在JVM世界的情况下，那些工件将是JAR，对于Ruby来说这些是宝石，而对于Docker来说，那些将是Docker镜像.您可以将这些工件存储在管理器中.此类经理的示例可以是[Artifactory](https://jfrog.com/artifactory/)或[Nexus](http://www.sonatype.org/nexus/).

### 91.6.2它是如何工作的

图像搜索 `/contracts` 文件夹下的Contract.运行测试的输出将在 `/spring-cloud-contract/build` 文件夹下可用（它对于调试非常有用）.

这足以让您挂载Contract，传递环境变量，图像将：

- 生成Contract测试

- 对提供的URL执行测试

- 生成[WireMock](http://wiremock.org)存根

- （可选 - 默认情况下处于启用状态）将存根发布到工件管理器

#### Environment变量

Docker镜像需要一些环境变量指向正在运行的应用程序，指向工件管理器实例等.

-  `PROJECT_GROUP`   - 您项目的组ID.默认为 `com.example` 

-  `PROJECT_VERSION`   - 您项目的版本.默认为 `0.0.1-SNAPSHOT` 

-  `PROJECT_NAME`   - 工件ID.默认为 `example` 

-  `REPO_WITH_BINARIES_URL`   - 您的工件管理器的URL.默认为 `http://localhost:8081/artifactory/libs-release-local` ，这是本地运行的[Artifactory](https://jfrog.com/artifactory/)的默认URL

-  `REPO_WITH_BINARIES_USERNAME`   - （可选）安装工件管理器时的用户名
当工件管理器受到保护时，
-  `REPO_WITH_BINARIES_PASSWORD`   - （可选）密码

-  `PUBLISH_ARTIFACTS`   - 如果设置为 `true` ，则会将工件发布到二进制存储.默认为 `true` .

当Contract位于外部存储库中时，将使用这些环境变量.要启用此功能，您必须设置 `EXTERNAL_CONTRACTS_ARTIFACT_ID` 环境变量.

-  `EXTERNAL_CONTRACTS_GROUP_ID`   - 包含Contract的项目的组ID.默认为 `com.example` 

-  `EXTERNAL_CONTRACTS_ARTIFACT_ID` -包含Contract的项目的工件ID.

-  `EXTERNAL_CONTRACTS_CLASSIFIER` -Contract项目的分类器.默认为空

-  `EXTERNAL_CONTRACTS_VERSION`   - 包含Contract的项目版本.默认为 `+` ，相当于选择最新的

-  `EXTERNAL_CONTRACTS_REPO_WITH_BINARIES_URL`   - 您的工件管理器的URL.默认值为 `REPO_WITH_BINARIES_URL`  env var.如果未设置，则默认为 `http://localhost:8081/artifactory/libs-release-local` ，这是本地运行的[Artifactory](https://jfrog.com/artifactory/)的默认URL

-  `EXTERNAL_CONTRACTS_PATH`   - 包含Contract的项目内给定项目的Contract路径.默认为斜杠分隔 `EXTERNAL_CONTRACTS_GROUP_ID` 与 `/` 和 `EXTERNAL_CONTRACTS_ARTIFACT_ID` 连接.例如.对于组ID  `foo.bar` 和工件id  `baz` ，将导致 `foo/bar/baz`  contract路径.

-  `EXTERNAL_CONTRACTS_WORK_OFFLINE`   - 如果设置为 `true` ，则将从容器的 `.m2` 中检索具有Contract的工件.将本地 `.m2` 挂载为容器的 `/root/.m2` 路径上可用的卷.您不能同时设置 `EXTERNAL_CONTRACTS_WORK_OFFLINE` 和 `EXTERNAL_CONTRACTS_REPO_WITH_BINARIES_URL` .

执行测试时使用这些环境变量：

-  `APPLICATION_BASE_URL`   - 应该执行测试的url.请记住，必须可以从Docker容器访问它（例如 `localhost` 将无效）

-  `APPLICATION_USERNAME`   - （可选）用于应用程序基本身份验证的用户名

-  `APPLICATION_PASSWORD`   - （可选）用于应用程序基本身份验证的密码

### 91.6.3使用示例

我们来看一个简单的MVC应用程序

```java
$ git clone https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs
$ cd bookstore
```

Contract可在 `/contracts` 文件夹下找到.

### 91.6.4服务器端（nodejs）

由于我们想要运行测试，我们可以执行：

```java
$ npm test
```

但是，为了学习目的，我们将它分成几部分：

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

会发生什么是通过bash脚本：

将Build
- infrastructure（MongoDb，Artifactory）.在现实生活中，您只需使用模拟数据库运行NodeJS应用程序.在此示例中，我们希望展示如何立即从Spring Cloud Contract中受益.

- 由于这些限制，Contract也代表了有状态

- first请求是 `POST` ，导致数据插入数据库

- second请求是 `GET` ，它返回带有1个先前插入元素的数据列表

- 将启动NodeJS应用程序（在端口 `3000` 上）

- contract测试将通过Docker生成，测试将针对正在运行的应用程序执行

- Contract将取自 `/contracts` 文件夹.

- 测试执行的输出在 `node_modules/spring-cloud-contract/output` 下可用.

- 存根将上传到Artifactory.你可以在[http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/](http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/)下查看它们.存根将在这里[http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/bookstore-0.0.1.RELEASE-stubs.jar](http://localhost:8081/artifactory/libs-release-local/com/example/bookstore/0.0.1.RELEASE/bookstore-0.0.1.RELEASE-stubs.jar).

要查看客户端的外观，请查看[Section 93.9, “Stub Runner Docker”](multi__spring_cloud_contract_stub_runner.html#stubrunner-docker)部分.


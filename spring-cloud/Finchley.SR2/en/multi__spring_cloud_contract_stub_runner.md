## 93. Spring Cloud Contract Stub Runner

One of the issues that you might encounter while using Spring Cloud Contract Verifier is passing the generated WireMock JSON stubs from the server side to the client side (or to various clients). The same takes place in terms of client-side generation for messaging.

Copying the JSON files and setting the client side for messaging manually is out of the question. That is why we introduced Spring Cloud Contract Stub Runner. It can automatically download and run the stubs for you.

## 93.1 Snapshot versions

Add the additional snapshot repository to your  `build.gradle`  file to use snapshot versions, which are automatically uploaded after every successful build:

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

## 93.2 Publishing Stubs as JARs

The easiest approach would be to centralize the way stubs are kept. For example, you can keep them as jars in a Maven repository.

> For both Maven and Gradle, the setup comes ready to work. However, you can customize it if you want to.

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

Runs stubs for service collaborators. Treating stubs as contracts of services allows to use stub-runner as an implementation of [Consumer Driven Contracts](http://martinfowler.com/articles/consumerDrivenContracts.html).

Stub Runner allows you to automatically download the stubs of the provided dependencies (or pick those from the classpath), start WireMock servers for them and feed them with proper stub definitions. For messaging, special stub routes are defined.

### 93.3.1 Retrieving stubs

You can pick the following options of acquiring stubs

- Aether based solution that downloads JARs with stubs from Artifactory / Nexus

- Classpath scanning solution that searches classpath via pattern to retrieve stubs

- Write your own implementation of the  `org.springframework.cloud.contract.stubrunner.StubDownloaderBuilder`  for full customization

The latter example is described in the [Custom Stub Runner]() section.

#### Stub downloading

You can control the stub downloading via the  `stubsMode`  switch. It picks value from the  `StubRunnerProperties.StubsMode`  enum. You can use the following options

-  `StubRunnerProperties.StubsMode.CLASSPATH`  (default value) - will pick stubs from the classpath

-  `StubRunnerProperties.StubsMode.LOCAL`  - will pick stubs from a local storage (e.g.  `.m2` )

-  `StubRunnerProperties.StubsMode.REMOTE`  - will pick stubs from a remote location

Example:

```java
@AutoConfigureStubRunner(repositoryRoot="http://foo.bar", ids = "com.example:beer-api-producer:+:stubs:8095", stubsMode = StubRunnerProperties.StubsMode.LOCAL)
```

#### Classpath scanning

If you set the  `stubsMode`  property to  `StubRunnerProperties.StubsMode.CLASSPATH`  (or set nothing since  `CLASSPATH`  is the default value) then classpath will get scanned. Let’s look at the following example:

```java
@AutoConfigureStubRunner(ids = {
"com.example:beer-api-producer:+:stubs:8095",
"com.example.foo:bar:1.0.0:superstubs:8096"
})
```

If you’ve added the dependencies to your classpath

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

Then the following locations on your classpath will get scanned. For  `com.example:beer-api-producer-restdocs` 

- /META-INF/com.example/beer-api-producer-restdocs/ ***/** .*

- /contracts/com.example/beer-api-producer-restdocs/ ***/** .*

- /mappings/com.example/beer-api-producer-restdocs/ ***/** .*

and  `com.example.foo:bar` 

- /META-INF/com.example.foo/bar/ ***/** .*

- /contracts/com.example.foo/bar/ ***/** .*

- /mappings/com.example.foo/bar/ ***/** .*

> As you can see you have to explicitly provide the group and artifact ids when packaging the producer stubs.

The producer would setup the contracts like this:

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

To achieve proper stub packaging.

Or using the [Maven assembly plugin](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/blob/2.0.x/producer_with_restdocs/pom.xml) or [Gradle Jar](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/blob/2.0.x/producer_with_restdocs/build.gradle) task you have to create the following structure in your stubs jar.

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

By maintaining this structure classpath gets scanned and you can profit from the messaging / HTTP stubs without the need to download artifacts.

### 93.3.2 Running stubs

#### Running using main app

You can set the following options to the main class:

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

#### HTTP Stubs

Stubs are defined in JSON documents, whose syntax is defined in [WireMock documentation](http://wiremock.org/stubbing.html)

Example:

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

#### Viewing registered mappings

Every stubbed collaborator exposes list of defined mappings under  `__/admin/`  endpoint.

You can also use the  `mappingsOutputFolder`  property to dump the mappings to files. For annotation based approach it would look like this

```java
@AutoConfigureStubRunner(ids="a.b.c:loanIssuance,a.b.c:fraudDetectionServer",
mappingsOutputFolder = "target/outputmappings/")
```

and for the JUnit approach like this:

```java
@ClassRule @Shared StubRunnerRule rule = new StubRunnerRule()
			.repoRoot("http://some_url")
			.downloadStub("a.b.c", "loanIssuance")
			.downloadStub("a.b.c:fraudDetectionServer")
			.withMappingsOutputFolder("target/outputmappings")
```

Then if you check out the folder  `target/outputmappings`  you would see the following structure

```java
.
├── fraudDetectionServer_13705
└── loanIssuance_12255
```

That means that there were two stubs registered.  `fraudDetectionServer`  was registered at port  `13705`  and  `loanIssuance`  at port  `12255` . If we take a look at one of the files we would see (for WireMock) mappings available for the given server:

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

Depending on the provided Stub Runner dependency and the DSL the messaging routes are automatically set up.

## 93.4 Stub Runner JUnit Rule

Stub Runner comes with a JUnit rule thanks to which you can very easily download and run stubs for given group and artifact id:

```java
@ClassRule public static StubRunnerRule rule = new StubRunnerRule()
		.repoRoot(repoRoot())
		.downloadStub("org.springframework.cloud.contract.verifier.stubs", "loanIssuance")
		.downloadStub("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer");
```

After that rule gets executed Stub Runner connects to your Maven repository and for the given list of dependencies tries to:

- download them

- cache them locally

- unzip them to a temporary folder

- start a WireMock server for each Maven dependency on a random port from the provided range of ports / provided port

- feed the WireMock server with all JSON files that are valid WireMock definitions

- can also send messages (remember to pass an implementation of  `MessageVerifier`  interface)

Stub Runner uses [Eclipse Aether](https://wiki.eclipse.org/Aether) mechanism to download the Maven dependencies. Check their [docs](https://wiki.eclipse.org/Aether) for more information.

Since the  `StubRunnerRule`  implements the  `StubFinder`  it allows you to find the started stubs:

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

Example of usage in Spock tests:

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

Example of usage in JUnit tests:

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

Check the  **Common properties for JUnit and Spring**  for more information on how to apply global configuration of Stub Runner.

|images/important.png|Important|
|----|----|
|To use the JUnit rule together with messaging you have to provide an implementation of the  `MessageVerifier`  interface to the rule builder (e.g.  `rule.messageVerifier(new MyMessageVerifier())` ). If you don’t do this then whenever you try to send a message an exception will be thrown. |

### 93.4.1 Maven settings

The stub downloader honors Maven settings for a different local repository folder. Authentication details for repositories and profiles are currently not taken into account, so you need to specify it using the properties mentioned above.

### 93.4.2 Providing fixed ports

You can also run your stubs on fixed ports. You can do it in two different ways. One is to pass it in the properties, and the other via fluent API of JUnit rule.

### 93.4.3 Fluent API

When using the  `StubRunnerRule`  you can add a stub to download and then pass the port for the last downloaded stub.

```java
@ClassRule public static StubRunnerRule rule = new StubRunnerRule()
		.repoRoot(repoRoot())
		.downloadStub("org.springframework.cloud.contract.verifier.stubs", "loanIssuance")
		.withPort(12345)
		.downloadStub("org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer:12346");
```

You can see that for this example the following test is valid:

```java
then(rule.findStubUrl("loanIssuance")).isEqualTo(URI.create("http://localhost:12345").toURL());
then(rule.findStubUrl("fraudDetectionServer")).isEqualTo(URI.create("http://localhost:12346").toURL());
```

### 93.4.4 Stub Runner with Spring

Sets up Spring configuration of the Stub Runner project.

By providing a list of stubs inside your configuration file the Stub Runner automatically downloads and registers in WireMock the selected stubs.

If you want to find the URL of your stubbed dependency you can autowire the  `StubFinder`  interface and use its methods as presented below:

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

for the following configuration file:

```java
stubrunner:
repositoryRoot: classpath:m2repo/repository/
ids:
- org.springframework.cloud.contract.verifier.stubs:loanIssuance
- org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer
- org.springframework.cloud.contract.verifier.stubs:bootService
stubs-mode: remote
```

Instead of using the properties you can also use the properties inside the  `@AutoConfigureStubRunner` . Below you can find an example of achieving the same result by setting values on the annotation.

```java
@AutoConfigureStubRunner(
		ids = ["org.springframework.cloud.contract.verifier.stubs:loanIssuance",
		"org.springframework.cloud.contract.verifier.stubs:fraudDetectionServer",
		"org.springframework.cloud.contract.verifier.stubs:bootService"],
		stubsMode = StubRunnerProperties.StubsMode.REMOTE,
		repositoryRoot = "classpath:m2repo/repository/")
```

Stub Runner Spring registers environment variables in the following manner for every registered WireMock server. Example for Stub Runner ids  `com.example:foo` ,  `com.example:bar` .

-  `stubrunner.runningstubs.foo.port` 

-  `stubrunner.runningstubs.com.example.foo.port` 

-  `stubrunner.runningstubs.bar.port` 

-  `stubrunner.runningstubs.com.example.bar.port` 

Which you can reference in your code.

You can also use the  `@StubRunnerPort`  annotation to inject the port of a running stub. Value of the annotation can be the  `groupid:artifactid`  or just the  `artifactid` . Example for Stub Runner ids  `com.example:foo` ,  `com.example:bar` .

```java
@StubRunnerPort("foo")
int fooPort;
@StubRunnerPort("com.example:bar")
int barPort;
```

## 93.5 Stub Runner Spring Cloud

Stub Runner can integrate with Spring Cloud.

For real life examples you can check the

- [producer app sample](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/2.0.x/producer)

- [consumer app sample](https://github.com/spring-cloud-samples/spring-cloud-contract-samples/tree/2.0.x/consumer_with_discovery)

### 93.5.1 Stubbing Service Discovery

The most important feature of  `Stub Runner Spring Cloud`  is the fact that it’s stubbing

-  `DiscoveryClient` 

-  `Ribbon`   `ServerList` 

that means that regardless of the fact whether you’re using Zookeeper, Consul, Eureka or anything else, you don’t need that in your tests. We’re starting WireMock instances of your dependencies and we’re telling your application whenever you’re using  `Feign` , load balanced  `RestTemplate`  or  `DiscoveryClient`  directly, to call those stubbed servers instead of calling the real Service Discovery tool.

For example this test will pass

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

for the following configuration file

```java
stubrunner:
idsToServiceIds:
ivyNotation: someValueInsideYourCode
fraudDetectionServer: someNameThatShouldMapFraudDetectionServer
```

#### Test profiles and service discovery

In your integration tests you typically don’t want to call neither a discovery service (e.g. Eureka) or Config Server. That’s why you create an additional test configuration in which you want to disable these features.

Due to certain limitations of [spring-cloud-commons](https://github.com/spring-cloud/spring-cloud-commons/issues/156) to achieve this you have disable these properties via a static block like presented below (example for Eureka)

```java
//Hack to work around https://github.com/spring-cloud/spring-cloud-commons/issues/156
static {
System.setProperty("eureka.client.enabled", "false");
System.setProperty("spring.cloud.config.failFast", "false");
}
```

### 93.5.2 Additional Configuration

You can match the artifactId of the stub with the name of your app by using the  `stubrunner.idsToServiceIds:`  map. You can disable Stub Runner Ribbon support by providing:  `stubrunner.cloud.ribbon.enabled`  equal to  `false`  You can disable Stub Runner support by providing:  `stubrunner.cloud.enabled`  equal to  `false` 

> By default all service discovery will be stubbed. That means that regardless of the fact if you have an existing  `DiscoveryClient`  its results will be ignored. However, if you want to reuse it, just set  `stubrunner.cloud.delegate.enabled`  to  `true`  and then your existing  `DiscoveryClient`  results will be merged with the stubbed ones.

The default Maven configuration used by Stub Runner can be tweaked either via the following system properties or environment variables

-  `maven.repo.local`  - path to the custom maven local repository location

-  `org.apache.maven.user-settings`  - path to custom maven user settings location

-  `org.apache.maven.global-settings`  - path to maven global settings location

## 93.6 Stub Runner Boot Application

Spring Cloud Contract Stub Runner Boot is a Spring Boot application that exposes REST endpoints to trigger the messaging labels and to access started WireMock servers.

One of the use-cases is to run some smoke (end to end) tests on a deployed application. You can check out the [Spring Cloud Pipelines](https://github.com/spring-cloud/spring-cloud-pipelines) project for more information.

### 93.6.1 How to use it?

#### Stub Runner Server

Just add the

```java
compile "org.springframework.cloud:spring-cloud-starter-stub-runner"
```

Annotate a class with  `@EnableStubRunnerServer` , build a fat-jar and you’re ready to go!

For the properties check the  **Stub Runner Spring**  section.

#### Stub Runner Server Fat Jar

You can download a standalone JAR from Maven (for example, for version 1.2.3.RELEASE), as follows:

```java
$ wget -O stub-runner.jar 'https://search.maven.org/remote_content?g=org.springframework.cloud&a=spring-cloud-contract-stub-runner-boot&v=1.2.3.RELEASE'
$ java -jar stub-runner.jar --stubrunner.ids=... --stubrunner.repositoryRoot=...
```

#### Spring Cloud CLI

Starting from  `1.4.0.RELEASE`  version of the [Spring Cloud CLI](https://cloud.spring.io/spring-cloud-cli) project you can start Stub Runner Boot by executing  `spring cloud stubrunner` .

In order to pass the configuration just create a  `stubrunner.yml`  file in the current working directory or a subdirectory called  `config`  or in  `~/.spring-cloud` . The file could look like this (example for running stubs installed locally)

**stubrunner.yml.**  

```java
stubrunner:
stubsMode: LOCAL
ids:
- com.example:beer-api-producer:+:9876
```

and then just call  `spring cloud stubrunner`  from your terminal window to start the Stub Runner server. It will be available at port  `8750` .

### 93.6.2 Endpoints

#### HTTP

- GET  `/stubs`  - returns a list of all running stubs in  `ivy:integer`  notation

- GET  `/stubs/{ivy}`  - returns a port for the given  `ivy`  notation (when calling the endpoint  `ivy`  can also be  `artifactId`  only)

#### Messaging

For Messaging

- GET  `/triggers`  - returns a list of all running labels in  `ivy : [ label1, label2 …]`  notation

- POST  `/triggers/{label}`  - executes a trigger with  `label` 

- POST  `/triggers/{ivy}/{label}`  - executes a trigger with  `label`  for the given  `ivy`  notation (when calling the endpoint  `ivy`  can also be  `artifactId`  only)

### 93.6.3 Example

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

One of the possibilities of using Stub Runner Boot is to use it as a feed of stubs for "smoke-tests". What does it mean? Let’s assume that you don’t want to deploy 50 microservice to a test environment in order to check if your application is working fine. You’ve already executed a suite of tests during the build process but you would also like to ensure that the packaging of your application is fine. What you can do is to deploy your application to an environment, start it and run a couple of tests on it to see if it’s working fine. We can call those tests smoke-tests since their idea is to check only a handful of testing scenarios.

The problem with this approach is such that if you’re doing microservices most likely you’re using a service discovery tool. Stub Runner Boot allows you to solve this issue by starting the required stubs and register them in a service discovery tool. Let’s take a look at an example of such a setup with Eureka. Let’s assume that Eureka was already running.

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

As you can see we want to start a Stub Runner Boot server  `@EnableStubRunnerServer` , enable Eureka client  `@EnableEurekaClient`  and we want to have the stub runner feature turned on  `@AutoConfigureStubRunner` .

Now let’s assume that we want to start this application so that the stubs get automatically registered. We can do it by running the app  `java -jar ${SYSTEM_PROPS} stub-runner-boot-eureka-example.jar`  where  `${SYSTEM_PROPS}`  would contain the following list of properties

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

That way your deployed application can send requests to started WireMock servers via the service discovery. Most likely points 1-3 could be set by default in  `application.yml`  cause they are not likely to change. That way you can provide only the list of stubs to download whenever you start the Stub Runner Boot.

## 93.7 Stubs Per Consumer

There are cases in which 2 consumers of the same endpoint want to have 2 different responses.

> This approach also allows you to immediately know which consumer is using which part of your API. You can remove part of a response that your API produces and you can see which of your autogenerated tests fails. If none fails then you can safely delete that part of the response cause nobody is using it.

Let’s look at the following example for contract defined for the producer called  `producer` . There are 2 consumers:  `foo-consumer`  and  `bar-consumer` .

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

You can’t produce for the same request 2 different responses. That’s why you can properly package the contracts and then profit from the  `stubsPerConsumer`  feature.

On the producer side the consumers can have a folder that contains contracts related only to them. By setting the  `stubrunner.stubs-per-consumer`  flag to  `true`  we no longer register all stubs but only those that correspond to the consumer application’s name. In other words we’ll scan the path of every stub and if it contains the subfolder with name of the consumer in the path only then will it get registered.

On the  `foo`  producer side the contracts would look like this

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

Being the  `bar-consumer`  consumer you can either set the  `spring.application.name`  or the  `stubrunner.consumer-name`  to  `bar-consumer`  Or set the test as follows:

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

Then only the stubs registered under a path that contains the  `bar-consumer`  in its name (i.e. those from the  `src/test/resources/contracts/bar-consumer/some/contracts/…`  folder) will be allowed to be referenced.

Or set the consumer name explicitly

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

Then only the stubs registered under a path that contains the  `foo-consumer`  in its name (i.e. those from the  `src/test/resources/contracts/foo-consumer/some/contracts/…`  folder) will be allowed to be referenced.

You can check out [issue 224](https://github.com/spring-cloud/spring-cloud-contract/issues/224) for more information about the reasons behind this change.

## 93.8 Common

This section briefly describes common properties, including:

- [Section 93.8.1, “Common Properties for JUnit and Spring”](multi__spring_cloud_contract_stub_runner.html#common-properties-junit-spring)

- [Section 93.8.2, “Stub Runner Stubs IDs”](multi__spring_cloud_contract_stub_runner.html#stub-runner-stub-ids)

### 93.8.1 Common Properties for JUnit and Spring

You can set repetitive properties by using system properties or Spring configuration properties. Here are their names with their default values:

|Property name|Default value|Description|
|----|----|----|
|stubrunner.minPort |10000 |Minimum value of a port for a started WireMock with stubs. |
|stubrunner.maxPort |15000 |Maximum value of a port for a started WireMock with stubs. |
|stubrunner.repositoryRoot | |Maven repo URL. If blank, then call the local maven repo. |
|stubrunner.classifier |stubs |Default classifier for the stub artifacts. |
|stubrunner.stubsMode |CLASSPATH |The way you want to fetch and register the stubs |
|stubrunner.ids | |Array of Ivy notation stubs to download. |
|stubrunner.username | |Optional username to access the tool that stores the JARs with stubs. |
|stubrunner.password | |Optional password to access the tool that stores the JARs with stubs. |
|stubrunner.stubsPerConsumer |false |Set to  `true`  if you want to use different stubs for each consumer instead of registering all stubs for every consumer. |
|stubrunner.consumerName | |If you want to use a stub for each consumer and want to override the consumer name just change this value. |

### 93.8.2 Stub Runner Stubs IDs

You can provide the stubs to download via the  `stubrunner.ids`  system property. They follow this pattern:

```java
groupId:artifactId:version:classifier:port
```

Note that  `version` ,  `classifier`  and  `port`  are optional.

- If you do not provide the  `port` , a random one will be picked.

- If you do not provide the  `classifier` , the default is used. (Note that you can pass an empty classifier this way:  `groupId:artifactId:version:` ).

- If you do not provide the  `version` , then the  `+`  will be passed and the latest one is downloaded.

`port`  means the port of the WireMock server.

|images/important.png|Important|
|----|----|
|Starting with version 1.0.4, you can provide a range of versions that you would like the Stub Runner to take into consideration. You can read more about the [Aether versioning ranges here](https://wiki.eclipse.org/Aether/New_and_Noteworthy#Version_Ranges). |

## 93.9 Stub Runner Docker

We’re publishing a  `spring-cloud/spring-cloud-contract-stub-runner`  Docker image that will start the standalone version of Stub Runner.

If you want to learn more about the basics of Maven, artifact ids, group ids, classifiers and Artifact Managers, just click here [Section 91.6, “Docker Project”](multi__spring_cloud_contract_verifier_setup.html#docker-project).

### 93.9.1 How to use it

Just execute the docker image. You can pass any of the [Section 93.8.1, “Common Properties for JUnit and Spring”](multi__spring_cloud_contract_stub_runner.html#common-properties-junit-spring) as environment variables. The convention is that all the letters should be upper case. The camel case notation should and the dot ( `.` ) should be separated via underscore ( `_` ). E.g. the  `stubrunner.repositoryRoot`  property should be represented as a  `STUBRUNNER_REPOSITORY_ROOT`  environment variable.

### 93.9.2 Example of client side usage in a non JVM project

We’d like to use the stubs created in this [Section 91.6.4, “Server side (nodejs)”](multi__spring_cloud_contract_verifier_setup.html#docker-server-side) step. Let’s assume that we want to run the stubs on port  `9876` . The NodeJS code is available here:

```java
$ git clone https://github.com/spring-cloud-samples/spring-cloud-contract-nodejs
$ cd bookstore
```

Let’s run the Stub Runner Boot application with the stubs.

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

What’s happening is that

- a standalone Stub Runner application got started

- it downloaded the stub with coordinates  `com.example:bookstore:0.0.1.RELEASE:stubs`  on port  `9876` 

- it got downloaded from Artifactory running at  `http://192.168.0.100:8081/artifactory/libs-release-local` 

- after a while Stub Runner will be running on port  `8083` 

- and the stubs will be running at port  `9876` 

On the server side we built a stateful stub. Let’s use curl to assert that the stubs are setup properly.

```java
# let's execute the first request (no response is returned)
$ curl -H "Content-Type:application/json" -X POST --data '{ "title" : "Title", "genre" : "Genre", "description" : "Description", "author" : "Author", "publisher" : "Publisher", "pages" : 100, "image_url" : "https://d213dhlpdb53mu.cloudfront.net/assets/pivotal-square-logo-41418bd391196c3022f3cd9f3959b3f6d7764c47873d858583384e759c7db435.svg", "buy_url" : "https://pivotal.io" }' http://localhost:9876/api/books
# Now time for the second request
$ curl -X GET http://localhost:9876/api/books
# You will receive contents of the JSON
```

|images/important.png|Important|
|----|----|
|If you want use the stubs that you have built locally, on your host, then you should pass the environment variable  `-e STUBRUNNER_STUBS_MODE=LOCAL`  and mount the volume of your local m2  `-v "${HOME}/.m2/:/root/.m2:ro"`  |


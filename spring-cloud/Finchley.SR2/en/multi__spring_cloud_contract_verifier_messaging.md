## 92. Spring Cloud Contract Verifier Messaging

Spring Cloud Contract Verifier lets you verify applications that use messaging as a means of communication. All of the integrations shown in this document work with Spring, but you can also create one of your own and use that.

## 92.1 Integrations

You can use one of the following four integration configurations:

- Apache Camel

- Spring Integration

- Spring Cloud Stream

- Spring AMQP

Since we use Spring Boot, if you have added one of these libraries to the classpath, all the messaging configuration is automatically set up.

|images/important.png|Important|
|----|----|
|Remember to put  `@AutoConfigureMessageVerifier`  on the base class of your generated tests. Otherwise, messaging part of Spring Cloud Contract Verifier does not work. |

|images/important.png|Important|
|----|----|
|If you want to use Spring Cloud Stream, remember to add a dependency on  `org.springframework.cloud:spring-cloud-stream-test-support` , as shown here: |

**Maven.**  

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-stream-test-support</artifactId>
<scope>test</scope>
</dependency>
```

**Gradle.**  

```java
testCompile "org.springframework.cloud:spring-cloud-stream-test-support"
```

## 92.2 Manual Integration Testing

The main interface used by the tests is  `org.springframework.cloud.contract.verifier.messaging.MessageVerifier` . It defines how to send and receive messages. You can create your own implementation to achieve the same goal.

In a test, you can inject a  `ContractVerifierMessageExchange`  to send and receive messages that follow the contract. Then add  `@AutoConfigureMessageVerifier`  to your test. Here’s an example:

```java
@RunWith(SpringTestRunner.class)
@SpringBootTest
@AutoConfigureMessageVerifier
public static class MessagingContractTests {

@Autowired
private MessageVerifier verifier;
...
}
```

> If your tests require stubs as well, then  `@AutoConfigureStubRunner`  includes the messaging configuration, so you only need the one annotation.

## 92.3 Publisher-Side Test Generation

Having the  `input`  or  `outputMessage`  sections in your DSL results in creation of tests on the publisher’s side. By default, JUnit tests are created. However, there is also a possibility to create Spock tests.

There are 3 main scenarios that we should take into consideration:

- Scenario 1: There is no input message that produces an output message. The output message is triggered by a component inside the application (for example, scheduler).

- Scenario 2: The input message triggers an output message.

- Scenario 3: The input message is consumed and there is no output message.

|images/important.png|Important|
|----|----|
|The destination passed to  `messageFrom`  or  `sentTo`  can have different meanings for different messaging implementations. For  **Stream**  and  **Integration**  it is first resolved as a  `destination`  of a channel. Then, if there is no such  `destination`  it is resolved as a channel name. For  **Camel** , that’s a certain component (for example,  `jms` ). |

### 92.3.1 Scenario 1: No Input Message

For the given contract:

**Groovy DSL.**  

```java
def contractDsl = Contract.make {
	label 'some_label'
	input {
		triggeredBy('bookReturnedTriggered()')
	}
	outputMessage {
		sentTo('activemq:output')
		body('''{ "bookName" : "foo" }''')
		headers {
			header('BOOK-NAME', 'foo')
			messagingContentType(applicationJson())
		}
	}
}
```

**YAML.**  

```java
label: some_label
input:
triggeredBy: bookReturnedTriggered
outputMessage:
sentTo: activemq:output
body:
bookName: foo
headers:
BOOK-NAME: foo
contentType: application/json
```

The following JUnit test is created:

```java
'''
// when:
bookReturnedTriggered();

// then:
ContractVerifierMessage response = contractVerifierMessaging.receive("activemq:output");
assertThat(response).isNotNull();
assertThat(response.getHeader("BOOK-NAME")).isNotNull();
assertThat(response.getHeader("BOOK-NAME").toString()).isEqualTo("foo");
assertThat(response.getHeader("contentType")).isNotNull();
assertThat(response.getHeader("contentType").toString()).isEqualTo("application/json");
// and:
DocumentContext parsedJson = JsonPath.parse(contractVerifierObjectMapper.writeValueAsString(response.getPayload()));
assertThatJson(parsedJson).field("bookName").isEqualTo("foo");
'''
```

And the following Spock test would be created:

```java
'''
when:
bookReturnedTriggered()

then:
ContractVerifierMessage response = contractVerifierMessaging.receive('activemq:output')
assert response != null
response.getHeader('BOOK-NAME')?.toString()  == 'foo'
response.getHeader('contentType')?.toString()  == 'application/json'
and:
DocumentContext parsedJson = JsonPath.parse(contractVerifierObjectMapper.writeValueAsString(response.payload))
assertThatJson(parsedJson).field("bookName").isEqualTo("foo")

'''
```

### 92.3.2 Scenario 2: Output Triggered by Input

For the given contract:

**Groovy DSL.**  

```java
def contractDsl = Contract.make {
	label 'some_label'
	input {
		messageFrom('jms:input')
		messageBody([
				bookName: 'foo'
		])
		messageHeaders {
			header('sample', 'header')
		}
	}
	outputMessage {
		sentTo('jms:output')
		body([
				bookName: 'foo'
		])
		headers {
			header('BOOK-NAME', 'foo')
		}
	}
}
```

**YAML.**  

```java
label: some_label
input:
messageFrom: jms:input
messageBody:
bookName: 'foo'
messageHeaders:
sample: header
outputMessage:
sentTo: jms:output
body:
bookName: foo
headers:
BOOK-NAME: foo
```

The following JUnit test is created:

```java
'''
// given:
ContractVerifierMessage inputMessage = contractVerifierMessaging.create(
"{\\"bookName\\":\\"foo\\"}"
, headers()
.header("sample", "header"));

// when:
contractVerifierMessaging.send(inputMessage, "jms:input");

// then:
ContractVerifierMessage response = contractVerifierMessaging.receive("jms:output");
assertThat(response).isNotNull();
assertThat(response.getHeader("BOOK-NAME")).isNotNull();
assertThat(response.getHeader("BOOK-NAME").toString()).isEqualTo("foo");
// and:
DocumentContext parsedJson = JsonPath.parse(contractVerifierObjectMapper.writeValueAsString(response.getPayload()));
assertThatJson(parsedJson).field("bookName").isEqualTo("foo");
'''
```

And the following Spock test would be created:

```java
"""\
given:
ContractVerifierMessage inputMessage = contractVerifierMessaging.create(
'''{"bookName":"foo"}''',
['sample': 'header']
)

when:
contractVerifierMessaging.send(inputMessage, 'jms:input')

then:
ContractVerifierMessage response = contractVerifierMessaging.receive('jms:output')
assert response !- null
response.getHeader('BOOK-NAME')?.toString()  == 'foo'
and:
DocumentContext parsedJson = JsonPath.parse(contractVerifierObjectMapper.writeValueAsString(response.payload))
assertThatJson(parsedJson).field("bookName").isEqualTo("foo")
"""
```

### 92.3.3 Scenario 3: No Output Message

For the given contract:

**Groovy DSL.**  

```java
def contractDsl = Contract.make {
	label 'some_label'
	input {
		messageFrom('jms:delete')
		messageBody([
				bookName: 'foo'
		])
		messageHeaders {
			header('sample', 'header')
		}
		assertThat('bookWasDeleted()')
	}
}
```

**YAML.**  

```java
label: some_label
input:
messageFrom: jms:delete
messageBody:
bookName: 'foo'
messageHeaders:
sample: header
assertThat: bookWasDeleted()
```

The following JUnit test is created:

```java
'''
// given:
ContractVerifierMessage inputMessage = contractVerifierMessaging.create(
	"{\\"bookName\\":\\"foo\\"}"
, headers()
	.header("sample", "header"));

// when:
contractVerifierMessaging.send(inputMessage, "jms:delete");

// then:
bookWasDeleted();
'''
```

And the following Spock test would be created:

```java
'''
given:
	 ContractVerifierMessage inputMessage = contractVerifierMessaging.create(
		\'\'\'{"bookName":"foo"}\'\'\',
		['sample': 'header']
	)

when:
	 contractVerifierMessaging.send(inputMessage, 'jms:delete')

then:
	 noExceptionThrown()
	 bookWasDeleted()
'''
```

## 92.4 Consumer Stub Generation

Unlike the HTTP part, in messaging, we need to publish the Groovy DSL inside the JAR with a stub. Then it is parsed on the consumer side and proper stubbed routes are created.

For more information, see [Chapter 94, Stub Runner for Messaging](multi_stub-runner-for-messaging.html) section.

**Maven.**  

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-stream-rabbit</artifactId>
	</dependency>

	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-stream-test-support</artifactId>
		<scope>test</scope>
	</dependency>
</dependencies>

<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>Finchley.BUILD-SNAPSHOT</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
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


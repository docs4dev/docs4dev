## 92. Spring Cloud Contract Verifier Messaging

Spring Cloud Contract Verifier允许您验证使用消息传递作为通信方式的应用程序.本文档中显示的所有集成都适用于Spring，但您也可以创建自己的集成并使用它.

## 92.1集成

您可以使用以下四种集成配置之一：

- Apache骆驼

- Spring Integration

- Spring Cloud Stream

- Spring AMQP

由于我们使用Spring Boot，如果您已将其中一个库添加到类路径中，则会自动设置所有消息传递配置.

|图片/ important.png |重要|
| ---- | ---- |
|请记住将 `@AutoConfigureMessageVerifier` 放在生成的测试的基类上.否则，Spring Cloud Contract Verifier的消息传递部分不起作用. |

|图片/ important.png |重要|
| ---- | ---- |
|如果您想使用Spring Cloud Stream，请记住在 `org.springframework.cloud:spring-cloud-stream-test-support` 上添加依赖项，如下所示：|

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

## 92.2手动集成测试

测试使用的主界面是 `org.springframework.cloud.contract.verifier.messaging.MessageVerifier` .它定义了如何发送和接收消息.您可以创建自己的实现来实现相同的目标.

在测试中，您可以注入 `ContractVerifierMessageExchange` 来发送和接收遵循Contract的消息.然后将 `@AutoConfigureMessageVerifier` 添加到您的测试中.这是一个例子：

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

> 如果您的测试也需要存根，那么 `@AutoConfigureStubRunner` 包括消息传递配置，因此您只需要一个注释.

## 92.3发布者端测试代

在DSL中使用 `input` 或 `outputMessage` 部分会导致在发布者端创建测试.默认情况下，会创建JUnit测试.但是，也有可能创建Spock测试.

我们应该考虑以下三种主要方案：

- Scenario 1：没有输出消息产生输出消息.输出消息由应用程序内的组件（例如，调度程序）触发.

- Scenario 2：输入消息触发输出消息.

- Scenario 3：消耗输入消息，没有输出消息.

|图片/ important.png |重要|
| ---- | ---- |
|传递给 `messageFrom` 或 `sentTo` 的目标可以对不同的消息传递实现具有不同的含义.对于 **Stream** 和 **Integration** ，它首先被解析为通道的 `destination` .然后，如果没有这样的 `destination` ，则将其解析为Channels名称.对于 **Camel** ，这是某个组件（例如， `jms` ）. |

### 92.3.1场景1：无输入消息

对于给定的Contract：

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

创建以下JUnit测试：

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

并将创建以下Spock测试：

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

### 92.3.2场景2：输入触发的输出

对于给定的Contract：

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

创建以下JUnit测试：

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

并将创建以下Spock测试：

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

### 92.3.3场景3：无输出消息

对于给定的Contract：

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

创建以下JUnit测试：

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

并将创建以下Spock测试：

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

## 92.4消费者存根生成

与HTTP部分不同，在消息传递中，我们需要使用存根在JAR中发布Groovy DSL.然后在消费者端解析它并创建适当的存根路径.

有关更多信息，请参阅[Chapter 94, Stub Runner for Messaging](multi_stub-runner-for-messaging.html)部分.

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


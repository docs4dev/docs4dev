## 94. Stub Runner for Messaging

Stub Runner可以在内存中运行已发布的存根.它可以与以下框架集成：

- Spring Integration

- Spring Cloud Stream

- Spring AMQP

它还提供了与市场上任何其他解决方案集成的切入点.

|图片/ important.png |重要|
| ---- | ---- |
|如果类路径上有多个框架，Stub Runner将需要定义应该使用哪个框架.假设您在类路径上同时拥有AMQP，Spring Cloud Stream和Spring Integration.然后你需要设置 `stubrunner.stream.enabled=false` 和 `stubrunner.integration.enabled=false` .这样唯一剩下的框架就是Spring AMQP. |

## 94.1存根触发

要触发消息，请使用 `StubTrigger` 接口：

```java
package org.springframework.cloud.contract.stubrunner;

import java.util.Collection;
import java.util.Map;

public interface StubTrigger {

	/**
	 * Triggers an event by a given label for a given {@code groupid:artifactid} notation. You can use only {@code artifactId} too.
	 *
	 * Feature related to messaging.
	 *
	 * @return true - if managed to run a trigger
	 */
	boolean trigger(String ivyNotation, String labelName);

	/**
	 * Triggers an event by a given label.
	 *
	 * Feature related to messaging.
	 *
	 * @return true - if managed to run a trigger
	 */
	boolean trigger(String labelName);

	/**
	 * Triggers all possible events.
	 *
	 * Feature related to messaging.
	 *
	 * @return true - if managed to run a trigger
	 */
	boolean trigger();

	/**
	 * Returns a mapping of ivy notation of a dependency to all the labels it has.
	 *
	 * Feature related to messaging.
	 */
	Map<String, Collection<String>> labels();
}
```

为方便起见， `StubFinder` 接口扩展 `StubTrigger` ，因此您只需要测试中的一个或另一个.

`StubTrigger` 为您提供以下选项来触发消息：

- [Section 94.1.1, “Trigger by Label”](multi_stub-runner-for-messaging.html#trigger-label)

- [Section 94.1.2, “Trigger by Group and Artifact Ids”](multi_stub-runner-for-messaging.html#trigger-group-artifact-ids)

- [Section 94.1.3, “Trigger by Artifact Ids”](multi_stub-runner-for-messaging.html#trigger-artifact-ids)

- [Section 94.1.4, “Trigger All Messages”](multi_stub-runner-for-messaging.html#trigger-all-messages)

### 94.1.1按标签触发

```java
stubFinder.trigger('return_book_1')
```

### 94.1.2按组和工件ID触发

```java
stubFinder.trigger('org.springframework.cloud.contract.verifier.stubs:streamService', 'return_book_1')
```

### 94.1.3由神器ID触发

```java
stubFinder.trigger('streamService', 'return_book_1')
```

### 94.1.4触发所有消息

```java
stubFinder.trigger()
```

## 94.2 Stub Runner Integration

Spring Cloud Contract Verifier Stub Runner的消息传递模块为您提供了一种与Spring Integration集成的简便方法.对于提供的工件，它会自动下载存根并注册所需的路由.

### 94.2.1将Runner添加到项目中

您可以在类路径上同时拥有Spring Integration和Spring Cloud Contract Stub Runner.请记住使用 `@AutoConfigureStubRunner` 注释您的测试类.

### 94.2.2禁用该功能

如果需要禁用此功能，请设置 `stubrunner.integration.enabled=false` 属性.

假设您有以下Maven存储库以及 `integrationService` 应用程序的已部署存根：

```java
└── .m2
└── repository
└── io
└── codearte
└── accurest
└── stubs
└── integrationService
├── 0.0.1-SNAPSHOT
│ ├── integrationService-0.0.1-SNAPSHOT.pom
│ ├── integrationService-0.0.1-SNAPSHOT-stubs.jar
│ └── maven-metadata-local.xml
└── maven-metadata-local.xml
```

进一步假设存根包含以下结构：

```java
├── META-INF
│ └── MANIFEST.MF
└── repository
├── accurest
│ ├── bookDeleted.groovy
│ ├── bookReturned1.groovy
│ └── bookReturned2.groovy
└── mappings
```

考虑以下Contract（编号为 **1** ）：

```java
Contract.make {
	label 'return_book_1'
	input {
		triggeredBy('bookReturnedTriggered()')
	}
	outputMessage {
		sentTo('output')
		body('''{ "bookName" : "foo" }''')
		headers {
			header('BOOK-NAME', 'foo')
		}
	}
}
```

现在考虑 **2** ：

```java
Contract.make {
	label 'return_book_2'
	input {
		messageFrom('input')
		messageBody([
				bookName: 'foo'
		])
		messageHeaders {
			header('sample', 'header')
		}
	}
	outputMessage {
		sentTo('output')
		body([
				bookName: 'foo'
		])
		headers {
			header('BOOK-NAME', 'foo')
		}
	}
}
```

以及以下Spring集成路径：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/integration"
			 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
			 xmlns:beans="http://www.springframework.org/schema/beans"
			 xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd
			http://www.springframework.org/schema/integration
			http://www.springframework.org/schema/integration/spring-integration.xsd">

	<!-- REQUIRED FOR TESTING -->
	<bridge input-channel="output"
			output-channel="outputTest"/>

	<channel id="outputTest">
		<queue/>
	</channel>

</beans:beans>
```

这些示例适用于三种情况：

- [the section called “Scenario 1 (no input message)”](multi_stub-runner-for-messaging.html#integration-scenario-1)

- [the section called “Scenario 2 (output triggered by input)”](multi_stub-runner-for-messaging.html#integration-scenario-2)

- [the section called “Scenario 3 (input with no output)”](multi_stub-runner-for-messaging.html#integration-scenario-3)

#### Scenario 1（无输入消息）

要通过 `return_book_1` 标签触发消息，请使用 `StubTigger` 接口，如下所示：

```java
stubFinder.trigger('return_book_1')
```

要收听发送到 `output` 的消息的输出：

```java
Message<?> receivedMessage = messaging.receive('outputTest')
```

收到的消息将传递以下断言：

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 2（输入触发输出）

由于为您设置了路线，您可以向 `output` 目的地发送消息：

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'input')
```

要收听发送到 `output` 的消息的输出：

```java
Message<?> receivedMessage = messaging.receive('outputTest')
```

收到的消息传递以下断言：

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 3（没有输出的输入）

由于为您设置了路线，您可以向 `input` 目的地发送消息：

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'delete')
```

## 94.3 Stub Runner Stream

Spring Cloud Contract Verifier Stub Runner的消息传递模块为您提供了一种与Spring Stream集成的简便方法.对于提供的工件，它会自动下载存根并注册所需的路由.

> 如果Stub Runner与Stream的集成，则 `messageFrom` 或 `sentTo` 字符串首先作为通道的 `destination` 解析，并且不存在此类 `destination` ，目标将被解析为通道名称.

|图片/ important.png |重要|
| ---- | ---- |
|如果要使用Spring Cloud Stream记住，要在 `org.springframework.cloud:spring-cloud-stream-test-support` 上添加依赖项. |

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

### 94.3.1将Runner添加到项目中

您可以在类路径上同时拥有Spring Cloud Stream和Spring Cloud Contract Stub Runner.请记住使用 `@AutoConfigureStubRunner` 注释您的测试类.

### 94.3.2禁用该功能

如果需要禁用此功能，请设置 `stubrunner.stream.enabled=false` 属性.

假设您有以下Maven存储库以及 `streamService` 应用程序的已部署存根：

```java
└── .m2
└── repository
└── io
└── codearte
└── accurest
└── stubs
└── streamService
├── 0.0.1-SNAPSHOT
│ ├── streamService-0.0.1-SNAPSHOT.pom
│ ├── streamService-0.0.1-SNAPSHOT-stubs.jar
│ └── maven-metadata-local.xml
└── maven-metadata-local.xml
```

进一步假设存根包含以下结构：

```java
├── META-INF
│ └── MANIFEST.MF
└── repository
├── accurest
│ ├── bookDeleted.groovy
│ ├── bookReturned1.groovy
│ └── bookReturned2.groovy
└── mappings
```

考虑以下Contract（编号为 **1** ）：

```java
Contract.make {
	label 'return_book_1'
	input { triggeredBy('bookReturnedTriggered()') }
	outputMessage {
		sentTo('returnBook')
		body('''{ "bookName" : "foo" }''')
		headers { header('BOOK-NAME', 'foo') }
	}
}
```

现在考虑 **2** ：

```java
Contract.make {
	label 'return_book_2'
	input {
		messageFrom('bookStorage')
		messageBody([
			bookName: 'foo'
		])
		messageHeaders { header('sample', 'header') }
	}
	outputMessage {
		sentTo('returnBook')
		body([
			bookName: 'foo'
		])
		headers { header('BOOK-NAME', 'foo') }
	}
}
```

现在考虑以下Spring配置：

```java
stubrunner.repositoryRoot: classpath:m2repo/repository/
stubrunner.ids: org.springframework.cloud.contract.verifier.stubs:streamService:0.0.1-SNAPSHOT:stubs
stubrunner.stubs-mode: remote
spring:
cloud:
stream:
bindings:
output:
destination: returnBook
input:
destination: bookStorage

server:
port: 0

debug: true
```

这些示例适用于三种情况：

- [the section called “Scenario 1 (no input message)”](multi_stub-runner-for-messaging.html#stream-scenario-1)

- [the section called “Scenario 2 (output triggered by input)”](multi_stub-runner-for-messaging.html#stream-scenario-2)

- [the section called “Scenario 3 (input with no output)”](multi_stub-runner-for-messaging.html#stream-scenario-3)

#### Scenario 1（无输入消息）

要通过 `return_book_1` 标签触发消息，请使用 `StubTrigger` 接口，如下所示：

```java
stubFinder.trigger('return_book_1')
```

要收听发送到 `destination` 为 `returnBook` 的Channels的消息输出：

```java
Message<?> receivedMessage = messaging.receive('returnBook')
```

收到的消息传递以下断言：

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 2（输入触发输出）

自路线为你设置，你可以发送消息给 `bookStorage`   `destination` ：

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'bookStorage')
```

要收听发送到 `returnBook` 的消息的输出：

```java
Message<?> receivedMessage = messaging.receive('returnBook')
```

收到的消息传递以下断言：

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 3（没有输出的输入）

由于为您设置了路线，您可以向 `output` 目的地发送消息：

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'delete')
```

## 94.4 Stub Runner Spring AMQP

Spring Cloud Contract Verifier Stub Runner的消息传递模块提供了一种与Spring AMQP的Rabbit Template集成的简便方法.对于提供的工件，它会自动下载存根并注册所需的路由.

集成尝试独立工作（即，不与正在运行的RabbitMQ消息代理交互）.它期望在应用程序上下文中使用 `RabbitTemplate` 并将其用作名为 `@SpyBean` 的spring启动测试.因此，它可以使用mockitoSpy功能来验证和检查应用程序发送的消息.

在消息使用者方面，存根运行器会考虑应用程序上下文中的所有 `@RabbitListener` 带注释的endpoints和所有 `SimpleMessageListenerContainer` 对象.

由于消息通常发送到AMQP中的交换机，因此消息Contract包含交换名称作为目标.另一侧的消息侦听器绑定到队列.绑定将交换连接到队列.如果触发了消息协定，则Spring AMQP存根运行器集成将查找与此交换匹配的应用程序上下文的绑定.然后它从Spring交换中收集队列并尝试查找绑定到这些队列的消息侦听器.将为所有匹配的消息侦听器触发消息.

### 94.4.1将Runner添加到项目中

您可以在类路径上同时拥有Spring AMQP和Spring Cloud Contract Stub Runner，并设置属性 `stubrunner.amqp.enabled=true` .请记住使用 `@AutoConfigureStubRunner` 注释您的测试类.

|图片/ important.png |重要|
| ---- | ---- |
|如果已在类路径上安装了Stream和Integration，则需要通过设置 `stubrunner.stream.enabled=false` 和 `stubrunner.integration.enabled=false` 属性来显式禁用它们. |

假设您有以下Maven存储库以及 `spring-cloud-contract-amqp-test` 应用程序的已部署存根.

```java
└── .m2
└── repository
└── com
└── example
└── spring-cloud-contract-amqp-test
├── 0.4.0-SNAPSHOT
│ ├── spring-cloud-contract-amqp-test-0.4.0-SNAPSHOT.pom
│ ├── spring-cloud-contract-amqp-test-0.4.0-SNAPSHOT-stubs.jar
│ └── maven-metadata-local.xml
└── maven-metadata-local.xml
```

进一步假设存根包含以下结构：

```java
├── META-INF
│ └── MANIFEST.MF
└── contracts
└── shouldProduceValidPersonData.groovy
```

考虑以下Contract：

```java
Contract.make {
// Human readable description
description 'Should produce valid person data'
// Label by means of which the output message can be triggered
label 'contract-test.person.created.event'
// input to the contract
input {
// the contract will be triggered by a method
triggeredBy('createPerson()')
}
// output message of the contract
outputMessage {
// destination to which the output message will be sent
sentTo 'contract-test.exchange'
headers {
header('contentType': 'application/json')
header('__TypeId__': 'org.springframework.cloud.contract.stubrunner.messaging.amqp.Person')
}
// the body of the output message
body ([
id: $(consumer(9), producer(regex("[0-9]+"))),
name: "me"
])
}
}
```

现在考虑以下Spring配置：

```java
stubrunner:
repositoryRoot: classpath:m2repo/repository/
ids: org.springframework.cloud.contract.verifier.stubs.amqp:spring-cloud-contract-amqp-test:0.4.0-SNAPSHOT:stubs
stubs-mode: remote
amqp:
enabled: true
server:
port: 0
```

#### Triggering消息

要使用上述Contract触发消息，请使用 `StubTrigger` 接口，如下所示：

```java
stubTrigger.trigger("contract-test.person.created.event")
```

该消息的目标为 `contract-test.exchange` ，因此Spring AMQP存根运行器集成会查找与此交换相关的绑定.

```java
@Bean
public Binding binding() {
	return BindingBuilder.bind(new Queue("test.queue")).to(new DirectExchange("contract-test.exchange")).with("#");
}
```

绑定定义绑定队列 `test.queue` .因此，以下侦听器定义与Contract消息匹配并调用.

```java
@Bean
public SimpleMessageListenerContainer simpleMessageListenerContainer(ConnectionFactory connectionFactory,
																		MessageListenerAdapter listenerAdapter) {
	SimpleMessageListenerContainer container = new SimpleMessageListenerContainer();
	container.setConnectionFactory(connectionFactory);
	container.setQueueNames("test.queue");
	container.setMessageListener(listenerAdapter);

	return container;
}
```

此外，以下带注释的侦听器匹配并被调用：

```java
@RabbitListener(bindings = @QueueBinding(
		value = @Queue(value = "test.queue"),
		exchange = @Exchange(value = "contract-test.exchange", ignoreDeclarationExceptions = "true")))
public void handlePerson(Person person) {
	this.person = person;
}
```

> 消息直接移交给与匹配 `SimpleMessageListenerContainer` 关联的 `MessageListener` 的 `onMessage` 方法.

#### Spring AMQP测试配置

为了避免Spring AMQP在我们的测试期间尝试连接到正在运行的代理，请配置mock  `ConnectionFactory` .

要禁用模拟的ConnectionFactory，请设置以下属性： `stubrunner.amqp.mockConnection=false` 

```java
stubrunner:
amqp:
mockConnection: false
```


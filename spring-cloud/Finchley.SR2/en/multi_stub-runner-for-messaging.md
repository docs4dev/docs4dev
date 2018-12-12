## 94. Stub Runner for Messaging

Stub Runner can run the published stubs in memory. It can integrate with the following frameworks:

- Spring Integration

- Spring Cloud Stream

- Spring AMQP

It also provides entry points to integrate with any other solution on the market.

|images/important.png|Important|
|----|----|
|If you have multiple frameworks on the classpath Stub Runner will need to define which one should be used. Let’s assume that you have both AMQP, Spring Cloud Stream and Spring Integration on the classpath. Then you need to set  `stubrunner.stream.enabled=false`  and  `stubrunner.integration.enabled=false` . That way the only remaining framework is Spring AMQP. |

## 94.1 Stub triggering

To trigger a message, use the  `StubTrigger`  interface:

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

For convenience, the  `StubFinder`  interface extends  `StubTrigger` , so you only need one or the other in your tests.

`StubTrigger`  gives you the following options to trigger a message:

- [Section 94.1.1, “Trigger by Label”](multi_stub-runner-for-messaging.html#trigger-label)

- [Section 94.1.2, “Trigger by Group and Artifact Ids”](multi_stub-runner-for-messaging.html#trigger-group-artifact-ids)

- [Section 94.1.3, “Trigger by Artifact Ids”](multi_stub-runner-for-messaging.html#trigger-artifact-ids)

- [Section 94.1.4, “Trigger All Messages”](multi_stub-runner-for-messaging.html#trigger-all-messages)

### 94.1.1 Trigger by Label

```java
stubFinder.trigger('return_book_1')
```

### 94.1.2 Trigger by Group and Artifact Ids

```java
stubFinder.trigger('org.springframework.cloud.contract.verifier.stubs:streamService', 'return_book_1')
```

### 94.1.3 Trigger by Artifact Ids

```java
stubFinder.trigger('streamService', 'return_book_1')
```

### 94.1.4 Trigger All Messages

```java
stubFinder.trigger()
```

## 94.2 Stub Runner Integration

Spring Cloud Contract Verifier Stub Runner’s messaging module gives you an easy way to integrate with Spring Integration. For the provided artifacts, it automatically downloads the stubs and registers the required routes.

### 94.2.1 Adding the Runner to the Project

You can have both Spring Integration and Spring Cloud Contract Stub Runner on the classpath. Remember to annotate your test class with  `@AutoConfigureStubRunner` .

### 94.2.2 Disabling the functionality

If you need to disable this functionality, set the  `stubrunner.integration.enabled=false`  property.

Assume that you have the following Maven repository with deployed stubs for the  `integrationService`  application:

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

Further assume the stubs contain the following structure:

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

Consider the following contracts (numbered  **1** ):

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

Now consider  **2** :

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

and the following Spring Integration Route:

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

These examples lend themselves to three scenarios:

- [the section called “Scenario 1 (no input message)”](multi_stub-runner-for-messaging.html#integration-scenario-1)

- [the section called “Scenario 2 (output triggered by input)”](multi_stub-runner-for-messaging.html#integration-scenario-2)

- [the section called “Scenario 3 (input with no output)”](multi_stub-runner-for-messaging.html#integration-scenario-3)

#### Scenario 1 (no input message)

To trigger a message via the  `return_book_1`  label, use the  `StubTigger`  interface, as follows:

```java
stubFinder.trigger('return_book_1')
```

To listen to the output of the message sent to  `output` :

```java
Message<?> receivedMessage = messaging.receive('outputTest')
```

The received message would pass the following assertions:

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 2 (output triggered by input)

Since the route is set for you, you can send a message to the  `output`  destination:

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'input')
```

To listen to the output of the message sent to  `output` :

```java
Message<?> receivedMessage = messaging.receive('outputTest')
```

The received message passes the following assertions:

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 3 (input with no output)

Since the route is set for you, you can send a message to the  `input`  destination:

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'delete')
```

## 94.3 Stub Runner Stream

Spring Cloud Contract Verifier Stub Runner’s messaging module gives you an easy way to integrate with Spring Stream. For the provided artifacts, it automatically downloads the stubs and registers the required routes.

> If Stub Runner’s integration with Stream the  `messageFrom`  or  `sentTo`  Strings are resolved first as a  `destination`  of a channel and no such  `destination`  exists, the destination is resolved as a channel name.

|images/important.png|Important|
|----|----|
|If you want to use Spring Cloud Stream remember, to add a dependency on  `org.springframework.cloud:spring-cloud-stream-test-support` . |

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

### 94.3.1 Adding the Runner to the Project

You can have both Spring Cloud Stream and Spring Cloud Contract Stub Runner on the classpath. Remember to annotate your test class with  `@AutoConfigureStubRunner` .

### 94.3.2 Disabling the functionality

If you need to disable this functionality, set the  `stubrunner.stream.enabled=false`  property.

Assume that you have the following Maven repository with a deployed stubs for the  `streamService`  application:

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

Further assume the stubs contain the following structure:

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

Consider the following contracts (numbered  **1** ):

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

Now consider  **2** :

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

Now consider the following Spring configuration:

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

These examples lend themselves to three scenarios:

- [the section called “Scenario 1 (no input message)”](multi_stub-runner-for-messaging.html#stream-scenario-1)

- [the section called “Scenario 2 (output triggered by input)”](multi_stub-runner-for-messaging.html#stream-scenario-2)

- [the section called “Scenario 3 (input with no output)”](multi_stub-runner-for-messaging.html#stream-scenario-3)

#### Scenario 1 (no input message)

To trigger a message via the  `return_book_1`  label, use the  `StubTrigger`  interface as follows:

```java
stubFinder.trigger('return_book_1')
```

To listen to the output of the message sent to a channel whose  `destination`  is  `returnBook` :

```java
Message<?> receivedMessage = messaging.receive('returnBook')
```

The received message passes the following assertions:

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 2 (output triggered by input)

Since the route is set for you, you can send a message to the  `bookStorage`   `destination` :

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'bookStorage')
```

To listen to the output of the message sent to  `returnBook` :

```java
Message<?> receivedMessage = messaging.receive('returnBook')
```

The received message passes the following assertions:

```java
receivedMessage != null
assertJsons(receivedMessage.payload)
receivedMessage.headers.get('BOOK-NAME') == 'foo'
```

#### Scenario 3 (input with no output)

Since the route is set for you, you can send a message to the  `output`  destination:

```java
messaging.send(new BookReturned('foo'), [sample: 'header'], 'delete')
```

## 94.4 Stub Runner Spring AMQP

Spring Cloud Contract Verifier Stub Runner’s messaging module provides an easy way to integrate with Spring AMQP’s Rabbit Template. For the provided artifacts, it automatically downloads the stubs and registers the required routes.

The integration tries to work standalone (that is, without interaction with a running RabbitMQ message broker). It expects a  `RabbitTemplate`  on the application context and uses it as a spring boot test named  `@SpyBean` . As a result, it can use the mockito spy functionality to verify and inspect messages sent by the application.

On the message consumer side, the stub runner considers all  `@RabbitListener`  annotated endpoints and all  `SimpleMessageListenerContainer`  objects on the application context.

As messages are usually sent to exchanges in AMQP, the message contract contains the exchange name as the destination. Message listeners on the other side are bound to queues. Bindings connect an exchange to a queue. If message contracts are triggered, the Spring AMQP stub runner integration looks for bindings on the application context that match this exchange. Then it collects the queues from the Spring exchanges and tries to find message listeners bound to these queues. The message is triggered for all matching message listeners.

### 94.4.1 Adding the Runner to the Project

You can have both Spring AMQP and Spring Cloud Contract Stub Runner on the classpath and set the property  `stubrunner.amqp.enabled=true` . Remember to annotate your test class with  `@AutoConfigureStubRunner` .

|images/important.png|Important|
|----|----|
|If you already have Stream and Integration on the classpath, you need to disable them explicitly by setting the  `stubrunner.stream.enabled=false`  and  `stubrunner.integration.enabled=false`  properties. |

Assume that you have the following Maven repository with a deployed stubs for the  `spring-cloud-contract-amqp-test`  application.

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

Further assume that the stubs contain the following structure:

```java
├── META-INF
│ └── MANIFEST.MF
└── contracts
└── shouldProduceValidPersonData.groovy
```

Consider the following contract:

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

Now consider the following Spring configuration:

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

#### Triggering the message

To trigger a message using the contract above, use the  `StubTrigger`  interface as follows:

```java
stubTrigger.trigger("contract-test.person.created.event")
```

The message has a destination of  `contract-test.exchange` , so the Spring AMQP stub runner integration looks for bindings related to this exchange.

```java
@Bean
public Binding binding() {
	return BindingBuilder.bind(new Queue("test.queue")).to(new DirectExchange("contract-test.exchange")).with("#");
}
```

The binding definition binds the queue  `test.queue` . As a result, the following listener definition is matched and invoked with the contract message.

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

Also, the following annotated listener matches and is invoked:

```java
@RabbitListener(bindings = @QueueBinding(
		value = @Queue(value = "test.queue"),
		exchange = @Exchange(value = "contract-test.exchange", ignoreDeclarationExceptions = "true")))
public void handlePerson(Person person) {
	this.person = person;
}
```

> The message is directly handed over to the  `onMessage`  method of the  `MessageListener`  associated with the matching  `SimpleMessageListenerContainer` .

#### Spring AMQP Test Configuration

In order to avoid Spring AMQP trying to connect to a running broker during our tests configure a mock  `ConnectionFactory` .

To disable the mocked ConnectionFactory, set the following property:  `stubrunner.amqp.mockConnection=false` 

```java
stubrunner:
amqp:
mockConnection: false
```


## 89. Spring Cloud Contract Verifier简介

>  Accurest项目最初由Marcin Grzejszczak和Jakub Kubrynski（[codearte.io](http://codearte.io)）发起

Spring Cloud Contract Verifier支持基于JVM的应用程序的消费者驱动Contract（CDC）开发.它将TDD转移到软件架构的水平.

Spring Cloud Contract Verifier附带Contract定义语言（CDL）.Contract定义用于生成以下资源：

在对客户端代码进行集成测试时，WireMock将使用
- JSON存根定义（客户端测试）.测试代码仍必须手工编写，测试数据由Spring Cloud Contract Verifier生成.

- Messaging路由，如果您正在使用消息传递服务.我们与Spring Integration，Spring Cloud Stream，Spring AMQP和Apache Camel集成.您还可以设置自己的集成.

- Acceptance测试（在JUnit或Spock中）用于验证API的服务器端实现是否符合Contract（服务器测试）. Spring Cloud Contract Verifier生成完整测试.

## 89.1为何选择Contract审核员？

假设我们有一个由多个微服务组成的系统：

![Microservices Architecture](https://www.docs4dev.com/images/48a00967-17c2-43ff-8fa7-1ac61b7054bc.png)

### 89.1.1测试问题

如果我们想在左上角测试应用程序以确定它是否可以与其他服务通信，我们可以做以下两件事之一：

- Deploy所有微服务并执行端到端测试.

- Mock单元/集成测试中的其他微服务.

两者都有其优点，但也有很多缺点.

**Deploy all microservices and perform end to end tests** 

好处：

- Simulations生产环境.

- 测试服务之间的真实通信.

缺点：

- 测试一个微服务，我们必须部署6个微服务，几个数据库等.

- 测试运行的环境被锁定为一组测试（在此期间没有其他人能够运行测试）.

- 他们需要很长时间才能运行.

- 反馈来得很晚.

- 他们很难调试.

**Mock other microservices in unit/integration tests** 

好处：

- 他们提供非常快速的反馈.

- 他们没有基础设施要求.

缺点：

- 该服务的实现者创建可能与现实无关的存根.

- 您可以通过测试和生产环境失败进入生产环境阶段.

为了解决上述问题，创建了带有Stub Runner的Spring Cloud Contract Verifier.主要思想是为您提供非常快速的反馈，而无需设置整个微服务世界.如果您使用存根，那么您需要的唯一应用程序是您的应用程序直接使用的应用程序.

![Stubbed Services](https://www.docs4dev.com/images/80351d47-f946-45a9-a7be-848b1401d613.png)

Spring Cloud Contract Verifier可以确保您使用的存根是由您正在调用的服务创建的.此外，如果您可以使用它们，则意味着它们是针对生产环境者方进行测试的.简而言之，您可以信任这些存根.

## 89.2目的

使用Stub Runner的Spring Cloud Contract Verifier的主要目的是：

- 确保WireMock / Messaging存根（在开发客户端时使用）完全符合实际的服务器端实现.

- To推广ATDD方法和微服务架构风格.

- To提供了一种方法来发布双方立即可见的Contract中的更改.

- 生成要在服务器端使用的样板测试代码.

|图片/ important.png |重要|
| ---- | ---- |
| Spring Cloud Contract Verifier的目的不是开始在Contract中编写业务功能.假设我们有欺诈检查的业务用例.如果用户出于100种不同的原因可能是欺诈行为，我们会假设你会创建2份Contract，一份是正面案件，另一份是负面案件.Contract测试用于测试应用程序之间的Contract，而不是模拟完整行为. |

## 89.3如何运作

本节探讨如何使用Stub Runner运行Spring Cloud Contract Verifier.

### 89.3.1三秒游

这个非常简短的游览将介绍使用Spring Cloud Contract：

- [the section called “On the Producer Side”](multi__spring_cloud_contract_verifier_introduction.html#spring-cloud-contract-verifier-intro-three-second-tour-producer)

- [the section called “On the Consumer Side”](multi__spring_cloud_contract_verifier_introduction.html#spring-cloud-contract-verifier-intro-three-second-tour-consumer)

你可以找一个更长的旅行[here](multi__spring_cloud_contract_verifier_introduction.html#spring-cloud-contract-verifier-intro-three-minute-tour).

#### 在制作人一边

要开始使用Spring Cloud Contract，请将包含以Groovy DSL或YAML表示的 `REST/` 消息传递Contract的文件添加到Contract目录，该目录由 `contractsDslDir` 属性设置.默认情况下，它是 `$rootDir/src/test/resources/contracts` .

然后添加Spring Cloud Contract Verifier依赖项和插件到您的构建文件，如以下示例所示：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-contract-verifier</artifactId>
	<scope>test</scope>
</dependency>
```

以下清单显示了如何添加插件，该插件应该放在文件的build / plugins部分中：

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<version>${spring-cloud-contract.version}</version>
	<extensions>true</extensions>
</plugin>
```

运行 `./mvnw clean install` 会自动生成测试，以验证应用程序是否符合添加的Contract.默认情况下，测试在 `org.springframework.cloud.contract.verifier.tests.` 下生成.

由于Contract描述的功能的实现尚未存在，测试失败.

要使它们通过，您必须添加处理HTTP请求或消息的正确实现.此外，您必须为项目添加自动生成的测试的正确基本测试类.此类由所有自动生成的测试扩展，它应包含运行它们所需的所有设置（例如 `RestAssuredMockMvc` 控制器设置或消息传递测试设置）.

一旦实现和测试基类到位，测试就会通过，并且应用程序和存根工件都将构建并安装在本地Maven存储库中.现在可以合并这些更改，并且可以在在线存储库中发布应用程序和存根工件.

#### 在消费者方面

可以在集成测试中使用 `Spring Cloud Contract Stub Runner` 来获取正在运行的模拟实际服务的WireMock实例或消息传递路由.

为此，请将依赖项添加到 `Spring Cloud Contract Stub Runner` ，如以下示例所示：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
	<scope>test</scope>
</dependency>
```

您可以通过以下两种方式之一获取Maven存储库中安装的Producer端存根：

- By通过运行以下命令检出Producer端存储库并添加Contract并生成存根：

```java
$ cd local-http-server-repo
$ ./mvnw clean install -DskipTests
```

> 正在跳过测试，因为生产环境者方Contract实施尚未到位，因此自动生成的Contract测试失败.

- By从远程存储库获取已存在的生产环境者服务存根.为此，请将存根工件ID和工件存储库URL作为 `Spring Cloud Contract Stub Runner` 属性传递，如以下示例所示：

```java
stubrunner:
ids: 'com.example:http-server-dsl:+:stubs:8080'
repositoryRoot: http://repo.spring.io/libs-snapshot
```

现在，您可以使用 `@AutoConfigureStubRunner` 注释测试类.在注释中，为 `Spring Cloud Contract Stub Runner` 提供 `group-id` 和 `artifact-id` 值以便为您运行协作者的存根，如以下示例所示：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.NONE)
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:stubs:6565"},
		stubsMode = StubRunnerProperties.StubsMode.LOCAL)
public class LoanApplicationServiceTests {
```

> 从在线存储库下载存根时使用 `REMOTE`   `stubsMode` ，在离线工作中使用 `LOCAL` .

现在，在集成测试中，您可以接收预期由协作者服务发出的HTTP响应或消息的存根版本.

### 89.3.2三分钟之旅

这个简短的游览将介绍使用Spring Cloud Contract：

- [the section called “On the Producer Side”](multi__spring_cloud_contract_verifier_introduction.html#spring-cloud-contract-verifier-intro-three-minute-tour-producer)

- [the section called “On the Consumer Side”](multi__spring_cloud_contract_verifier_introduction.html#spring-cloud-contract-verifier-intro-three-minute-tour-consumer)

你可以找到一个更简短的旅游[here](multi__spring_cloud_contract_verifier_introduction.html#spring-cloud-contract-verifier-intro-three-second-tour).

#### 在制作人一边

要开始使用 `Spring Cloud Contract` ，请将包含以Groovy DSL或YAML表示的 `REST/` 消息传递Contract的文件添加到Contract目录，该目录由 `contractsDslDir` 属性设置.默认情况下，它是 `$rootDir/src/test/resources/contracts` .

对于HTTP存根，Contract定义应为给定请求返回何种响应（考虑HTTP方法，URL，Headers，状态代码等）.以下示例显示了Groovy DSL中的HTTP存根协定：

```java
package contracts

org.springframework.cloud.contract.spec.Contract.make {
	request {
		method 'PUT'
		url '/fraudcheck'
		body([
			   "client.id": $(regex('[0-9]{10}')),
			   loanAmount: 99999
		])
		headers {
			contentType('application/json')
		}
	}
	response {
		status OK()
		body([
			   fraudCheckStatus: "FRAUD",
			   "rejection.reason": "Amount too high"
		])
		headers {
			contentType('application/json')
		}
	}
}
```

在YAML中表达的相同Contract将类似于以下示例：

```java
request:
method: PUT
url: /fraudcheck
body:
"client.id": 1234567890
loanAmount: 99999
headers:
Content-Type: application/json
matchers:
body:
- path: $.['client.id']
type: by_regex
value: "[0-9]{10}"
response:
status: 200
body:
fraudCheckStatus: "FRAUD"
"rejection.reason": "Amount too high"
headers:
Content-Type: application/json;charset=UTF-8
```

在消息传递的情况下，您可以定义：

- 可以定义输入和输出消息（考虑发送它和发送的位置，消息正文和Headers）.

- 收到消息后应调用的方法.

- 调用时应触发消息的方法.

以下示例显示了以Groovy DSL表示的Camel消息传递Contract：

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

以下示例显示了在YAML中表达的相同Contract：

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

然后，您可以将Spring Cloud Contract Verifier依赖项和插件添加到构建文件中，如以下示例所示：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-contract-verifier</artifactId>
	<scope>test</scope>
</dependency>
```

以下清单显示了如何添加插件，该插件应该放在文件的build / plugins部分中：

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<version>${spring-cloud-contract.version}</version>
	<extensions>true</extensions>
</plugin>
```

运行 `./mvnw clean install` 会自动生成测试，以验证应用程序是否符合添加的Contract.默认情况下，生成的测试位于 `org.springframework.cloud.contract.verifier.tests.` 之下.

以下示例显示了HTTPContract的自动生成测试示例：

```java
@Test
public void validate_shouldMarkClientAsFraud() throws Exception {
// given:
MockMvcRequestSpecification request = given()
.header("Content-Type", "application/vnd.fraud.v1+json")
.body("{\"client.id\":\"1234567890\",\"loanAmount\":99999}");

// when:
ResponseOptions response = given().spec(request)
.put("/fraudcheck");

// then:
assertThat(response.statusCode()).isEqualTo(200);
assertThat(response.header("Content-Type")).matches("application/vnd.fraud.v1.json.*");
// and:
DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
assertThatJson(parsedJson).field("['fraudCheckStatus']").matches("[A-Z]{5}");
assertThatJson(parsedJson).field("['rejection.reason']").isEqualTo("Amount too high");
}
```

前面的示例使用Spring的 `MockMvc` 来运行测试.这是HTTPContract的默认测试模式.但是，也可以使用JAX-RX客户端和显式HTTP调用. （为此，请将插件的 `testMode` 属性分别更改为 `JAX-RS` 或 `EXPLICIT` .）

除了默认的JUnit之外，您还可以通过将插件 `testFramework` 属性设置为 `Spock` 来使用Spock测试.

> 您现在还可以根据Contract生成WireMock方案，方法是在Contract文件名的开头加一个订单号后跟一个下划线.

以下示例显示了Spock中针对消息传递存根协定的自动生成的测试：

```java
[source,groovy,indent=0]
```

```java
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
```

由于Contract描述的功能的实现尚未存在，测试失败.

要让它们通过，你必须添加正确实现处理HTTP请求或消息.此外，您必须为项目添加自动生成的测试的正确基本测试类.此类由所有自动生成的测试扩展，并应包含运行它们所需的所有设置（例如， `RestAssuredMockMvc` 控制器设置或消息传递测试设置）.

一旦实现和测试基类到位，测试就会通过，并且应用程序和存根工件都将构建并安装在本地Maven存储库中.有关将存根jar安装到本地存储库的信息将显示在日志中，如以下示例所示：

```java
[INFO] --- spring-cloud-contract-maven-plugin:1.0.0.BUILD-SNAPSHOT:generateStubs (default-generateStubs) @ http-server ---
[INFO] Building jar: /some/path/http-server/target/http-server-0.0.1-SNAPSHOT-stubs.jar
[INFO]
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ http-server ---
[INFO] Building jar: /some/path/http-server/target/http-server-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:1.5.5.BUILD-SNAPSHOT:repackage (default) @ http-server ---
[INFO]
[INFO] --- maven-install-plugin:2.5.2:install (default-install) @ http-server ---
[INFO] Installing /some/path/http-server/target/http-server-0.0.1-SNAPSHOT.jar to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT.jar
[INFO] Installing /some/path/http-server/pom.xml to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT.pom
[INFO] Installing /some/path/http-server/target/http-server-0.0.1-SNAPSHOT-stubs.jar to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT-stubs.jar
```

您现在可以合并更改并在联机存储库中发布应用程序和存根工件.

**Docker Project** 

为了在非JVM技术中创建应用程序时能够使用Contract，已创建了 `springcloud/spring-cloud-contract`  Docker镜像.它包含一个自动为HTTPContract生成测试的项目，并在 `EXPLICIT` 测试模式下执行它们.然后，如果测试通过，它将生成Wiremock存根，并可选择将它们发布到工件管理器.要使用该映像，可以将Contract装入 `/contracts` 目录并设置一些环境变量.

#### 在消费者方面

可以在集成测试中使用 `Spring Cloud Contract Stub Runner` 来获取正在运行的模拟实际服务的WireMock实例或消息传递路由.

要开始，请将依赖项添加到 `Spring Cloud Contract Stub Runner` ：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
	<scope>test</scope>
</dependency>
```

您可以通过以下两种方式之一获取Maven存储库中安装的Producer端存根：

- By通过运行以下命令检出Producer端存储库并添加Contract并生成存根：

```java
$ cd local-http-server-repo
$ ./mvnw clean install -DskipTests
```

> 测试被跳过，因为生产环境者方Contract实施尚未到位，因此自动生成的Contract测试失败.

- 从远程存储库获取已存在的生产环境者服务存根.为此，请将存根工件ID和工件存储库URl作为 `Spring Cloud Contract Stub Runner` 属性传递，如以下示例所示：

```java
stubrunner:
ids: 'com.example:http-server-dsl:+:stubs:8080'
repositoryRoot: http://repo.spring.io/libs-snapshot
```

现在，您可以使用 `@AutoConfigureStubRunner` 注释测试类.在注释中，为 `Spring Cloud Contract Stub Runner` 提供 `group-id` 和 `artifact-id` 以便为您运行协作者的存根，如以下示例所示：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.NONE)
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:stubs:6565"},
		stubsMode = StubRunnerProperties.StubsMode.LOCAL)
public class LoanApplicationServiceTests {
```

> 从在线存储库下载存根时使用 `REMOTE`   `stubsMode` ，在离线工作中使用 `LOCAL` .

在集成测试中，您可以接收预期由协作者服务发出的HTTP响应或消息的存根版本.您可以在构建日志中看到类似于以下内容的条目：

```java
2016-07-19 14:22:25.403  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Desired version is + - will try to resolve the latest version
2016-07-19 14:22:25.438  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Resolved version is 0.0.1-SNAPSHOT
2016-07-19 14:22:25.439  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Resolving artifact com.example:http-server:jar:stubs:0.0.1-SNAPSHOT using remote repositories []
2016-07-19 14:22:25.451  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Resolved artifact com.example:http-server:jar:stubs:0.0.1-SNAPSHOT to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT-stubs.jar
2016-07-19 14:22:25.465  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Unpacking stub from JAR [URI: file:/path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT-stubs.jar]
2016-07-19 14:22:25.475  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Unpacked file to [/var/folders/0p/xwq47sq106x1_g3dtv6qfm940000gq/T/contracts100276532569594265]
2016-07-19 14:22:27.737  INFO 41050 --- [           main] o.s.c.c.stubrunner.StubRunnerExecutor    : All stubs are now running RunningStubs [namesAndPorts={com.example:http-server:0.0.1-SNAPSHOT:stubs=8080}]
```

### 89.3.3定义Contract

作为服务的消费者，我们需要定义我们想要实现的目标.我们需要制定我们的期望.这就是我们写Contract的原因.

假设您要发送包含客户公司ID的请求以及它希望从我们借入的金额.您还希望通过PUT方法将其发送到/ fraudcheck URL.

**Groovy DSL.** 

```java
package contracts

org.springframework.cloud.contract.spec.Contract.make {
	request { // (1)
		method 'PUT' // (2)
		url '/fraudcheck' // (3)
		body([ // (4)
			   "client.id": $(regex('[0-9]{10}')),
			   loanAmount: 99999
		])
		headers { // (5)
			contentType('application/json')
		}
	}
	response { // (6)
		status OK() // (7)
		body([ // (8)
			   fraudCheckStatus: "FRAUD",
			   "rejection.reason": "Amount too high"
		])
		headers { // (9)
			contentType('application/json')
		}
	}
}

/*
From the Consumer perspective, when shooting a request in the integration test:

(1) - If the consumer sends a request
(2) - With the "PUT" method
(3) - to the URL "/fraudcheck"
(4) - with the JSON body that
* has a field `client.id` that matches a regular expression `[0-9]{10}`
* has a field `loanAmount` that is equal to `99999`
(5) - with header `Content-Type` equal to `application/json`
(6) - then the response will be sent with
(7) - status equal `200`
(8) - and JSON body equal to
{ "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
(9) - with header `Content-Type` equal to `application/json`

From the Producer perspective, in the autogenerated producer-side test:

(1) - A request will be sent to the producer
(2) - With the "PUT" method
(3) - to the URL "/fraudcheck"
(4) - with the JSON body that
* has a field `client.id` that will have a generated value that matches a regular expression `[0-9]{10}`
* has a field `loanAmount` that is equal to `99999`
(5) - with header `Content-Type` equal to `application/json`
(6) - then the test will assert if the response has been sent with
(7) - status equal `200`
(8) - and JSON body equal to
{ "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
(9) - with header `Content-Type` matching `application/json.*`
*/
```

**YAML.** 

```java
request: # (1)
method: PUT # (2)
url: /fraudcheck # (3)
body: # (4)
"client.id": 1234567890
loanAmount: 99999
headers: # (5)
Content-Type: application/json
matchers:
body:
- path: $.['client.id'] # (6)
type: by_regex
value: "[0-9]{10}"
response: # (7)
status: 200 # (8)
body:  # (9)
fraudCheckStatus: "FRAUD"
"rejection.reason": "Amount too high"
headers: # (10)
Content-Type: application/json;charset=UTF-8

#From the Consumer perspective, when shooting a request in the integration test:
#
#(1) - If the consumer sends a request
#(2) - With the "PUT" method
#(3) - to the URL "/fraudcheck"
#(4) - with the JSON body that
# * has a field `client.id`
# * has a field `loanAmount` that is equal to `99999`
#(5) - with header `Content-Type` equal to `application/json`
#(6) - and a `client.id` json entry matches the regular expression `[0-9]{10}`
#(7) - then the response will be sent with
#(8) - status equal `200`
#(9) - and JSON body equal to
# { "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
#(10) - with header `Content-Type` equal to `application/json`
#
#From the Producer perspective, in the autogenerated producer-side test:
#
#(1) - A request will be sent to the producer
#(2) - With the "PUT" method
#(3) - to the URL "/fraudcheck"
#(4) - with the JSON body that
# * has a field `client.id` `1234567890`
# * has a field `loanAmount` that is equal to `99999`
#(5) - with header `Content-Type` equal to `application/json`
#(7) - then the test will assert if the response has been sent with
#(8) - status equal `200`
#(9) - and JSON body equal to
# { "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
#(10) - with header `Content-Type` equal to `application/json;charset=UTF-8`
```

### 89.3.4客户端

Spring Cloud Contract生成存根，您可以在客户端测试期间使用它们.您将获得一个模拟服务的正在运行的WireMock实例/消息传递路由.您希望使用适当的存根定义来提供该实例.

在某个时间点，您需要向欺诈检测服务发送请求.

```java
ResponseEntity<FraudServiceResponse> response =
		restTemplate.exchange("http://localhost:" + port + "/fraudcheck", HttpMethod.PUT,
				new HttpEntity<>(request, httpHeaders),
				FraudServiceResponse.class);
```

使用 `@AutoConfigureStubRunner` 注释您的测试类.在注释中，为Stub Runner提供组ID和工件ID，以下载协作者的存根.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.NONE)
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:stubs:6565"},
		stubsMode = StubRunnerProperties.StubsMode.LOCAL)
public class LoanApplicationServiceTests {
```

之后，在测试期间，Spring Cloud Contract会自动在Maven存储库中找到存根（模拟真实服务）并在配置（或随机）端口上公开它们.

### 89.3.5服务器端

由于您正在开发存根，因此需要确保它实际上类似于您的具体实现.您不能遇到存根以一种方式运行而应用程序以不同方式运行的情况，尤其是在生产环境环境中.

要确保应用程序的行为与您在存根中定义的方式相同，测试将从您提供的存根中生成.

自动生成的测试或多或少看起来像这样：

```java
@Test
public void validate_shouldMarkClientAsFraud() throws Exception {
// given:
MockMvcRequestSpecification request = given()
.header("Content-Type", "application/vnd.fraud.v1+json")
.body("{\"client.id\":\"1234567890\",\"loanAmount\":99999}");

// when:
ResponseOptions response = given().spec(request)
.put("/fraudcheck");

// then:
assertThat(response.statusCode()).isEqualTo(200);
assertThat(response.header("Content-Type")).matches("application/vnd.fraud.v1.json.*");
// and:
DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
assertThatJson(parsedJson).field("['fraudCheckStatus']").matches("[A-Z]{5}");
assertThatJson(parsedJson).field("['rejection.reason']").isEqualTo("Amount too high");
}
```

## 89.4消费者驱动Contract（CDC）循序渐进指南

考虑欺诈检测和贷款发放流程的示例.业务场景是这样的，我们想向人们发放贷款，但不希望他们从我们这里偷窃.我们系统的当前实施向每个人提供贷款.

假设 `Loan Issuance` 是 `Fraud Detection` 服务器的客户端.在当前的sprint中，我们必须开发一个新功能：如果客户想要借太多钱，那么我们将客户标记为欺诈.

技术评论 - 欺诈检测的 `artifact-id` 为 `http-server` ，而贷款发放的工件ID为 `http-client` ，两者的 `group-id` 为 `com.example` .

社交评论 - 客户和服务器开发团队需要直接沟通并在整个过程中讨论变更. CDC就是沟通.

该[server side code is available here](https://github.com/spring-cloud/spring-cloud-contract/tree/2.0.x/samples/standalone/dsl/http-server)和[the client code here](https://github.com/spring-cloud/spring-cloud-contract/tree/2.0.x/samples/standalone/dsl/http-client).

> 在这种情况下，生产环境者拥有Contract.在物理上，所有Contract都在生产环境者的存储库中.

### 89.4.1技术说明

如果使用 **SNAPSHOT**  /  **Milestone**  /  **Release Candidate** 版本，请在构建中添加以下部分：

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
repositories {
	mavenCentral()
	mavenLocal()
	maven { url "http://repo.spring.io/snapshot" }
	maven { url "http://repo.spring.io/milestone" }
	maven { url "http://repo.spring.io/release" }
}
```

### 89.4.2消费者方（贷款发行）

作为贷款发放服务（欺诈检测服务器的使用者）的开发人员，您可以执行以下步骤：

通过为您的功能编写测试来开始执行TDD.写下缺少的实现.在本地克隆欺诈检测服务存储库.在欺诈检测服务的回购中本地定义Contract.添加Spring Cloud Contract Verifier插件.运行集成测试.提交拉取请求.创建初始实现.接管拉取请求.写下缺少的实现.部署您的应用.在线工作.

**Start doing TDD by writing a test for your feature.** 

```java
@Test
public void shouldBeRejectedDueToAbnormalLoanAmount() {
	// given:
	LoanApplication application = new LoanApplication(new Client("1234567890"),
			99999);
	// when:
	LoanApplicationResult loanApplication = service.loanApplication(application);
	// then:
	assertThat(loanApplication.getLoanApplicationStatus())
			.isEqualTo(LoanApplicationStatus.LOAN_APPLICATION_REJECTED);
	assertThat(loanApplication.getRejectionReason()).isEqualTo("Amount too high");
}
```

假设您已编写了对新功能的测试.如果收到大额贷款申请，系统应拒绝该贷款申请并附上一些说明.

**Write the missing implementation.** 

在某个时间点，您需要向欺诈检测服务发送请求.假设您需要发送包含客户端ID和客户端要借入的金额的请求.您想通过 `PUT` 方法将其发送到 `/fraudcheck`  url.

```java
ResponseEntity<FraudServiceResponse> response =
		restTemplate.exchange("http://localhost:" + port + "/fraudcheck", HttpMethod.PUT,
				new HttpEntity<>(request, httpHeaders),
				FraudServiceResponse.class);
```

为简单起见，欺诈检测服务的端口设置为 `8080` ，应用程序在 `8090` 上运行.

如果此时启动测试，则会中断，因为当前没有服务在端口 `8080` 上运行.

**Clone the Fraud Detection service repository locally.** 

您可以从服务器端Contract开始.为此，您必须先克隆它.

```java
$ git clone https://your-git-server.com/server-side.git local-http-server-repo
```

**Define the contract locally in the repo of Fraud Detection service.** 

作为消费者，您需要定义您想要实现的目标.你需要制定你的期望.为此，请写下以下Contract：

|图片/ important.png |重要|
| ---- | ---- |
|将Contract放在 `src/test/resources/contracts/fraud` 文件夹下.  `fraud` 文件夹很重要，因为生产环境者的测试基类名称引用该文件夹. |

**Groovy DSL.** 

```java
package contracts

org.springframework.cloud.contract.spec.Contract.make {
	request { // (1)
		method 'PUT' // (2)
		url '/fraudcheck' // (3)
		body([ // (4)
			   "client.id": $(regex('[0-9]{10}')),
			   loanAmount: 99999
		])
		headers { // (5)
			contentType('application/json')
		}
	}
	response { // (6)
		status OK() // (7)
		body([ // (8)
			   fraudCheckStatus: "FRAUD",
			   "rejection.reason": "Amount too high"
		])
		headers { // (9)
			contentType('application/json')
		}
	}
}

/*
From the Consumer perspective, when shooting a request in the integration test:

(1) - If the consumer sends a request
(2) - With the "PUT" method
(3) - to the URL "/fraudcheck"
(4) - with the JSON body that
* has a field `client.id` that matches a regular expression `[0-9]{10}`
* has a field `loanAmount` that is equal to `99999`
(5) - with header `Content-Type` equal to `application/json`
(6) - then the response will be sent with
(7) - status equal `200`
(8) - and JSON body equal to
{ "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
(9) - with header `Content-Type` equal to `application/json`

From the Producer perspective, in the autogenerated producer-side test:

(1) - A request will be sent to the producer
(2) - With the "PUT" method
(3) - to the URL "/fraudcheck"
(4) - with the JSON body that
* has a field `client.id` that will have a generated value that matches a regular expression `[0-9]{10}`
* has a field `loanAmount` that is equal to `99999`
(5) - with header `Content-Type` equal to `application/json`
(6) - then the test will assert if the response has been sent with
(7) - status equal `200`
(8) - and JSON body equal to
{ "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
(9) - with header `Content-Type` matching `application/json.*`
*/
```

**YAML.** 

```java
request: # (1)
method: PUT # (2)
url: /fraudcheck # (3)
body: # (4)
"client.id": 1234567890
loanAmount: 99999
headers: # (5)
Content-Type: application/json
matchers:
body:
- path: $.['client.id'] # (6)
type: by_regex
value: "[0-9]{10}"
response: # (7)
status: 200 # (8)
body:  # (9)
fraudCheckStatus: "FRAUD"
"rejection.reason": "Amount too high"
headers: # (10)
Content-Type: application/json;charset=UTF-8

#From the Consumer perspective, when shooting a request in the integration test:
#
#(1) - If the consumer sends a request
#(2) - With the "PUT" method
#(3) - to the URL "/fraudcheck"
#(4) - with the JSON body that
# * has a field `client.id`
# * has a field `loanAmount` that is equal to `99999`
#(5) - with header `Content-Type` equal to `application/json`
#(6) - and a `client.id` json entry matches the regular expression `[0-9]{10}`
#(7) - then the response will be sent with
#(8) - status equal `200`
#(9) - and JSON body equal to
# { "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
#(10) - with header `Content-Type` equal to `application/json`
#
#From the Producer perspective, in the autogenerated producer-side test:
#
#(1) - A request will be sent to the producer
#(2) - With the "PUT" method
#(3) - to the URL "/fraudcheck"
#(4) - with the JSON body that
# * has a field `client.id` `1234567890`
# * has a field `loanAmount` that is equal to `99999`
#(5) - with header `Content-Type` equal to `application/json`
#(7) - then the test will assert if the response has been sent with
#(8) - status equal `200`
#(9) - and JSON body equal to
# { "fraudCheckStatus": "FRAUD", "rejectionReason": "Amount too high" }
#(10) - with header `Content-Type` equal to `application/json;charset=UTF-8`
```

YML合约非常简单.但是，当您查看使用静态类型的Groovy DSL编写的Contract时 - 您可能想知道 `value(client(…), server(…))` 部分是什么.通过使用此表示法，Spring Cloud Contract允许您定义动态的JSON块，URL等部分.如果是标识符或时间戳，则无需对值进行硬编码.您希望允许一些不同的值范围.要启用值范围，可以设置与消费者端的这些值匹配的正则表达式.您可以通过Map表示法或带插值的字符串来提供正文.有关更多信息，请参阅[Chapter 95, Contract DSL](multi_contract-dsl.html)部分.我们强烈建议使用Map表示法！

> 您必须了解Map符号才能设置Contract.请阅读[Groovy docs regarding JSON](http://groovy-lang.org/json.html).

之前显示的Contract是双方之间达成的协议：

- 如果发送HTTP请求全部

`/fraudcheck` endpoints上的
- a  `PUT` 方法，

- a JSON正文 `client.id` 与正则表达式 `[0-9]{10}` 和 `loanAmount` 等于 `99999` 匹配，

- 和 `Content-Type` 标头，其值为 `application/vnd.fraud.v1+json` ，

- 然后将HTTP响应发送给消费者

- has status  `200` ，

- 包含一个JSON正文，其中 `fraudCheckStatus` 字段包含值 `FRAUD` ， `rejectionReason` 字段的值为 `Amount too high` ，

- 和 `Content-Type` 标头，其值为 `application/vnd.fraud.v1+json` .

一旦准备好在集成测试中检查API，就需要在本地安装存根.

**Add the Spring Cloud Contract Verifier plugin.** 

我们可以添加Maven或Gradle插件.在此示例中，您将了解如何添加Maven.首先，添加 `Spring Cloud Contract`  BOM.

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

接下来，添加 `Spring Cloud Contract Verifier`  Maven插件

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

自添加插件后，您将获得 `Spring Cloud Contract Verifier` 功能，这些功能来自提供的Contract：

- generate并运行测试

- produce并安装存根

您不希望生成测试，因为您作为消费者只想使用存根.您需要跳过测试生成和执行.执行时：

```java
$ cd local-http-server-repo
$ ./mvnw clean install -DskipTests
```

在日志中，您会看到以下内容：

```java
[INFO] --- spring-cloud-contract-maven-plugin:1.0.0.BUILD-SNAPSHOT:generateStubs (default-generateStubs) @ http-server ---
[INFO] Building jar: /some/path/http-server/target/http-server-0.0.1-SNAPSHOT-stubs.jar
[INFO]
[INFO] --- maven-jar-plugin:2.6:jar (default-jar) @ http-server ---
[INFO] Building jar: /some/path/http-server/target/http-server-0.0.1-SNAPSHOT.jar
[INFO]
[INFO] --- spring-boot-maven-plugin:1.5.5.BUILD-SNAPSHOT:repackage (default) @ http-server ---
[INFO]
[INFO] --- maven-install-plugin:2.5.2:install (default-install) @ http-server ---
[INFO] Installing /some/path/http-server/target/http-server-0.0.1-SNAPSHOT.jar to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT.jar
[INFO] Installing /some/path/http-server/pom.xml to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT.pom
[INFO] Installing /some/path/http-server/target/http-server-0.0.1-SNAPSHOT-stubs.jar to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT-stubs.jar
```

以下行非常重要：

```java
[INFO] Installing /some/path/http-server/target/http-server-0.0.1-SNAPSHOT-stubs.jar to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT-stubs.jar
```

它确认 `http-server` 的存根已安装在本地存储库中.

**Run the integration tests.** 

为了从自动存根下载的Spring Cloud Contract Stub Runner功能中获益，您必须在您的客户端项目（ `Loan Application service` ）中执行以下操作：

添加 `Spring Cloud Contract`  BOM：

```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-dependencies</artifactId>
			<version>${spring-cloud-release-train.version}</version>
			<type>pom</type>
			<scope>import</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```

将依赖项添加到 `Spring Cloud Contract Stub Runner` ：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
	<scope>test</scope>
</dependency>
```

使用 `@AutoConfigureStubRunner` 注释您的测试类.在注释中，为Stub Runner提供 `group-id` 和 `artifact-id` 以下载协作者的存根. （可选步骤）由于您正在脱机协作者，还可以提供脱机工作开关（ `StubRunnerProperties.StubsMode.LOCAL` ）.

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment=WebEnvironment.NONE)
@AutoConfigureStubRunner(ids = {"com.example:http-server-dsl:+:stubs:6565"},
		stubsMode = StubRunnerProperties.StubsMode.LOCAL)
public class LoanApplicationServiceTests {
```

现在，当您运行测试时，您会看到以下内容：

```java
2016-07-19 14:22:25.403  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Desired version is + - will try to resolve the latest version
2016-07-19 14:22:25.438  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Resolved version is 0.0.1-SNAPSHOT
2016-07-19 14:22:25.439  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Resolving artifact com.example:http-server:jar:stubs:0.0.1-SNAPSHOT using remote repositories []
2016-07-19 14:22:25.451  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Resolved artifact com.example:http-server:jar:stubs:0.0.1-SNAPSHOT to /path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT-stubs.jar
2016-07-19 14:22:25.465  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Unpacking stub from JAR [URI: file:/path/to/your/.m2/repository/com/example/http-server/0.0.1-SNAPSHOT/http-server-0.0.1-SNAPSHOT-stubs.jar]
2016-07-19 14:22:25.475  INFO 41050 --- [           main] o.s.c.c.stubrunner.AetherStubDownloader  : Unpacked file to [/var/folders/0p/xwq47sq106x1_g3dtv6qfm940000gq/T/contracts100276532569594265]
2016-07-19 14:22:27.737  INFO 41050 --- [           main] o.s.c.c.stubrunner.StubRunnerExecutor    : All stubs are now running RunningStubs [namesAndPorts={com.example:http-server:0.0.1-SNAPSHOT:stubs=8080}]
```

此输出表示Stub Runner已找到您的存根并为您的应用启动了服务器组ID  `com.example` ，工件ID  `http-server` ，存根的版本为 `0.0.1-SNAPSHOT` ，端口 `8080` 上的 `stubs` 分类器.

**File a pull request.** 

到目前为止，您所做的是一个迭代过程.您可以使用Contract，在本地安装，并在消费者方面工作，直到Contract按您的意愿工作.

一旦您对结果和测试通过感到满意，就向服务器端发布一个拉取请求.目前，消费者方面的工作已经完成.

### 89.4.3生产环境者方（欺诈检测服务器）

作为欺诈检测服务器（Loan Issuance服务的服务器）的开发人员：

**Create an initial implementation.** 

提醒一下，您可以在此处查看初始实现：

```java
@RequestMapping(value = "/fraudcheck", method = PUT)
public FraudCheckResult fraudCheck(@RequestBody FraudCheck fraudCheck) {
return new FraudCheckResult(FraudCheckStatus.OK, NO_REASON);
}
```

**Take over the pull request.** 

```java
$ git checkout -b contract-change-pr master
$ git pull https://your-git-server.com/server-side-fork.git contract-change-pr
```

您必须添加自动生成的测试所需的依赖项：

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-contract-verifier</artifactId>
	<scope>test</scope>
</dependency>
```

在Maven插件的配置中，传递 `packageWithBaseClasses` 属性

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

|图片/ important.png |重要|
| ---- | ---- |
|此示例通过设置 `packageWithBaseClasses` 属性使用"convention based"命名.这样做意味着最后两个包组合起来构成基本测试类的名称.在我们的案例中，Contract被置于 `src/test/resources/contracts/fraud` 之下.由于您没有从 `contracts` 文件夹开始的两个包，因此只选择一个，应该是 `fraud` .添加 `Base` 后缀并将 `fraud` 大写.这为您提供了 `FraudBase` 测试类名称. |

所有生成的测试都扩展了该类.在那里，您可以设置Spring Context或任何必要的内容.在这种情况下，使用[Rest Assured MVC](http://rest-assured.io/)启动服务器端 `FraudDetectionController` .

```java
package com.example.fraud;

import org.junit.Before;

import io.restassured.module.mockmvc.RestAssuredMockMvc;

public class FraudBase {
	@Before
	public void setup() {
		RestAssuredMockMvc.standaloneSetup(new FraudDetectionController(),
				new FraudStatsController(stubbedStatsProvider()));
	}

	private StatsProvider stubbedStatsProvider() {
		return fraudType -> {
			switch (fraudType) {
			case DRUNKS:
				return 100;
			case ALL:
				return 200;
			}
			return 0;
		};
	}

	public void assertThatRejectionReasonIsNull(Object rejectionReason) {
		assert rejectionReason == null;
	}
}
```

现在，如果你运行 `./mvnw clean install` ，你得到这样的东西：

```java
Results :

Tests in error:
ContractVerifierTest.validate_shouldMarkClientAsFraud:32 » IllegalState Parsed...
```

发生此错误的原因是您有一个新的Contract，从中生成测试并且由于您尚未实现该功能而失败.自动生成的测试如下所示：

```java
@Test
public void validate_shouldMarkClientAsFraud() throws Exception {
// given:
MockMvcRequestSpecification request = given()
.header("Content-Type", "application/vnd.fraud.v1+json")
.body("{\"client.id\":\"1234567890\",\"loanAmount\":99999}");

// when:
ResponseOptions response = given().spec(request)
.put("/fraudcheck");

// then:
assertThat(response.statusCode()).isEqualTo(200);
assertThat(response.header("Content-Type")).matches("application/vnd.fraud.v1.json.*");
// and:
DocumentContext parsedJson = JsonPath.parse(response.getBody().asString());
assertThatJson(parsedJson).field("['fraudCheckStatus']").matches("[A-Z]{5}");
assertThatJson(parsedJson).field("['rejection.reason']").isEqualTo("Amount too high");
}
```

如果您使用了Groovy DSL，您可以看到， `value(consumer(…), producer(…))` 块中存在的Contract的所有 `producer()` 部分都被注入到测试中.在使用YAML的情况下，同样适用于 `response` 的 `matchers` 部分.

请注意，在生产环境者方面，您也在进行TDD.期望以测试的形式表达.此测试使用Contract中定义的URL，Headers和正文向我们自己的应用程序发送请求.它还期望在响应中精确定义的值.换句话说，你有 `red` ， `green` 和 `refactor` 的 `red` 部分.是时候将 `red` 转换为 `green` .

**Write the missing implementation.** 

因为您知道预期的输入和预期输出，所以可以编写缺少的实现：

```java
@RequestMapping(value = "/fraudcheck", method = PUT)
public FraudCheckResult fraudCheck(@RequestBody FraudCheck fraudCheck) {
if (amountGreaterThanThreshold(fraudCheck)) {
	return new FraudCheckResult(FraudCheckStatus.FRAUD, AMOUNT_TOO_HIGH);
}
return new FraudCheckResult(FraudCheckStatus.OK, NO_REASON);
}
```

再次执行 `./mvnw clean install` 时，测试通过.由于 `Spring Cloud Contract Verifier` 插件将测试添加到 `generated-test-sources` ，您实际上可以从IDE运行这些测试.

**Deploy your app.** 

完成工作后，您可以部署更改.首先，合并分支：

```java
$ git checkout master
$ git merge --no-ff contract-change-pr
$ git push origin master
```

您的CI可能会运行类似 `./mvnw clean deploy` 的内容，它会发布应用程序和存根工件.

### 89.4.4消费者方（贷款发行）最后一步

作为贷款发放服务（欺诈检测服务器的消费者）的开发者：

**Merge branch to master.** 

```java
$ git checkout master
$ git merge --no-ff contract-change-pr
```

**Work online.** 

现在，您可以禁用Spring Cloud Contract Stub Runner的脱机工作，并指明存储库与存根的位置.此时，服务器端的存根将自动从Nexus / Artifactory下载.您可以将 `stubsMode` 的值设置为 `REMOTE` .以下代码显示了通过更改属性实现相同功能的示例.

```java
stubrunner:
ids: 'com.example:http-server-dsl:+:stubs:8080'
repositoryRoot: http://repo.spring.io/libs-snapshot
```

而已！

## 89.5依赖关系

添加依赖项的最佳方法是使用正确的 `starter` 依赖项.

对于 `stub-runner` ，请使用 `spring-cloud-starter-stub-runner` .使用插件时，请添加 `spring-cloud-starter-contract-verifier` .

## 89.6其他链接

以下是与Spring Cloud Contract Verifier和Stub Runner相关的一些资源.请注意，有些可能已过时，因为Spring Cloud Contract Verifier项目正在不断发展.

### 89.6.1 Spring Cloud Contract视频

您可以查看Warsaw JUG关于Spring Cloud Contract的视频：

### 89.6.2读物

- [Slides from Marcin Grzejszczak’s talk about Accurest](http://www.slideshare.net/MarcinGrzejszczak/stick-to-the-rules-consumer-driven-contracts-201507-confitura)

- [Accurest related articles from Marcin Grzejszczak’s blog](http://toomuchcoding.com/blog/categories/accurest/)

- [Spring Cloud Contract related articles from Marcin Grzejszczak’s blog](http://toomuchcoding.com/blog/categories/spring-cloud-contract/)

- [Groovy docs regarding JSON](http://groovy-lang.org/json.html)

## 89.7样品

你可以在[samples](https://github.com/spring-cloud-samples/spring-cloud-contract-samples)找到一些样品.


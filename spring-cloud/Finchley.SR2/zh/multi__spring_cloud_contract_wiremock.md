## 98. Spring Cloud Contract WireMock

Spring Cloud Contract WireMock模块允许您在Spring Boot应用程序中使用[WireMock](http://wiremock.org).查看[samples](https://github.com/spring-cloud/spring-cloud-contract/tree/master/samples)了解更多详情.

如果您有一个使用Tomcat作为嵌入式服务器的Spring Boot应用程序（默认使用 `spring-boot-starter-web` ），您可以将 `spring-cloud-starter-contract-stub-runner` 添加到类路径并添加 `@AutoConfigureWireMock` ，以便能够在测试中使用Wiremock. Wiremock作为存根服务器运行，您可以使用Java API或静态JSON声明来注册存根行为，作为测试的一部分.以下代码显示了一个示例：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureWireMock(port = 0)
public class WiremockForDocsTests {
	// A service that calls out over HTTP
	@Autowired private Service service;

	// Using the WireMock APIs in the normal way:
	@Test
	public void contextLoads() throws Exception {
		// Stubbing WireMock
		stubFor(get(urlEqualTo("/resource"))
				.willReturn(aResponse().withHeader("Content-Type", "text/plain").withBody("Hello World!")));
		// We're asserting if WireMock responded properly
		assertThat(this.service.go()).isEqualTo("Hello World!");
	}

}
```

要在不同的端口上启动存根服务器，请使用（例如） `@AutoConfigureWireMock(port=9999)` .对于随机端口，请使用 `0` 的值.可以使用"wiremock.server.port"属性将存根服务器端口绑定在测试应用程序上下文中.使用 `@AutoConfigureWireMock` 将一个类型为 `WiremockConfiguration` 的bean添加到您的测试应用程序上下文中，它将被缓存在具有相同上下文的方法和类之间，与Spring集成测试相同.

## 98.1自动注册存根

如果使用 `@AutoConfigureWireMock` ，它会从文件系统或类路径中注册WireMock JSON存根（默认情况下，从 `file:src/test/resources/mappings` 开始）.您可以使用注释中的 `stubs` 属性自定义位置，该属性可以是Ant样式的资源模式或目录.如果是目录，则追加 `*/.json` .以下代码显示了一个示例：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureWireMock(stubs="classpath:/stubs")
public class WiremockImportApplicationTests {

	@Autowired
	private Service service;

	@Test
	public void contextLoads() throws Exception {
		assertThat(this.service.go()).isEqualTo("Hello World!");
	}

}
```

> 实际上，WireMock始终从存储属性中的自定义位置加载 `src/test/resources/mappings`   **as well as** 的映射.要更改此行为，您还可以指定文件根目录，如本文档的下一部分所述.

## 98.2使用文件指定存根体

WireMock可以从类路径或文件系统上的文件中读取响应主体.在这种情况下，您可以在JSON DSL中看到响应具有 `bodyFileName` 而不是（文字） `body` .相对于根目录（默认情况下为 `src/test/resources/__files` ）解析文件.要自定义此位置，可以将 `@AutoConfigureWireMock` 注释中的 `files` 属性设置为父目录的位置（换句话说， `__files` 是子目录）.您可以使用Spring资源表示法来引用 `file:…` 或 `classpath:…` 位置.不支持通用URL.可以给出值列表，在这种情况下，WireMock会在需要查找响应主体时解析存在的第一个文件.

> 当您配置 `files`  root时，它还会影响存根的自动加载，因为它们来自名为"mappings"的子目录中的根位置.  `files` 的值对从 `stubs` 属性显式加载的存根没有影响.

## 98.3替代方案：使用JUnit规则

对于更传统的WireMock体验，您可以使用JUnit  `@Rules` 来启动和停止服务器.为此，请使用 `WireMockSpring`  convenience类获取 `Options` 实例，如以下示例所示：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class WiremockForDocsClassRuleTests {

	// Start WireMock on some dynamic port
	// for some reason `dynamicPort()` is not working properly
	@ClassRule
	public static WireMockClassRule wiremock = new WireMockClassRule(
			WireMockSpring.options().dynamicPort());
	// A service that calls out over HTTP to localhost:${wiremock.port}
	@Autowired
	private Service service;

	// Using the WireMock APIs in the normal way:
	@Test
	public void contextLoads() throws Exception {
		// Stubbing WireMock
		wiremock.stubFor(get(urlEqualTo("/resource"))
				.willReturn(aResponse().withHeader("Content-Type", "text/plain").withBody("Hello World!")));
		// We're asserting if WireMock responded properly
		assertThat(this.service.go()).isEqualTo("Hello World!");
	}

}
```

`@ClassRule` 表示服务器在运行此类中的所有方法后关闭.

## 98.4 Rest模板的轻松SSL验证

WireMock允许您使用“https”URL协议存根“安全”服务器.如果您的应用程序想要在集成测试中联系该存根服务器，则会发现SSL证书无效（自安装证书的常见问题）.最好的选择通常是重新配置客户端以使用“http”.如果这不是一个选项，您可以要求Spring配置忽略SSL验证错误的HTTP客户端（当然，仅对测试执行此操作）.

为了最大限度地减少这项工作，您需要在应用程序中使用Spring Boot  `RestTemplateBuilder` ，如以下示例所示：

```java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
	return builder.build();
}
```

您需要 `RestTemplateBuilder` ，因为构建器通过回调进行初始化，因此可以在客户端设置SSL验证.如果您使用 `@AutoConfigureWireMock` 注释或存根运行器，则会在测试中自动执行此操作.如果使用JUnit  `@Rule` 方法，则还需要添加 `@AutoConfigureHttpClient` 注释，如以下示例所示：

```java
@RunWith(SpringRunner.class)
@SpringBootTest("app.baseUrl=https://localhost:6443")
@AutoConfigureHttpClient
public class WiremockHttpsServerApplicationTests {

	@ClassRule
	public static WireMockClassRule wiremock = new WireMockClassRule(
			WireMockSpring.options().httpsPort(6443));
...
}
```

如果您使用 `spring-boot-starter-test` ，则在类路径上有Apache HTTP客户端，它由 `RestTemplateBuilder` 选择并配置为忽略SSL错误.如果使用默认的 `java.net` 客户端，则不需要注释（但不会造成任何伤害）.目前没有其他客户端的支持，但可能会在将来的版本中添加.

要禁用自定义 `RestTemplateBuilder` ，请将 `wiremock.rest-template-ssl-enabled` 属性设置为 `false` .

## 98.5 WireMock和Spring MVC模拟

Spring Cloud Contract提供了一个便利类，可以将JSON WireMock存根加载到Spring  `MockRestServiceServer` 中.以下代码显示了一个示例：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.NONE)
public class WiremockForDocsMockServerApplicationTests {

	@Autowired
	private RestTemplate restTemplate;

	@Autowired
	private Service service;

	@Test
	public void contextLoads() throws Exception {
		// will read stubs classpath
		MockRestServiceServer server = WireMockRestServiceServer.with(this.restTemplate)
				.baseUrl("http://example.org").stubs("classpath:/stubs/resource.json")
				.build();
		// We're asserting if WireMock responded properly
		assertThat(this.service.go()).isEqualTo("Hello World");
		server.verify();
	}
}
```

`baseUrl` 值前置于所有人模拟调用， `stubs()` 方法将存根路径资源模式作为参数.在前面的示例中， `/stubs/resource.json` 中定义的存根被加载到模拟服务器中.如果要求 `RestTemplate` 访问 `http://example.org/` ，则会在该URL处声明响应.可以指定多个存根模式，每个存根模式可以是一个目录（对于所有".json"的递归列表），固定文件名（如上例所示）或Ant样式模式. JSON格式是普通的WireMock格式，您可以在[WireMock website](http://wiremock.org/docs/stubbing/)中阅读.

目前，Spring Cloud Contract Verifier支持Tomcat，Jetty和Undertow作为Spring Boot嵌入式服务器，而Wiremock本身对特定版本的Jetty（目前为9.2）具有“本机”支持.要使用本机Jetty，您需要添加本机Wiremock依赖项并排除Spring Boot容器（如果有）.

## 98.6自定义WireMock配置

您可以注册 `org.springframework.cloud.contract.wiremock.WireMockConfigurationCustomizer` 类型的bean以自定义WireMock配置（例如，添加自定义变换器）.例：

```java
@Bean WireMockConfigurationCustomizer optionsCustomizer() {
			return new WireMockConfigurationCustomizer() {
				@Override public void customize(WireMockConfiguration options) {
// perform your customization here
				}
			};
		}
```

## 98.7使用REST文档生成存根

[Spring REST Docs](https://projects.spring.io/spring-restdocs)可用于为使用Spring MockMvc或 `WebTestClient` 或Rest Assured的HTTP API生成文档（例如，以Asciidoctor格式）.在为API生成文档的同时，您还可以使用Spring Cloud Contract WireMock生成WireMock存根.为此，请编写正常的REST Docs测试用例并使用 `@AutoConfigureRestDocs` 在REST Docs输出目录中自动生成存根.以下代码显示了使用 `MockMvc` 的示例：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureRestDocs(outputDir = "target/snippets")
@AutoConfigureMockMvc
public class ApplicationTests {

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void contextLoads() throws Exception {
		mockMvc.perform(get("/resource"))
				.andExpect(content().string("Hello World"))
				.andDo(document("resource"));
	}
}
```

此测试在"target/snippets/stubs/resource.json"生成WireMock存根.它匹配"/resource"路径的所有GET请求.  `WebTestClient` （用于测试Spring WebFlux应用程序）的相同示例如下所示：

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureRestDocs(outputDir = "target/snippets")
@AutoConfigureWebTestClient
public class ApplicationTests {

	@Autowired
	private WebTestClient client;

	@Test
	public void contextLoads() throws Exception {
		client.get().uri("/resource").exchange()
				.expectBody(String.class).isEqualTo("Hello World")
				.consumeWith(document("resource"));
	}
}
```

在没有任何其他配置的情况下，这些测试会创建一个存根，其中包含HTTP方法的请求匹配器以及除“host”和“content-length”之外的所有标头.为了更精确地匹配请求（例如，匹配POST或PUT的主体），我们需要显式创建请求匹配器.这样做有两个影响：

- 创建仅按指定方式匹配的存根.

- 断言测试用例中的请求也匹配相同的条件.

此功能的主要入口点是 `WireMockRestDocs.verify()` ，可用作 `document()` 便捷方法的替代，如以下示例所示：

```java
import static org.springframework.cloud.contract.wiremock.restdocs.WireMockRestDocs.verify;
```

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureRestDocs(outputDir = "target/snippets")
@AutoConfigureMockMvc
public class ApplicationTests {

	@Autowired
	private MockMvc mockMvc;

	@Test
	public void contextLoads() throws Exception {
		mockMvc.perform(post("/resource")
.content("{\"id\":\"123456\",\"message\":\"Hello World\"}"))
				.andExpect(status().isOk())
				.andDo(verify().jsonPath("$.id")
.stub("resource"));
	}
}
```

此合约指定任何带有"id"字段的有效POST都会收到此测试中定义的响应.您可以将对 `.jsonPath()` 的调用链接在一起以添加其他匹配器.如果JSON Path不熟悉，[JayWay documentation](https://github.com/jayway/JsonPath)可以帮助您加快速度.此测试的 `WebTestClient` 版本具有类似的 `verify()` 静态助手，您可以将其插入到同一位置.

您还可以使用WireMock API来验证请求是否与创建的存根匹配，而不是 `jsonPath` 和 `contentType` 便捷方法，如以下示例所示：

```java
@Test
public void contextLoads() throws Exception {
	mockMvc.perform(post("/resource")
.content("{\"id\":\"123456\",\"message\":\"Hello World\"}"))
			.andExpect(status().isOk())
			.andDo(verify()
					.wiremock(WireMock.post(
						urlPathEquals("/resource"))
						.withRequestBody(matchingJsonPath("$.id"))
.stub("post-resource"));
}
```

WireMock API很丰富.您可以通过正则表达式和JSON路径匹配Headers，查询参数和请求正文.这些功能可用于创建具有更多参数的存根.上面的示例生成类似于以下示例的存根：

**post-resource.json.** 

```java
{
"request" : {
"url" : "/resource",
"method" : "POST",
"bodyPatterns" : [ {
"matchesJsonPath" : "$.id"
}]
},
"response" : {
"status" : 200,
"body" : "Hello World",
"headers" : {
"X-Application-Context" : "application:-1",
"Content-Type" : "text/plain"
}
}
}
```

> 您可以使用 `wiremock()` 方法或 `jsonPath()` 和 `contentType()` 方法创建请求匹配器，但不能同时使用这两种方法.

在消费者方面，您可以在类路径中提供本节前面生成的 `resource.json` （例如，通过<< publishing-stubs-as-jars].之后，您可以使用WireMock以多种不同方式创建存根，包括使用 `@AutoConfigureWireMock(stubs="classpath:resource.json")` ，如本文档前面所述.

## 98.8使用REST文档生成Contract

您还可以使用Spring REST Docs生成Spring Cloud Contract DSL文件和文档.如果您与Spring Cloud WireMock结合使用，则可以同时获得Contract和存根.

你为什么要使用这个功能？社区中的一些人询问了他们想要转向基于DSL的Contract定义的情况，但他们已经有很多Spring MVC测试.使用此功能可以生成合约文件，稍后可以修改这些文件并移动到文件夹（在配置中定义），以便插件找到它们.

> 您可能想知道为什么这个功能在WireMock模块中.功能就在那里，因为生成Contract和存根是有意义的.

考虑以下测试：

```java
this.mockMvc.perform(post("/foo")
					.accept(MediaType.APPLICATION_PDF)
					.accept(MediaType.APPLICATION_JSON)
					.contentType(MediaType.APPLICATION_JSON)
					.content("{\"foo\": 23, \"bar\" : \"baz\" }"))
				.andExpect(status().isOk())
				.andExpect(content().string("bar"))
				// first WireMock
				.andDo(WireMockRestDocs.verify()
						.jsonPath("$[?(@.foo >= 20)]")
						.jsonPath("$[?(@.bar in ['baz','bazz','bazzz'])]")
						.contentType(MediaType.valueOf("application/json"))
						.stub("shouldGrantABeerIfOldEnough"))
				// then Contract DSL documentation
				.andDo(document("index", Spring CloudContractRestDocs.dslContract()));
```

上述测试创建了上一节中提供的存根，生成了Contract和文档文件.

该合约名为 `index.groovy` ，可能类似于以下示例：

```java
import org.springframework.cloud.contract.spec.Contract

Contract.make {
request {
method 'POST'
url '/foo'
body('''
{"foo": 23 }
''')
headers {
header('''Accept''', '''application/json''')
header('''Content-Type''', '''application/json''')
}
}
response {
status OK()
body('''
bar
''')
headers {
header('''Content-Type''', '''application/json;charset=UTF-8''')
header('''Content-Length''', '''3''')
}
testMatchers {
jsonPath('$[?(@.foo >= 20)]', byType())
}
}
}
```

生成的文档（在本例中为Asciidoc格式）包含格式化Contract.该文件的位置为 `index/dsl-contract.adoc` .


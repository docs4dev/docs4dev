## 98. Spring Cloud Contract WireMock

The Spring Cloud Contract WireMock modules let you use [WireMock](http://wiremock.org) in a Spring Boot application. Check out the [samples](https://github.com/spring-cloud/spring-cloud-contract/tree/master/samples) for more details.

If you have a Spring Boot application that uses Tomcat as an embedded server (which is the default with  `spring-boot-starter-web` ), you can add  `spring-cloud-starter-contract-stub-runner`  to your classpath and add  `@AutoConfigureWireMock`  in order to be able to use Wiremock in your tests. Wiremock runs as a stub server and you can register stub behavior using a Java API or via static JSON declarations as part of your test. The following code shows an example:

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

To start the stub server on a different port use (for example),  `@AutoConfigureWireMock(port=9999)` . For a random port, use a value of  `0` . The stub server port can be bound in the test application context with the "wiremock.server.port" property. Using  `@AutoConfigureWireMock`  adds a bean of type  `WiremockConfiguration`  to your test application context, where it will be cached in between methods and classes having the same context, the same as for Spring integration tests.

## 98.1 Registering Stubs Automatically

If you use  `@AutoConfigureWireMock` , it registers WireMock JSON stubs from the file system or classpath (by default, from  `file:src/test/resources/mappings` ). You can customize the locations using the  `stubs`  attribute in the annotation, which can be an Ant-style resource pattern or a directory. In the case of a directory,  `*/.json`  is appended. The following code shows an example:

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

> Actually, WireMock always loads mappings from  `src/test/resources/mappings`   **as well as**  the custom locations in the stubs attribute. To change this behavior, you can also specify a files root as described in the next section of this document.

## 98.2 Using Files to Specify the Stub Bodies

WireMock can read response bodies from files on the classpath or the file system. In that case, you can see in the JSON DSL that the response has a  `bodyFileName`  instead of a (literal)  `body` . The files are resolved relative to a root directory (by default,  `src/test/resources/__files` ). To customize this location you can set the  `files`  attribute in the  `@AutoConfigureWireMock`  annotation to the location of the parent directory (in other words,  `__files`  is a subdirectory). You can use Spring resource notation to refer to  `file:…`  or  `classpath:…`  locations. Generic URLs are not supported. A list of values can be given, in which case WireMock resolves the first file that exists when it needs to find a response body.

> When you configure the  `files`  root, it also affects the automatic loading of stubs, because they come from the root location in a subdirectory called "mappings". The value of  `files`  has no effect on the stubs loaded explicitly from the  `stubs`  attribute.

## 98.3 Alternative: Using JUnit Rules

For a more conventional WireMock experience, you can use JUnit  `@Rules`  to start and stop the server. To do so, use the  `WireMockSpring`  convenience class to obtain an  `Options`  instance, as shown in the following example:

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

The  `@ClassRule`  means that the server shuts down after all the methods in this class have been run.

## 98.4 Relaxed SSL Validation for Rest Template

WireMock lets you stub a "secure" server with an "https" URL protocol. If your application wants to contact that stub server in an integration test, it will find that the SSL certificates are not valid (the usual problem with self-installed certificates). The best option is often to re-configure the client to use "http". If that’s not an option, you can ask Spring to configure an HTTP client that ignores SSL validation errors (do so only for tests, of course).

To make this work with minimum fuss, you need to be using the Spring Boot  `RestTemplateBuilder`  in your app, as shown in the following example:

```java
@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
	return builder.build();
}
```

You need  `RestTemplateBuilder`  because the builder is passed through callbacks to initialize it, so the SSL validation can be set up in the client at that point. This happens automatically in your test if you are using the  `@AutoConfigureWireMock`  annotation or the stub runner. If you use the JUnit  `@Rule`  approach, you need to add the  `@AutoConfigureHttpClient`  annotation as well, as shown in the following example:

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

If you are using  `spring-boot-starter-test` , you have the Apache HTTP client on the classpath and it is selected by the  `RestTemplateBuilder`  and configured to ignore SSL errors. If you use the default  `java.net`  client, you do not need the annotation (but it won’t do any harm). There is no support currently for other clients, but it may be added in future releases.

To disable the custom  `RestTemplateBuilder` , set the  `wiremock.rest-template-ssl-enabled`  property to  `false` .

## 98.5 WireMock and Spring MVC Mocks

Spring Cloud Contract provides a convenience class that can load JSON WireMock stubs into a Spring  `MockRestServiceServer` . The following code shows an example:

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

The  `baseUrl`  value is prepended to all mock calls, and the  `stubs()`  method takes a stub path resource pattern as an argument. In the preceding example, the stub defined at  `/stubs/resource.json`  is loaded into the mock server. If the  `RestTemplate`  is asked to visit  `http://example.org/` , it gets the responses as being declared at that URL. More than one stub pattern can be specified, and each one can be a directory (for a recursive list of all ".json"), a fixed filename (as in the example above), or an Ant-style pattern. The JSON format is the normal WireMock format, which you can read about in the [WireMock website](http://wiremock.org/docs/stubbing/).

Currently, the Spring Cloud Contract Verifier supports Tomcat, Jetty, and Undertow as Spring Boot embedded servers, and Wiremock itself has "native" support for a particular version of Jetty (currently 9.2). To use the native Jetty, you need to add the native Wiremock dependencies and exclude the Spring Boot container (if there is one).

## 98.6 Customization of WireMock configuration

You can register a bean of  `org.springframework.cloud.contract.wiremock.WireMockConfigurationCustomizer`  type in order to customize the WireMock configuration (e.g. add custom transformers). Example:

```java
@Bean WireMockConfigurationCustomizer optionsCustomizer() {
			return new WireMockConfigurationCustomizer() {
				@Override public void customize(WireMockConfiguration options) {
// perform your customization here
				}
			};
		}
```

## 98.7 Generating Stubs using REST Docs

[Spring REST Docs](https://projects.spring.io/spring-restdocs) can be used to generate documentation (for example in Asciidoctor format) for an HTTP API with Spring MockMvc or  `WebTestClient`  or Rest Assured. At the same time that you generate documentation for your API, you can also generate WireMock stubs by using Spring Cloud Contract WireMock. To do so, write your normal REST Docs test cases and use  `@AutoConfigureRestDocs`  to have stubs be automatically generated in the REST Docs output directory. The following code shows an example using  `MockMvc` :

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

This test generates a WireMock stub at "target/snippets/stubs/resource.json". It matches all GET requests to the "/resource" path. The same example with  `WebTestClient`  (used for testing Spring WebFlux applications) would look like this:

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

Without any additional configuration, these tests create a stub with a request matcher for the HTTP method and all headers except "host" and "content-length". To match the request more precisely (for example, to match the body of a POST or PUT), we need to explicitly create a request matcher. Doing so has two effects:

- Creating a stub that matches only in the way you specify.

- Asserting that the request in the test case also matches the same conditions.

The main entry point for this feature is  `WireMockRestDocs.verify()` , which can be used as a substitute for the  `document()`  convenience method, as shown in the following example:

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

This contract specifies that any valid POST with an "id" field receives the response defined in this test. You can chain together calls to  `.jsonPath()`  to add additional matchers. If JSON Path is unfamiliar, The [JayWay documentation](https://github.com/jayway/JsonPath) can help you get up to speed. The  `WebTestClient`  version of this test has a similar  `verify()`  static helper that you insert in the same place.

Instead of the  `jsonPath`  and  `contentType`  convenience methods, you can also use the WireMock APIs to verify that the request matches the created stub, as shown in the following example:

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

The WireMock API is rich. You can match headers, query parameters, and request body by regex as well as by JSON path. These features can be used to create stubs with a wider range of parameters. The above example generates a stub resembling the following example:

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

> You can use either the  `wiremock()`  method or the  `jsonPath()`  and  `contentType()`  methods to create request matchers, but you can’t use both approaches.

On the consumer side, you can make the  `resource.json`  generated earlier in this section available on the classpath (by <<publishing-stubs-as-jars], for example). After that, you can create a stub using WireMock in a number of different ways, including by using  `@AutoConfigureWireMock(stubs="classpath:resource.json")` , as described earlier in this document.

## 98.8 Generating Contracts by Using REST Docs

You can also generate Spring Cloud Contract DSL files and documentation with Spring REST Docs. If you do so in combination with Spring Cloud WireMock, you get both the contracts and the stubs.

Why would you want to use this feature? Some people in the community asked questions about a situation in which they would like to move to DSL-based contract definition, but they already have a lot of Spring MVC tests. Using this feature lets you generate the contract files that you can later modify and move to folders (defined in your configuration) so that the plugin finds them.

> You might wonder why this functionality is in the WireMock module. The functionality is there because it makes sense to generate both the contracts and the stubs.

Consider the following test:

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
				.andDo(document("index", SpringCloudContractRestDocs.dslContract()));
```

The preceding test creates the stub presented in the previous section, generating both the contract and a documentation file.

The contract is called  `index.groovy`  and might look like the following example:

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

The generated document (formatted in Asciidoc in this case) contains a formatted contract. The location of this file would be  `index/dsl-contract.adoc` .


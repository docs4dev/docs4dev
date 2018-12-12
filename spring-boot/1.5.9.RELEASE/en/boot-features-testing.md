## 45. Testing

Spring Boot provides a number of utilities and annotations to help when testing your application. Test support is provided by two modules:  `spring-boot-test`  contains core items, and  `spring-boot-test-autoconfigure`  supports auto-configuration for tests.

Most developers use the  `spring-boot-starter-test`  “Starter”, which imports both Spring Boot test modules as well as JUnit, AssertJ, Hamcrest, and a number of other useful libraries.

## 45.1 Test Scope Dependencies

The  `spring-boot-starter-test`  “Starter” (in the  `test`   `scope` ) contains the following provided libraries:

- [JUnit](http://junit.org): The de-facto standard for unit testing Java applications.

- [Spring Test](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#integration-testing) & Spring Boot Test: Utilities and integration test support for Spring Boot applications.

- [AssertJ](https://joel-costigliola.github.io/assertj/): A fluent assertion library.

- [Hamcrest](http://hamcrest.org/JavaHamcrest/): A library of matcher objects (also known as constraints or predicates).

- [Mockito](http://mockito.org/): A Java mocking framework.

- [JSONassert](https://github.com/skyscreamer/JSONassert): An assertion library for JSON.

- [JsonPath](https://github.com/jayway/JsonPath): XPath for JSON.

We generally find these common libraries to be useful when writing tests. If these libraries do not suit your needs, you can add additional test dependencies of your own.

## 45.2 Testing Spring Applications

One of the major advantages of dependency injection is that it should make your code easier to unit test. You can instantiate objects by using the  `new`  operator without even involving Spring. You can also use mock objects instead of real dependencies.

Often, you need to move beyond unit testing and start integration testing (with a Spring  `ApplicationContext` ). It is useful to be able to perform integration testing without requiring deployment of your application or needing to connect to other infrastructure.

The Spring Framework includes a dedicated test module for such integration testing. You can declare a dependency directly to  `org.springframework:spring-test`  or use the  `spring-boot-starter-test`  “Starter” to pull it in transitively.

If you have not used the  `spring-test`  module before, you should start by reading the [relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testing) of the Spring Framework reference documentation.

## 45.3 Testing Spring Boot Applications

A Spring Boot application is a Spring  `ApplicationContext` , so nothing very special has to be done to test it beyond what you would normally do with a vanilla Spring context.

> External properties, logging, and other features of Spring Boot are installed in the context by default only if you use  `SpringApplication`  to create it.

Spring Boot provides a  `@SpringBootTest`  annotation, which can be used as an alternative to the standard  `spring-test`   `@ContextConfiguration`  annotation when you need Spring Boot features. The annotation works by [creating the ApplicationContext used in your tests through SpringApplication](boot-features-testing.html#boot-features-testing-spring-boot-applications-detecting-config). In addition to  `@SpringBootTest`  a number of other annotations are also provided for [testing more specific slices](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests) of an application.

> If you are using JUnit 4, don’t forget to also add  `@RunWith(SpringRunner.class)`  to your test, otherwise the annotations will be ignored. If you are using JUnit 5, there’s no need to add the equivalent  `@ExtendWith(SpringExtension)`  as  `@SpringBootTest`  and the other  `@…Test`  annotations are already annotated with it.

By default,  `@SpringBootTest`  will not start a server. You can use the  `webEnvironment`  attribute of  `@SpringBootTest`  to further refine how your tests run:

-  `MOCK` (Default) : Loads a web  `ApplicationContext`  and provides a mock web environment. Embedded servers are not started when using this annotation. If a web environment is not available on your classpath, this mode transparently falls back to creating a regular non-web  `ApplicationContext` . It can be used in conjunction with [@AutoConfigureMockMvc or @AutoConfigureWebTestClient](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-mock-environment) for mock-based testing of your web application.

-  `RANDOM_PORT` : Loads a  `WebServerApplicationContext`  and provides a real web environment. Embedded servers are started and listen on a random port.

-  `DEFINED_PORT` : Loads a  `WebServerApplicationContext`  and provides a real web environment. Embedded servers are started and listen on a defined port (from your  `application.properties` ) or on the default port of  `8080` .

-  `NONE` : Loads an  `ApplicationContext`  by using  `SpringApplication`  but does not provide any web environment (mock or otherwise).

> If your test is  `@Transactional` , it rolls back the transaction at the end of each test method by default. However, as using this arrangement with either  `RANDOM_PORT`  or  `DEFINED_PORT`  implicitly provides a real servlet environment, the HTTP client and server run in separate threads and, thus, in separate transactions. Any transaction initiated on the server does not roll back in this case.

>  `@SpringBootTest`  with  `webEnvironment = WebEnvironment.RANDOM_PORT`  will also start the management server on a separate random port if your application uses a different port for the management server.

### 45.3.1 Detecting Web Application Type

If Spring MVC is available, a regular MVC-based application context is configured. If you have only Spring WebFlux, we’ll detect that and configure a WebFlux-based application context instead.

If both are present, Spring MVC takes precedence. If you want to test a reactive web application in this scenario, you must set the  `spring.main.web-application-type`  property:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(properties = "spring.main.web-application-type=reactive")
public class MyWebFluxTests { ... }
```

### 45.3.2 Detecting Test Configuration

If you are familiar with the Spring Test Framework, you may be used to using  `@ContextConfiguration(classes=…)`  in order to specify which Spring  `@Configuration`  to load. Alternatively, you might have often used nested  `@Configuration`  classes within your test.

When testing Spring Boot applications, this is often not required. Spring Boot’s  `@*Test`  annotations search for your primary configuration automatically whenever you do not explicitly define one.

The search algorithm works up from the package that contains the test until it finds a class annotated with  `@SpringBootApplication`  or  `@SpringBootConfiguration` . As long as you [structured your code](using-boot-structuring-your-code.html) in a sensible way, your main configuration is usually found.

> If you use a [test annotation to test a more specific slice of your application](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests), you should avoid adding configuration settings that are specific to a particular area on the [main method’s application class](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-user-configuration).

If you want to customize the primary configuration, you can use a nested  `@TestConfiguration`  class. Unlike a nested  `@Configuration`  class, which would be used instead of your application’s primary configuration, a nested  `@TestConfiguration`  class is used in addition to your application’s primary configuration.

> Spring’s test framework caches application contexts between tests. Therefore, as long as your tests share the same configuration (no matter how it is discovered), the potentially time-consuming process of loading the context happens only once.

### 45.3.3 Excluding Test Configuration

If your application uses component scanning (for example, if you use  `@SpringBootApplication`  or  `@ComponentScan` ), you may find top-level configuration classes that you created only for specific tests accidentally get picked up everywhere.

As we [have seen earlier](boot-features-testing.html#boot-features-testing-spring-boot-applications-detecting-config),  `@TestConfiguration`  can be used on an inner class of a test to customize the primary configuration. When placed on a top-level class,  `@TestConfiguration`  indicates that classes in  `src/test/java`  should not be picked up by scanning. You can then import that class explicitly where it is required, as shown in the following example:

```java
@RunWith(SpringRunner.class)
@SpringBootTest
@Import(MyTestsConfiguration.class)
public class MyTests {

	@Test
	public void exampleTest() {
		...
	}

}
```

> If you directly use  `@ComponentScan`  (that is, not through  `@SpringBootApplication` ) you need to register the  `TypeExcludeFilter`  with it. See [the Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/context/TypeExcludeFilter.html) for details.

### 45.3.4 Testing with a mock environment

By default,  `@SpringBootTest`  does not start the server. If you have web endpoints that you want to test against this mock environment, you can additionally configure [MockMvc](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference//testing.html#spring-mvc-test-framework) as shown in the following example:

```java
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.AutoConfigureMockMvc;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureMockMvc
public class MockMvcExampleTests {

	@Autowired
	private MockMvc mvc;

	@Test
	public void exampleTest() throws Exception {
		this.mvc.perform(get("/")).andExpect(status().isOk())
				.andExpect(content().string("Hello World"));
	}

}
```

> If you want to focus only on the web layer and not start a complete  `ApplicationContext` , consider [using @WebMvcTest instead](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-mvc-tests).

Alternatively, you can configure a [WebTestClient](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#webtestclient-tests) as shown in the following example:

```java
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.AutoConfigureWebTestClient;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.reactive.server.WebTestClient;

@RunWith(SpringRunner.class)
@SpringBootTest
@AutoConfigureWebTestClient
public class MockWebTestClientExampleTests {

	@Autowired
	private WebTestClient webClient;

	@Test
	public void exampleTest() {
		this.webClient.get().uri("/").exchange().expectStatus().isOk()
				.expectBody(String.class).isEqualTo("Hello World");
	}

}
```

### 45.3.5 Testing with a running server

If you need to start a full running server, we recommend that you use random ports. If you use  `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)` , an available port is picked at random each time your test runs.

The  `@LocalServerPort`  annotation can be used to [inject the actual port used](howto-embedded-web-servers.html#howto-discover-the-http-port-at-runtime) into your test. For convenience, tests that need to make REST calls to the started server can additionally  `@Autowire`  a [WebTestClient](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#webtestclient-tests), which resolves relative links to the running server and comes with a dedicated API for verifying responses, as shown in the following example:

```java
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.reactive.server.WebTestClient;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class RandomPortWebTestClientExampleTests {

	@Autowired
	private WebTestClient webClient;

	@Test
	public void exampleTest() {
		this.webClient.get().uri("/").exchange().expectStatus().isOk()
				.expectBody(String.class).isEqualTo("Hello World");
	}

}
```

This setup requires  `spring-webflux`  on the classpath. If you can’t or won’t add webflux, Spring Boot also provides a  `TestRestTemplate`  facility:

```java
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.test.web.client.TestRestTemplate;
import org.springframework.test.context.junit4.SpringRunner;

import static org.assertj.core.api.Assertions.assertThat;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class RandomPortTestRestTemplateExampleTests {

	@Autowired
	private TestRestTemplate restTemplate;

	@Test
	public void exampleTest() {
		String body = this.restTemplate.getForObject("/", String.class);
		assertThat(body).isEqualTo("Hello World");
	}

}
```

### 45.3.6 Using JMX

As the test context framework caches context, JMX is disabled by default to prevent identical components to register on the same domain. If such test needs access to an  `MBeanServer` , consider marking it dirty as well:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(properties = "spring.jmx.enabled=true")
@DirtiesContext
public class SampleJmxTests {

	@Autowired
	private MBeanServer mBeanServer;

	@Test
	public void exampleTest() {
		// ...
	}

}
```

### 45.3.7 Mocking and Spying Beans

When running tests, it is sometimes necessary to mock certain components within your application context. For example, you may have a facade over some remote service that is unavailable during development. Mocking can also be useful when you want to simulate failures that might be hard to trigger in a real environment.

Spring Boot includes a  `@MockBean`  annotation that can be used to define a Mockito mock for a bean inside your  `ApplicationContext` . You can use the annotation to add new beans or replace a single existing bean definition. The annotation can be used directly on test classes, on fields within your test, or on  `@Configuration`  classes and fields. When used on a field, the instance of the created mock is also injected. Mock beans are automatically reset after each test method.

> If your test uses one of Spring Boot’s test annotations (such as  `@SpringBootTest` ), this feature is automatically enabled. To use this feature with a different arrangement, a listener must be explicitly added, as shown in the following example:

The following example replaces an existing  `RemoteService`  bean with a mock implementation:

```java
import org.junit.*;
import org.junit.runner.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.context.*;
import org.springframework.boot.test.mock.mockito.*;
import org.springframework.test.context.junit4.*;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.BDDMockito.*;

@RunWith(SpringRunner.class)
@SpringBootTest
public class MyTests {

	@MockBean
	private RemoteService remoteService;

	@Autowired
	private Reverser reverser;

	@Test
	public void exampleTest() {
		// RemoteService has been injected into the reverser bean
		given(this.remoteService.someCall()).willReturn("mock");
		String reverse = reverser.reverseSomeCall();
		assertThat(reverse).isEqualTo("kcom");
	}

}
```

Additionally, you can use  `@SpyBean`  to wrap any existing bean with a Mockito  `spy` . See the [Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/test/mock/mockito/SpyBean.html) for full details.

> While Spring’s test framework caches application contexts between tests and reuses a context for tests sharing the same configuration, the use of  `@MockBean`  or  `@SpyBean`  influences the cache key, which will most likely increase the number of contexts.

> If you are using  `@SpyBean`  to spy on a bean with  `@Cacheable`  methods that refer to parameters by name, your application must be compiled with  `-parameters` . This ensures that the parameter names are available to the caching infrastructure once the bean has been spied upon.

### 45.3.8 Auto-configured Tests

Spring Boot’s auto-configuration system works well for applications but can sometimes be a little too much for tests. It often helps to load only the parts of the configuration that are required to test a “slice” of your application. For example, you might want to test that Spring MVC controllers are mapping URLs correctly, and you do not want to involve database calls in those tests, or you might want to test JPA entities, and you are not interested in the web layer when those tests run.

The  `spring-boot-test-autoconfigure`  module includes a number of annotations that can be used to automatically configure such “slices”. Each of them works in a similar way, providing a  `@…Test`  annotation that loads the  `ApplicationContext`  and one or more  `@AutoConfigure…`  annotations that can be used to customize auto-configuration settings.

> Each slice restricts component scan to appropriate components and loads a very restricted set of auto-configuration classes. If you need to exclude one of them, most  `@…Test`  annotations provide an  `excludeAutoConfiguration`  attribute. Alternatively, you can use  `@ImportAutoConfiguration#exclude` .

> It is also possible to use the  `@AutoConfigure…`  annotations with the standard  `@SpringBootTest`  annotation. You can use this combination if you are not interested in “slicing” your application but you want some of the auto-configured test beans.

### 45.3.9 Auto-configured JSON Tests

To test that object JSON serialization and deserialization is working as expected, you can use the  `@JsonTest`  annotation.  `@JsonTest`  auto-configures the available supported JSON mapper, which can be one of the following libraries:

- Jackson  `ObjectMapper` , any  `@JsonComponent`  beans and any Jackson  `Module` s

-  `Gson` 

-  `Jsonb` 

> A list of the auto-configurations that are enabled by  `@JsonTest`  can be [found in the appendix](test-auto-configuration.html).

If you need to configure elements of the auto-configuration, you can use the  `@AutoConfigureJsonTesters`  annotation.

Spring Boot includes AssertJ-based helpers that work with the JSONAssert and JsonPath libraries to check that JSON appears as expected. The  `JacksonTester` ,  `GsonTester` ,  `JsonbTester` , and  `BasicJsonTester`  classes can be used for Jackson, Gson, Jsonb, and Strings respectively. Any helper fields on the test class can be  `@Autowired`  when using  `@JsonTest` . The following example shows a test class for Jackson:

```java
import org.junit.*;
import org.junit.runner.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.autoconfigure.json.*;
import org.springframework.boot.test.context.*;
import org.springframework.boot.test.json.*;
import org.springframework.test.context.junit4.*;

import static org.assertj.core.api.Assertions.*;

@RunWith(SpringRunner.class)
@JsonTest
public class MyJsonTests {

	@Autowired
	private JacksonTester<VehicleDetails> json;

	@Test
	public void testSerialize() throws Exception {
		VehicleDetails details = new VehicleDetails("Honda", "Civic");
		// Assert against a `.json` file in the same package as the test
		assertThat(this.json.write(details)).isEqualToJson("expected.json");
		// Or use JSON path based assertions
		assertThat(this.json.write(details)).hasJsonPathStringValue("@.make");
		assertThat(this.json.write(details)).extractingJsonPathStringValue("@.make")
				.isEqualTo("Honda");
	}

	@Test
	public void testDeserialize() throws Exception {
		String content = "{\"make\":\"Ford\",\"model\":\"Focus\"}";
		assertThat(this.json.parse(content))
				.isEqualTo(new VehicleDetails("Ford", "Focus"));
		assertThat(this.json.parseObject(content).getMake()).isEqualTo("Ford");
	}

}
```

> JSON helper classes can also be used directly in standard unit tests. To do so, call the  `initFields`  method of the helper in your  `@Before`  method if you do not use  `@JsonTest` .

### 45.3.10 Auto-configured Spring MVC Tests

To test whether Spring MVC controllers are working as expected, use the  `@WebMvcTest`  annotation.  `@WebMvcTest`  auto-configures the Spring MVC infrastructure and limits scanned beans to  `@Controller` ,  `@ControllerAdvice` ,  `@JsonComponent` ,  `Converter` ,  `GenericConverter` ,  `Filter` ,  `WebMvcConfigurer` , and  `HandlerMethodArgumentResolver` . Regular  `@Component`  beans are not scanned when using this annotation.

> A list of the auto-configuration settings that are enabled by  `@WebMvcTest`  can be [found in the appendix](test-auto-configuration.html).

> If you need to register extra components, such as the Jackson  `Module` , you can import additional configuration classes by using  `@Import`  on your test.

Often,  `@WebMvcTest`  is limited to a single controller and is used in combination with  `@MockBean`  to provide mock implementations for required collaborators.

`@WebMvcTest`  also auto-configures  `MockMvc` . Mock MVC offers a powerful way to quickly test MVC controllers without needing to start a full HTTP server.

> You can also auto-configure  `MockMvc`  in a non- `@WebMvcTest`  (such as  `@SpringBootTest` ) by annotating it with  `@AutoConfigureMockMvc` . The following example uses  `MockMvc` :

```java
import org.junit.*;
import org.junit.runner.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.autoconfigure.web.servlet.*;
import org.springframework.boot.test.mock.mockito.*;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.BDDMockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@WebMvcTest(UserVehicleController.class)
public class MyControllerTests {

	@Autowired
	private MockMvc mvc;

	@MockBean
	private UserVehicleService userVehicleService;

	@Test
	public void testExample() throws Exception {
		given(this.userVehicleService.getVehicleDetails("sboot"))
				.willReturn(new VehicleDetails("Honda", "Civic"));
		this.mvc.perform(get("/sboot/vehicle").accept(MediaType.TEXT_PLAIN))
				.andExpect(status().isOk()).andExpect(content().string("Honda Civic"));
	}

}
```

> If you need to configure elements of the auto-configuration (for example, when servlet filters should be applied) you can use attributes in the  `@AutoConfigureMockMvc`  annotation.

If you use HtmlUnit or Selenium, auto-configuration also provides an HTMLUnit  `WebClient`  bean and/or a  `WebDriver`  bean. The following example uses HtmlUnit:

```java
import com.gargoylesoftware.htmlunit.*;
import org.junit.*;
import org.junit.runner.*;
import org.springframework.beans.factory.annotation.*;
import org.springframework.boot.test.autoconfigure.web.servlet.*;
import org.springframework.boot.test.mock.mockito.*;

import static org.assertj.core.api.Assertions.*;
import static org.mockito.BDDMockito.*;

@RunWith(SpringRunner.class)
@WebMvcTest(UserVehicleController.class)
public class MyHtmlUnitTests {

	@Autowired
	private WebClient webClient;

	@MockBean
	private UserVehicleService userVehicleService;

	@Test
	public void testExample() throws Exception {
		given(this.userVehicleService.getVehicleDetails("sboot"))
				.willReturn(new VehicleDetails("Honda", "Civic"));
		HtmlPage page = this.webClient.getPage("/sboot/vehicle.html");
		assertThat(page.getBody().getTextContent()).isEqualTo("Honda Civic");
	}

}
```

> By default, Spring Boot puts  `WebDriver`  beans in a special “scope” to ensure that the driver exits after each test and that a new instance is injected. If you do not want this behavior, you can add  `@Scope("singleton")`  to your  `WebDriver`   `@Bean`  definition.

> The  `webDriver`  scope created by Spring Boot will replace any user defined scope of the same name. If you define your own  `webDriver`  scope you may find it stops working when you use  `@WebMvcTest` .

If you have Spring Security on the classpath,  `@WebMvcTest`  will also scan  `WebSecurityConfigurer`  beans. Instead of disabling security completely for such tests, you can use Spring Security’s test support. More details on how to use Spring Security’s  `MockMvc`  support can be found in this [Chapter 80, Testing With Spring Security](howto-use-test-with-spring-security.html) how-to section.

> Sometimes writing Spring MVC tests is not enough; Spring Boot can help you run [full end-to-end tests with an actual server](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-running-server).

### 45.3.11 Auto-configured Spring WebFlux Tests

To test that [Spring WebFlux](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference//web-reactive.html) controllers are working as expected, you can use the  `@WebFluxTest`  annotation.  `@WebFluxTest`  auto-configures the Spring WebFlux infrastructure and limits scanned beans to  `@Controller` ,  `@ControllerAdvice` ,  `@JsonComponent` ,  `Converter` ,  `GenericConverter` , and  `WebFluxConfigurer` . Regular  `@Component`  beans are not scanned when the  `@WebFluxTest`  annotation is used.

> A list of the auto-configurations that are enabled by  `@WebFluxTest`  can be [found in the appendix](test-auto-configuration.html).

> If you need to register extra components, such as Jackson  `Module` , you can import additional configuration classes using  `@Import`  on your test.

Often,  `@WebFluxTest`  is limited to a single controller and used in combination with the  `@MockBean`  annotation to provide mock implementations for required collaborators.

`@WebFluxTest`  also auto-configures [WebTestClient](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#webtestclient), which offers a powerful way to quickly test WebFlux controllers without needing to start a full HTTP server.

> You can also auto-configure  `WebTestClient`  in a non- `@WebFluxTest`  (such as  `@SpringBootTest` ) by annotating it with  `@AutoConfigureWebTestClient` . The following example shows a class that uses both  `@WebFluxTest`  and a  `WebTestClient` :

```java
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.reactive.WebFluxTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.reactive.server.WebTestClient;

@RunWith(SpringRunner.class)
@WebFluxTest(UserVehicleController.class)
public class MyControllerTests {

	@Autowired
	private WebTestClient webClient;

	@MockBean
	private UserVehicleService userVehicleService;

	@Test
	public void testExample() throws Exception {
		given(this.userVehicleService.getVehicleDetails("sboot"))
				.willReturn(new VehicleDetails("Honda", "Civic"));
		this.webClient.get().uri("/sboot/vehicle").accept(MediaType.TEXT_PLAIN)
				.exchange()
				.expectStatus().isOk()
				.expectBody(String.class).isEqualTo("Honda Civic");
	}

}
```

> This setup is only supported by WebFlux applications as using  `WebTestClient`  in a mocked web application only works with WebFlux at the moment.

>  `@WebFluxTest`  cannot detect routes registered via the functional web framework. For testing  `RouterFunction`  beans in the context, consider importing your  `RouterFunction`  yourself via  `@Import`  or using  `@SpringBootTest` .

> Sometimes writing Spring WebFlux tests is not enough; Spring Boot can help you run [full end-to-end tests with an actual server](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-running-server).

### 45.3.12 Auto-configured Data JPA Tests

You can use the  `@DataJpaTest`  annotation to test JPA applications. By default, it configures an in-memory embedded database, scans for  `@Entity`  classes, and configures Spring Data JPA repositories. Regular  `@Component`  beans are not loaded into the  `ApplicationContext` .

> A list of the auto-configuration settings that are enabled by  `@DataJpaTest`  can be [found in the appendix](test-auto-configuration.html).

By default, data JPA tests are transactional and roll back at the end of each test. See the [relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions) in the Spring Framework Reference Documentation for more details. If that is not what you want, you can disable transaction management for a test or for the whole class as follows:

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@RunWith(SpringRunner.class)
@DataJpaTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public class ExampleNonTransactionalTests {

}
```

Data JPA tests may also inject a [TestEntityManager](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure/orm/jpa/TestEntityManager.java) bean, which provides an alternative to the standard JPA  `EntityManager`  that is specifically designed for tests. If you want to use  `TestEntityManager`  outside of  `@DataJpaTest`  instances, you can also use the  `@AutoConfigureTestEntityManager`  annotation. A  `JdbcTemplate`  is also available if you need that. The following example shows the  `@DataJpaTest`  annotation in use:

```java
import org.junit.*;
import org.junit.runner.*;
import org.springframework.boot.test.autoconfigure.orm.jpa.*;

import static org.assertj.core.api.Assertions.*;

@RunWith(SpringRunner.class)
@DataJpaTest
public class ExampleRepositoryTests {

	@Autowired
	private TestEntityManager entityManager;

	@Autowired
	private UserRepository repository;

	@Test
	public void testExample() throws Exception {
		this.entityManager.persist(new User("sboot", "1234"));
		User user = this.repository.findByUsername("sboot");
		assertThat(user.getUsername()).isEqualTo("sboot");
		assertThat(user.getVin()).isEqualTo("1234");
	}

}
```

In-memory embedded databases generally work well for tests, since they are fast and do not require any installation. If, however, you prefer to run tests against a real database you can use the  `@AutoConfigureTestDatabase`  annotation, as shown in the following example:

```java
@RunWith(SpringRunner.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace=Replace.NONE)
public class ExampleRepositoryTests {

	// ...

}
```

### 45.3.13 Auto-configured JDBC Tests

`@JdbcTest`  is similar to  `@DataJpaTest`  but is for tests that only require a  `DataSource`  and do not use Spring Data JDBC. By default, it configures an in-memory embedded database and a  `JdbcTemplate` . Regular  `@Component`  beans are not loaded into the  `ApplicationContext` .

> A list of the auto-configurations that are enabled by  `@JdbcTest`  can be [found in the appendix](test-auto-configuration.html).

By default, JDBC tests are transactional and roll back at the end of each test. See the [relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions) in the Spring Framework Reference Documentation for more details. If that is not what you want, you can disable transaction management for a test or for the whole class, as follows:

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.autoconfigure.jdbc.JdbcTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@RunWith(SpringRunner.class)
@JdbcTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public class ExampleNonTransactionalTests {

}
```

If you prefer your test to run against a real database, you can use the  `@AutoConfigureTestDatabase`  annotation in the same way as for  `DataJpaTest` . (See "[Section 45.3.12, “Auto-configured Data JPA Tests”](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test)".)

### 45.3.14 Auto-configured Data JDBC Tests

`@DataJdbcTest`  is similar to  `@JdbcTest`  but is for tests that use Spring Data JDBC repositories. By default, it configures an in-memory embedded database, a  `JdbcTemplate` , and Spring Data JDBC repositories. Regular  `@Component`  beans are not loaded into the  `ApplicationContext` .

> A list of the auto-configurations that are enabled by  `@DataJdbcTest`  can be [found in the appendix](test-auto-configuration.html).

By default, Data JDBC tests are transactional and roll back at the end of each test. See the [relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions) in the Spring Framework Reference Documentation for more details. If that is not what you want, you can disable transaction management for a test or for the whole test class as [shown in the JDBC example](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jdbc-test).

If you prefer your test to run against a real database, you can use the  `@AutoConfigureTestDatabase`  annotation in the same way as for  `DataJpaTest` . (See "[Section 45.3.12, “Auto-configured Data JPA Tests”](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test)".)

### 45.3.15 Auto-configured jOOQ Tests

You can use  `@JooqTest`  in a similar fashion as  `@JdbcTest`  but for jOOQ-related tests. As jOOQ relies heavily on a Java-based schema that corresponds with the database schema, the existing  `DataSource`  is used. If you want to replace it with an in-memory database, you can use  `@AutoConfigureTestDatabase`  to override those settings. (For more about using jOOQ with Spring Boot, see "[Section 30.6, “Using jOOQ”](boot-features-sql.html#boot-features-jooq)", earlier in this chapter.) Regular  `@Component`  beans are not loaded into the  `ApplicationContext` .

> A list of the auto-configurations that are enabled by  `@JooqTest`  can be [found in the appendix](test-auto-configuration.html).

`@JooqTest`  configures a  `DSLContext` . Regular  `@Component`  beans are not loaded into the  `ApplicationContext` . The following example shows the  `@JooqTest`  annotation in use:

```java
import org.jooq.DSLContext;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.autoconfigure.jooq.JooqTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@JooqTest
public class ExampleJooqTests {

	@Autowired
	private DSLContext dslContext;
}
```

JOOQ tests are transactional and roll back at the end of each test by default. If that is not what you want, you can disable transaction management for a test or for the whole test class as [shown in the JDBC example](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jdbc-test).

### 45.3.16 Auto-configured Data MongoDB Tests

You can use  `@DataMongoTest`  to test MongoDB applications. By default, it configures an in-memory embedded MongoDB (if available), configures a  `MongoTemplate` , scans for  `@Document`  classes, and configures Spring Data MongoDB repositories. Regular  `@Component`  beans are not loaded into the  `ApplicationContext` . (For more about using MongoDB with Spring Boot, see "[Section 31.2, “MongoDB”](boot-features-nosql.html#boot-features-mongodb)", earlier in this chapter.)

> A list of the auto-configuration settings that are enabled by  `@DataMongoTest`  can be [found in the appendix](test-auto-configuration.html).

The following class shows the  `@DataMongoTest`  annotation in use:

```java
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;
import org.springframework.data.mongodb.core.MongoTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataMongoTest
public class ExampleDataMongoTests {

	@Autowired
	private MongoTemplate mongoTemplate;

	//
}
```

In-memory embedded MongoDB generally works well for tests, since it is fast and does not require any developer installation. If, however, you prefer to run tests against a real MongoDB server, you should exclude the embedded MongoDB auto-configuration, as shown in the following example:

```java
import org.junit.runner.RunWith;
import org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration;
import org.springframework.boot.test.autoconfigure.data.mongo.DataMongoTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataMongoTest(excludeAutoConfiguration = EmbeddedMongoAutoConfiguration.class)
public class ExampleDataMongoNonEmbeddedTests {

}
```

### 45.3.17 Auto-configured Data Neo4j Tests

You can use  `@DataNeo4jTest`  to test Neo4j applications. By default, it uses an in-memory embedded Neo4j (if the embedded driver is available), scans for  `@NodeEntity`  classes, and configures Spring Data Neo4j repositories. Regular  `@Component`  beans are not loaded into the  `ApplicationContext` . (For more about using Neo4J with Spring Boot, see "[Section 31.3, “Neo4j”](boot-features-nosql.html#boot-features-neo4j)", earlier in this chapter.)

> A list of the auto-configuration settings that are enabled by  `@DataNeo4jTest`  can be [found in the appendix](test-auto-configuration.html).

The following example shows a typical setup for using Neo4J tests in Spring Boot:

```java
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.neo4j.DataNeo4jTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataNeo4jTest
public class ExampleDataNeo4jTests {

	@Autowired
	private YourRepository repository;

	//
}
```

By default, Data Neo4j tests are transactional and roll back at the end of each test. See the [relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions) in the Spring Framework Reference Documentation for more details. If that is not what you want, you can disable transaction management for a test or for the whole class, as follows:

```java
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.autoconfigure.data.neo4j.DataNeo4jTest;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@RunWith(SpringRunner.class)
@DataNeo4jTest
@Transactional(propagation = Propagation.NOT_SUPPORTED)
public class ExampleNonTransactionalTests {

}
```

### 45.3.18 Auto-configured Data Redis Tests

You can use  `@DataRedisTest`  to test Redis applications. By default, it scans for  `@RedisHash`  classes and configures Spring Data Redis repositories. Regular  `@Component`  beans are not loaded into the  `ApplicationContext` . (For more about using Redis with Spring Boot, see "[Section 31.1, “Redis”](boot-features-nosql.html#boot-features-redis)", earlier in this chapter.)

> A list of the auto-configuration settings that are enabled by  `@DataRedisTest`  can be [found in the appendix](test-auto-configuration.html).

The following example shows the  `@DataRedisTest`  annotation in use:

```java
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.redis.DataRedisTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataRedisTest
public class ExampleDataRedisTests {

	@Autowired
	private YourRepository repository;

	//
}
```

### 45.3.19 Auto-configured Data LDAP Tests

You can use  `@DataLdapTest`  to test LDAP applications. By default, it configures an in-memory embedded LDAP (if available), configures an  `LdapTemplate` , scans for  `@Entry`  classes, and configures Spring Data LDAP repositories. Regular  `@Component`  beans are not loaded into the  `ApplicationContext` . (For more about using LDAP with Spring Boot, see "[Section 31.9, “LDAP”](boot-features-nosql.html#boot-features-ldap)", earlier in this chapter.)

> A list of the auto-configuration settings that are enabled by  `@DataLdapTest`  can be [found in the appendix](test-auto-configuration.html).

The following example shows the  `@DataLdapTest`  annotation in use:

```java
import org.junit.runner.RunWith;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.data.ldap.DataLdapTest;
import org.springframework.ldap.core.LdapTemplate;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataLdapTest
public class ExampleDataLdapTests {

	@Autowired
	private LdapTemplate ldapTemplate;

	//
}
```

In-memory embedded LDAP generally works well for tests, since it is fast and does not require any developer installation. If, however, you prefer to run tests against a real LDAP server, you should exclude the embedded LDAP auto-configuration, as shown in the following example:

```java
import org.junit.runner.RunWith;
import org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration;
import org.springframework.boot.test.autoconfigure.data.ldap.DataLdapTest;
import org.springframework.test.context.junit4.SpringRunner;

@RunWith(SpringRunner.class)
@DataLdapTest(excludeAutoConfiguration = EmbeddedLdapAutoConfiguration.class)
public class ExampleDataLdapNonEmbeddedTests {

}
```

### 45.3.20 Auto-configured REST Clients

You can use the  `@RestClientTest`  annotation to test REST clients. By default, it auto-configures Jackson, GSON, and Jsonb support, configures a  `RestTemplateBuilder` , and adds support for  `MockRestServiceServer` . Regular  `@Component`  beans are not loaded into the  `ApplicationContext` .

> A list of the auto-configuration settings that are enabled by  `@RestClientTest`  can be [found in the appendix](test-auto-configuration.html).

The specific beans that you want to test should be specified by using the  `value`  or  `components`  attribute of  `@RestClientTest` , as shown in the following example:

```java
@RunWith(SpringRunner.class)
@RestClientTest(RemoteVehicleDetailsService.class)
public class ExampleRestClientTest {

	@Autowired
	private RemoteVehicleDetailsService service;

	@Autowired
	private MockRestServiceServer server;

	@Test
	public void getVehicleDetailsWhenResultIsSuccessShouldReturnDetails()
			throws Exception {
		this.server.expect(requestTo("/greet/details"))
				.andRespond(withSuccess("hello", MediaType.TEXT_PLAIN));
		String greeting = this.service.callRestService();
		assertThat(greeting).isEqualTo("hello");
	}

}
```

### 45.3.21 Auto-configured Spring REST Docs Tests

You can use the  `@AutoConfigureRestDocs`  annotation to use [Spring REST Docs](https://projects.spring.io/spring-restdocs/) in your tests with Mock MVC or REST Assured. It removes the need for the JUnit rule in Spring REST Docs.

`@AutoConfigureRestDocs`  can be used to override the default output directory ( `target/generated-snippets`  if you are using Maven or  `build/generated-snippets`  if you are using Gradle). It can also be used to configure the host, scheme, and port that appears in any documented URIs.

#### Auto-configured Spring REST Docs Tests with Mock MVC

`@AutoConfigureRestDocs`  customizes the  `MockMvc`  bean to use Spring REST Docs. You can inject it by using  `@Autowired`  and use it in your tests as you normally would when using Mock MVC and Spring REST Docs, as shown in the following example:

```java
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.http.MediaType;
import org.springframework.test.context.junit4.SpringRunner;
import org.springframework.test.web.servlet.MockMvc;

import static org.springframework.restdocs.mockmvc.MockMvcRestDocumentation.document;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@RunWith(SpringRunner.class)
@WebMvcTest(UserController.class)
@AutoConfigureRestDocs
public class UserDocumentationTests {

	@Autowired
	private MockMvc mvc;

	@Test
	public void listUsers() throws Exception {
		this.mvc.perform(get("/users").accept(MediaType.TEXT_PLAIN))
				.andExpect(status().isOk())
				.andDo(document("list-users"));
	}

}
```

If you require more control over Spring REST Docs configuration than offered by the attributes of  `@AutoConfigureRestDocs` , you can use a  `RestDocsMockMvcConfigurationCustomizer`  bean, as shown in the following example:

```java
@TestConfiguration
static class CustomizationConfiguration
		implements RestDocsMockMvcConfigurationCustomizer {

	@Override
	public void customize(MockMvcRestDocumentationConfigurer configurer) {
		configurer.snippets().withTemplateFormat(TemplateFormats.markdown());
	}

}
```

If you want to make use of Spring REST Docs support for a parameterized output directory, you can create a  `RestDocumentationResultHandler`  bean. The auto-configuration calls  `alwaysDo`  with this result handler, thereby causing each  `MockMvc`  call to automatically generate the default snippets. The following example shows a  `RestDocumentationResultHandler`  being defined:

```java
@TestConfiguration
static class ResultHandlerConfiguration {

	@Bean
	public RestDocumentationResultHandler restDocumentation() {
		return MockMvcRestDocumentation.document("{method-name}");
	}

}
```

#### Auto-configured Spring REST Docs Tests with REST Assured

`@AutoConfigureRestDocs`  makes a  `RequestSpecification`  bean, preconfigured to use Spring REST Docs, available to your tests. You can inject it by using  `@Autowired`  and use it in your tests as you normally would when using REST Assured and Spring REST Docs, as shown in the following example:

```java
import io.restassured.specification.RequestSpecification;
import org.junit.Test;
import org.junit.runner.RunWith;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.restdocs.AutoConfigureRestDocs;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.context.SpringBootTest.WebEnvironment;
import org.springframework.boot.web.server.LocalServerPort;
import org.springframework.test.context.junit4.SpringRunner;

import static io.restassured.RestAssured.given;
import static org.hamcrest.CoreMatchers.is;
import static org.springframework.restdocs.restassured3.RestAssuredRestDocumentation.document;

@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@AutoConfigureRestDocs
public class UserDocumentationTests {

	@LocalServerPort
	private int port;

	@Autowired
	private RequestSpecification documentationSpec;

	@Test
	public void listUsers() {
		given(this.documentationSpec).filter(document("list-users")).when()
				.port(this.port).get("/").then().assertThat().statusCode(is(200));
	}

}
```

If you require more control over Spring REST Docs configuration than offered by the attributes of  `@AutoConfigureRestDocs` , a  `RestDocsRestAssuredConfigurationCustomizer`  bean can be used, as shown in the following example:

```java
@TestConfiguration
public static class CustomizationConfiguration
		implements RestDocsRestAssuredConfigurationCustomizer {

	@Override
	public void customize(RestAssuredRestDocumentationConfigurer configurer) {
		configurer.snippets().withTemplateFormat(TemplateFormats.markdown());
	}

}
```

### 45.3.22 Additional Auto-configuration and Slicing

Each slice provides one or more  `@AutoConfigure…`  annotations that namely defines the auto-configurations that should be included as part of a slice. Additional auto-configurations can be added by creating a custom  `@AutoConfigure…`  annotation or simply by adding  `@ImportAutoConfiguration`  to the test as shown in the following example:

```java
@RunWith(SpringRunner.class)
@JdbcTest
@ImportAutoConfiguration(IntegrationAutoConfiguration.class)
public class ExampleJdbcTests {

}
```

> Make sure to not use the regular  `@Import`  annotation to import auto-configurations as they are handled in a specific way by Spring Boot.

### 45.3.23 User Configuration and Slicing

If you [structure your code](using-boot-structuring-your-code.html) in a sensible way, your  `@SpringBootApplication`  class is [used by default](boot-features-testing.html#boot-features-testing-spring-boot-applications-detecting-config) as the configuration of your tests.

It then becomes important not to litter the application’s main class with configuration settings that are specific to a particular area of its functionality.

Assume that you are using Spring Batch and you rely on the auto-configuration for it. You could define your  `@SpringBootApplication`  as follows:

```java
@SpringBootApplication
@EnableBatchProcessing
public class SampleApplication { ... }
```

Because this class is the source configuration for the test, any slice test actually tries to start Spring Batch, which is definitely not what you want to do. A recommended approach is to move that area-specific configuration to a separate  `@Configuration`  class at the same level as your application, as shown in the following example:

```java
@Configuration
@EnableBatchProcessing
public class BatchConfiguration { ... }
```

> Depending on the complexity of your application, you may either have a single  `@Configuration`  class for your customizations or one class per domain area. The latter approach lets you enable it in one of your tests, if necessary, with the  `@Import`  annotation.

Another source of confusion is classpath scanning. Assume that, while you structured your code in a sensible way, you need to scan an additional package. Your application may resemble the following code:

```java
@SpringBootApplication
@ComponentScan({ "com.example.app", "org.acme.another" })
public class SampleApplication { ... }
```

Doing so effectively overrides the default component scan directive with the side effect of scanning those two packages regardless of the slice that you chose. For instance, a  `@DataJpaTest`  seems to suddenly scan components and user configurations of your application. Again, moving the custom directive to a separate class is a good way to fix this issue.

> If this is not an option for you, you can create a  `@SpringBootConfiguration`  somewhere in the hierarchy of your test so that it is used instead. Alternatively, you can specify a source for your test, which disables the behavior of finding a default one.

### 45.3.24 Using Spock to Test Spring Boot Applications

If you wish to use Spock to test a Spring Boot application, you should add a dependency on Spock’s  `spock-spring`  module to your application’s build.  `spock-spring`  integrates Spring’s test framework into Spock. It is recommended that you use Spock 1.1 or later to benefit from a number of improvements to Spock’s Spring Framework and Spring Boot integration. See [the documentation for Spock’s Spring module](http://spockframework.org/spock/docs/1.1/modules.html) for further details.

## 45.4 Test Utilities

A few test utility classes that are generally useful when testing your application are packaged as part of  `spring-boot` .

### 45.4.1 ConfigFileApplicationContextInitializer

`ConfigFileApplicationContextInitializer`  is an  `ApplicationContextInitializer`  that you can apply to your tests to load Spring Boot  `application.properties`  files. You can use it when you do not need the full set of features provided by  `@SpringBootTest` , as shown in the following example:

```java
@ContextConfiguration(classes = Config.class,
	initializers = ConfigFileApplicationContextInitializer.class)
```

> Using  `ConfigFileApplicationContextInitializer`  alone does not provide support for  `@Value("${…}")`  injection. Its only job is to ensure that  `application.properties`  files are loaded into Spring’s  `Environment` . For  `@Value`  support, you need to either additionally configure a  `PropertySourcesPlaceholderConfigurer`  or use  `@SpringBootTest` , which auto-configures one for you.

### 45.4.2 TestPropertyValues

`TestPropertyValues`  lets you quickly add properties to a  `ConfigurableEnvironment`  or  `ConfigurableApplicationContext` . You can call it with  `key=value`  strings, as follows:

```java
TestPropertyValues.of("org=Spring", "name=Boot").applyTo(env);
```

### 45.4.3 OutputCapture

`OutputCapture`  is a JUnit  `Rule`  that you can use to capture  `System.out`  and  `System.err`  output. You can declare the capture as a  `@Rule`  and then use  `toString()`  for assertions, as follows:

```java
import org.junit.Rule;
import org.junit.Test;
import org.springframework.boot.test.rule.OutputCapture;

import static org.hamcrest.Matchers.*;
import static org.junit.Assert.*;

public class MyTest {

	@Rule
	public OutputCapture capture = new OutputCapture();

	@Test
	public void testName() throws Exception {
		System.out.println("Hello World!");
		assertThat(capture.toString(), containsString("World"));
	}

}
```

### 45.4.4 TestRestTemplate

> Spring Framework 5.0 provides a new  `WebTestClient`  that works for [WebFlux integration tests](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-webflux-tests) and both [WebFlux and MVC end-to-end testing](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-running-server). It provides a fluent API for assertions, unlike  `TestRestTemplate` .

`TestRestTemplate`  is a convenience alternative to Spring’s  `RestTemplate`  that is useful in integration tests. You can get a vanilla template or one that sends Basic HTTP authentication (with a username and password). In either case, the template behaves in a test-friendly way by not throwing exceptions on server-side errors. It is recommended, but not mandatory, to use the Apache HTTP Client (version 4.3.2 or better). If you have that on your classpath, the  `TestRestTemplate`  responds by configuring the client appropriately. If you do use Apache’s HTTP client, some additional test-friendly features are enabled:

- Redirects are not followed (so you can assert the response location).

- Cookies are ignored (so the template is stateless).

`TestRestTemplate`  can be instantiated directly in your integration tests, as shown in the following example:

```java
public class MyTest {

	private TestRestTemplate template = new TestRestTemplate();

	@Test
	public void testRequest() throws Exception {
		HttpHeaders headers = this.template.getForEntity(
				"http://myhost.example.com/example", String.class).getHeaders();
		assertThat(headers.getLocation()).hasHost("other.example.com");
	}

}
```

Alternatively, if you use the  `@SpringBootTest`  annotation with  `WebEnvironment.RANDOM_PORT`  or  `WebEnvironment.DEFINED_PORT` , you can inject a fully configured  `TestRestTemplate`  and start using it. If necessary, additional customizations can be applied through the  `RestTemplateBuilder`  bean. Any URLs that do not specify a host and port automatically connect to the embedded server, as shown in the following example:

```java
@RunWith(SpringRunner.class)
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
public class SampleWebClientTests {

	@Autowired
	private TestRestTemplate template;

	@Test
	public void testRequest() {
		HttpHeaders headers = this.template.getForEntity("/example", String.class)
				.getHeaders();
		assertThat(headers.getLocation()).hasHost("other.example.com");
	}

	@TestConfiguration
	static class Config {

		@Bean
		public RestTemplateBuilder restTemplateBuilder() {
			return new RestTemplateBuilder().setConnectTimeout(Duration.ofSeconds(1))
					.setReadTimeout(Duration.ofSeconds(1));
		}

	}

}
```


## 45.测试

Spring Boot提供了许多实用程序和注释来帮助您测试应用程序.测试支持由两个模块提供： `spring-boot-test` 包含核心项， `spring-boot-test-autoconfigure` 支持测试的自动配置.

大多数开发人员使用 `spring-boot-starter-test` “Starter”，它导入Spring Boot测试模块以及JUnit，AssertJ，Hamcrest和许多其他有用的库.

## 45.1测试范围依赖关系

`spring-boot-starter-test` “Starter”（在 `test`   `scope` 中）包含以下提供的库：

- [JUnit](http://junit.org)：单元测试Java应用程序的事实上的标准.

- [Spring Test](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#integration-testing)＆Spring Boot Test：Spring Boot应用程序的实用程序和集成测试支持.

- [AssertJ](https://joel-costigliola.github.io/assertj/)：流畅的断言库.

- [Hamcrest](http://hamcrest.org/JavaHamcrest/)：匹配器对象库（也称为约束或谓词）.

- [Mockito](http://mockito.org/)：Java模拟框架.

- [JSONassert](https://github.com/skyscreamer/JSONassert)：JSON的断言库.

- [JsonPath](https://github.com/jayway/JsonPath)：XPath for JSON.

我们通常发现这些常用库在编写测试时很有用.如果这些库不适合您的需求，您可以添加自己的其他测试依赖项.

## 45.2测试Spring应用程序

依赖注入的一个主要优点是它应该使您的代码更容易进行单元测试.您可以使用 `new` 运算符实例化对象，甚至不涉及Spring.您还可以使用模拟对象而不是真正的依赖项.

通常，您需要超越单元测试并开始集成测试（使用Spring  `ApplicationContext` ）.能够在不需要部署应用程序或需要连接到其他基础结构的情况下执行集成测试非常有用.

Spring Framework包含一个用于此类集成测试的专用测试模块.您可以直接向 `org.springframework:spring-test` 声明一个依赖项，或者使用 `spring-boot-starter-test` “Starter”来传递它.

如果您之前没有使用过 `spring-test` 模块，那么首先应该阅读Spring Framework参考文档的[relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testing).

## 45.3测试Spring Boot应用程序

Spring Boot应用程序是一个Spring  `ApplicationContext` ，所以没有什么特别的东西可以用来测试它超出你通常使用的vanilla上下文.

> 默认情况下，只有在使用 `SpringApplication` 创建Spring Boot属性，日志记录和Spring Boot的其他功能时才会在上下文中安装.

Spring Boot提供 `@SpringBootTest` 注释，当您需要Spring Boot功能时，可以将其用作标准 `spring-test`   `@ContextConfiguration` 注释的替代方法.注释由[creating the ApplicationContext used in your tests through SpringApplication](boot-features-testing.html#boot-features-testing-spring-boot-applications-detecting-config)起作用.除了 `@SpringBootTest` 之外，还为应用程序的[testing more specific slices](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)提供了许多其他注释.

> 如果您使用的是JUnit 4，请不要忘记在测试中添加 `@RunWith(SpringRunner.class)` ，否则注释将被忽略.如果您使用的是JUnit 5，则无需将等效的 `@ExtendWith(SpringExtension)` 添加为 `@SpringBootTest` ，而其他 `@…Test` 注释已经使用它进行注释.

默认情况下， `@SpringBootTest` 将无法启动服务器.您可以使用 `@SpringBootTest` 的 `webEnvironment` 属性来进一步优化测试的运行方式：

-  `MOCK` （默认）：加载Web  `ApplicationContext` 并提供模拟Web环境.使用此批注时，不会启动嵌入式服务器.如果您的类路径上没有Web环境，则此模式将透明地回退到创建常规非Web  `ApplicationContext` .它可以与[@AutoConfigureMockMvc or @AutoConfigureWebTestClient](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-mock-environment)结合使用，以进行基于模拟的Web应用程序测试.

-  `RANDOM_PORT` ：加载 `WebServerApplicationContext` 并提供真实的Web环境.嵌入式服务器启动并在随机端口上侦听.

-  `DEFINED_PORT` ：加载 `WebServerApplicationContext` 并提供真实的Web环境.嵌入式服务器启动并侦听定义的端口（来自 `application.properties` ）或默认端口 `8080` .

-  `NONE` ：使用 `SpringApplication` 加载 `ApplicationContext` 但不提供任何Web环境（模拟或其他）.

> 如果您的测试是 `@Transactional` ，则默认情况下会在每个测试方法结束时回滚事务.但是，由于使用 `RANDOM_PORT` 或 `DEFINED_PORT` 这种安排隐式提供了一个真正的servlet环境，因此HTTP客户端和服务器在不同的线程中运行，因此在单独的事务中运行.在这种情况下，在服务器上启动的任何事务都不会回滚.

如果您的应用程序为管理服务器使用不同的端口，
>  `@SpringBootTest`  with  `webEnvironment = WebEnvironment.RANDOM_PORT` 也将在单独的随机端口上启动管理服务器.

### 45.3.1检测Web应用程序类型

如果Spring MVC可用，则配置基于MVC的常规应用程序上下文.如果您只有Spring WebFlux，我们将检测到并配置基于WebFlux的应用程序上下文.

如果两者都存在，则Spring MVC优先.如果要在此方案中测试响应式Web应用程序，则必须设置 `spring.main.web-application-type` 属性：

```java
@RunWith(SpringRunner.class)
@SpringBootTest(properties = "spring.main.web-application-type=reactive")
public class MyWebFluxTests { ... }
```

### 45.3.2检测测试配置

如果您熟悉Spring Test Framework，则可能习惯使用 `@ContextConfiguration(classes=…)` 来指定要加载的Spring  `@Configuration` .或者，您可能经常在测试中使用嵌套的 `@Configuration` 类.

在测试Spring Boot应用程序时，通常不需要这样做.只要您没有明确定义，Spring Boot的 `@*Test` 注释就会自动搜索您的主要配置.

搜索算法从包含测试的包开始工作，直到找到使用 `@SpringBootApplication` 或 `@SpringBootConfiguration` 注释的类.只要您以合理的方式[structured your code](using-boot-structuring-your-code.html)，通常会找到您的主要配置.

> 如果使用[test annotation to test a more specific slice of your application](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-tests)，则应避免添加特定于[main method’s application class](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-user-configuration)上特定区域的配置设置.

如果要自定义主要配置，可以使用嵌套的 `@TestConfiguration` 类.与嵌套的 `@Configuration` 类不同，它将用于代替应用程序的主要配置，除了应用程序的主要配置之外，还使用嵌套的 `@TestConfiguration` 类.

> Spring的测试框架在测试之间缓存应用程序上下文.因此，只要您的测试共享相同的配置（无论如何发现），加载上下文的潜在耗时过程只会发生一次.

### 45.3.3不包括测试配置

如果您的应用程序使用组件扫描（例如，如果您使用 `@SpringBootApplication` 或 `@ComponentScan` ），您可能会发现仅为特定测试创建的顶级配置类会意外地在任何地方被拾取.

正如我们[have seen earlier](boot-features-testing.html#boot-features-testing-spring-boot-applications-detecting-config)， `@TestConfiguration` 可用于测试的内部类以自定义主要配置.放置在顶级类时， `@TestConfiguration` 表示不应通过扫描拾取 `src/test/java` 中的类.然后，您可以在需要的位置显式导入该类，如以下示例所示：

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

> 如果直接使用 `@ComponentScan` （即不通过 `@SpringBootApplication` ），则需要使用 `TypeExcludeFilter` 注册.有关详细信息，请参阅[the Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/context/TypeExcludeFilter.html).

### 45.3.4使用模拟环境进行测试

默认情况下， `@SpringBootTest` 不会启动服务器.如果您要针对此模拟环境测试Webendpoints，则可以另外配置[MockMvc](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference//testing.html#spring-mvc-test-framework)，如以下示例所示：

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

> 如果您只想关注Web图层而不是启动完整的 `ApplicationContext` ，请考虑[using @WebMvcTest instead](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-mvc-tests).

或者，您可以配置[WebTestClient](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#webtestclient-tests)，如以下示例所示：

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

### 45.3.5使用正在运行的服务器进行测试

如果您需要启动完整运行的服务器，我们建议您使用随机端口.如果使用 `@SpringBootTest(webEnvironment=WebEnvironment.RANDOM_PORT)` ，则每次运行测试时都会随机选取一个可用端口.

`@LocalServerPort` 注释可用于[inject the actual port used](howto-embedded-web-servers.html#howto-discover-the-http-port-at-runtime)进入测试.为方便起见，需要对启动的服务器进行REST调用的测试还可以 `@Autowire` a [WebTestClient](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#webtestclient-tests)，它解析了与正在运行的服务器的相对链接，并附带了用于验证响应的专用API，如以下示例所示：

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

此设置在类路径上需要 `spring-webflux` .如果您不能或不会添加webflux，Spring Boot还提供 `TestRestTemplate` 工具：

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

### 45.3.6使用JMX

当测试上下文框架缓存上下文时，默认情况下禁用JMX以防止相同的组件在同一域上注册.如果此类测试需要访问 `MBeanServer` ，请考虑将其标记为脏：

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

### 45.3.7Mock和Spy beans

运行测试时，有时需要在应用程序上下文中模拟某些组件.例如，您可能拥有在开发期间不可用的某些远程服务的外观.当您想要模拟在真实环境中可能难以触发的故障时，模拟也很有用.

Spring Boot包含一个可用于的 `@MockBean` 注释为 `ApplicationContext` 中的bean定义一个Mockito模拟.您可以使用批注添加新bean或替换单个现有bean定义.注释可以直接用于测试类，测试中的字段或 `@Configuration` 类和字段.在字段上使用时，也会注入创建的模拟的实例.每种测试方法后，模拟 beans都会自动重置.

> 如果您的测试使用Spring Boot的一个测试注释（例如 `@SpringBootTest` ），则会自动启用此功能.要以不同的排列方式使用此功能，必须显式添加侦听器，如以下示例所示：

以下示例使用模拟实现替换现有的 `RemoteService`  bean：

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

此外，您可以使用 `@SpyBean` 用Mockito  `spy` 包装任何现有bean.有关详细信息，请参阅[Javadoc](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/test/mock/mockito/SpyBean.html).

> While Spring的测试框架在测试之间缓存应用程序上下文并重用共享相同配置的测试的上下文，使用 `@MockBean` 或 `@SpyBean` 会影响缓存键，这很可能会增加上下文的数量.

> 如果您使用 `@SpyBean` 监视bean，并且 `@Cacheable` 方法按名称引用参数，则必须使用 `-parameters` 编译应用程序.这确保了一旦bean被监视，参数名称可用于缓存基础结构.

### 45.3.8自动配置测试

Spring Boot的自动配置系统适用于应用程序，但有时对于测试来说有点太多了.通常，只需加载测试应用程序“切片”所需的配置部分.例如，您可能希望测试Spring MVC控制器是否正确映射URL，并且您不希望在这些测试中涉及数据库调用，或者您可能想要测试JPA实体，并且您对Web层不感兴趣测试运行.

`spring-boot-test-autoconfigure` 模块包括许多可用于自动配置此类“切片”的注释.它们中的每一个都以类似的方式工作，提供 `@…Test` 注释，用于加载 `ApplicationContext` 和一个或多个 `@AutoConfigure…` 注释，可用于自定义自动配置设置.

> 每个切片将组件扫描限制为适当的组件，并加载一组非常有限的自动配置类.如果您需要排除其中一个，则大多数 `@…Test` 注释都会提供 `excludeAutoConfiguration` 属性.或者，您可以使用 `@ImportAutoConfiguration#exclude` .

> 也可以将 `@AutoConfigure…` 注释与标准 `@SpringBootTest` 注释一起使用.如果您对“切片”应用程序不感兴趣但想要一些自动配置的测试bean，则可以使用此组合.

### 45.3.9自动配置的JSON测试

要测试该对象JSON序列化和反序列化是否按预期工作，您可以使用 `@JsonTest` 注释.  `@JsonTest` 自动配置可用的受支持JSON映射器，它可以是以下库之一：

- Jackson  `ObjectMapper` ，任何 `@JsonComponent`  beans子和任何Jackson `Module` s

-  `Gson` 

-  `Jsonb` 

>   `@JsonTest` 启用的自动配置列表可以是[found in the appendix](test-auto-configuration.html).

如果需要配置自动配置的元素，可以使用 `@AutoConfigureJsonTesters` 注释.

Spring Boot包括基于AssertJ的助手，它们与JSONAssert和JsonPath库一起使用，以检查JSON是否按预期显示.  `JacksonTester` ， `GsonTester` ， `JsonbTester` 和 `BasicJsonTester` 类可分别用于Jackson，Gson，Jsonb和Strings.使用 `@JsonTest` 时，测试类上的任何辅助字段都可以是 `@Autowired` .以下示例显示了Jackson的测试类：

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

> JSON助手类也可以直接在标准单元测试中使用.为此，如果不使用 `@JsonTest` ，请在 `@Before` 方法中调用助手的 `initFields` 方法.

### 45.3.10自动配置的Spring MVC测试

要测试Spring MVC控制器是否按预期工作，请使用 `@WebMvcTest` 注释.  `@WebMvcTest` 自动配置Spring MVC基础结构并将扫描的bean限制为 `@Controller` ， `@ControllerAdvice` ， `@JsonComponent` ， `Converter` ， `GenericConverter` ， `Filter` ， `WebMvcConfigurer` 和 `HandlerMethodArgumentResolver` .使用此批注时，不会扫描常规 `@Component`  bean.

>   `@WebMvcTest` 启用的自动配置设置列表可以是[found in the appendix](test-auto-configuration.html).

> 如果您需要注册额外的组件，例如Jackson  `Module` ，您可以在测试中使用 `@Import` 导入其他配置类.

通常， `@WebMvcTest` 仅限于一个控制器，并与 `@MockBean` 结合使用，为所需的协作者提供模拟实现.

`@WebMvcTest` 也自动配置 `MockMvc` . Mock MVC提供了一种快速测试MVC控制器的强大方法，无需启动完整的HTTP服务器.

> 您也可以通过用 `@AutoConfigureMockMvc` 注释来自动配置非 `@WebMvcTest` （例如 `@SpringBootTest` ）中的 `MockMvc` .以下示例使用 `MockMvc` ：

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

> 如果需要配置自动配置的元素（例如，应该应用servlet过滤器时），可以使用 `@AutoConfigureMockMvc` 注释中的属性.

如果您使用HtmlUnit或Selenium，auto-configuration还提供了HTMLUnit  `WebClient`  bean和/或 `WebDriver`  bean.以下示例使用HtmlUnit：

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

> By默认情况下，Spring Boot将 `WebDriver`  beans放在一个特殊的“范围”中，以确保驱动程序在每次测试后退出并注入新实例.如果您不想要此行为，可以将 `@Scope("singleton")` 添加到 `WebDriver`   `@Bean` 定义中.

>  Spring Boot创建的 `webDriver` 范围将替换任何用户定义的同名范围.如果您定义自己的 `webDriver` 范围，则可能会在使用 `@WebMvcTest` 时发现它停止工作.

如果类路径上有Spring Security， `@WebMvcTest` 也会扫描 `WebSecurityConfigurer`  beans.您可以使用Spring Security的测试支持，而不是完全禁用此类测试的安全性.有关如何使用Spring Security的 `MockMvc` 支持的更多详细信息，请参阅此[Chapter 80, Testing With Spring Security](howto-use-test-with-spring-security.html)操作方法部分.

_0015.Sometimes编写Spring MVC测试是不够的; Spring Boot可以帮助您运行[full end-to-end tests with an actual server](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-running-server).

### 45.3.11自动配置Spring WebFlux测试

要测试[Spring WebFlux](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference//web-reactive.html)控制器是否按预期工作，可以使用 `@WebFluxTest` 注释.  `@WebFluxTest` 自动配置Spring WebFlux基础结构并将扫描的bean限制为 `@Controller` ， `@ControllerAdvice` ， `@JsonComponent` ， `Converter` ， `GenericConverter` 和 `WebFluxConfigurer` .使用 `@WebFluxTest` 注释时，不扫描常规 `@Component`  bean.

>   `@WebFluxTest` 启用的自动配置列表可以是[found in the appendix](test-auto-configuration.html).

> 如果您需要注册额外的组件，例如Jackson  `Module` ，您可以在测试中使用 `@Import` 导入其他配置类.

通常， `@WebFluxTest` 仅限于单个控制器，并与 `@MockBean` 注释结合使用，以便为所需的协作者提供模拟实现.

`@WebFluxTest` 还自动配置[WebTestClient](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#webtestclient)，它提供了一种快速测试WebFlux控制器的强大方法，无需启动完整的HTTP服务器.

> 您还可以通过使用 `@AutoConfigureWebTestClient` 注释，在非 `@WebFluxTest` （例如 `@SpringBootTest` ）中自动配置 `WebTestClient` .以下示例显示了一个同时使用 `@WebFluxTest` 和 `WebTestClient` 的类：

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

> 此设置仅受WebFlux应用程序支持，因为在模拟的Web应用程序中使用 `WebTestClient` 仅适用于WebFlux.

>  `@WebFluxTest` 无法检测通过功能Web框架注册的路由.要在上下文中测试 `RouterFunction`  beans，请考虑通过 `@Import` 或使用 `@SpringBootTest` 自行导入 `RouterFunction` .

_0015.Sometimes撰写Spring WebFlux测试是不够的; Spring Boot可以帮助你运行[full end-to-end tests with an actual server](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-running-server).

### 45.3.12自动配置数据JPA测试

您可以使用 `@DataJpaTest` 批注来测试JPA应用程序.默认情况下，它配置内存中的嵌入式数据库，扫描 `@Entity` 类，并配置Spring Data JPA存储库.常规 `@Component`  bean未加载到 `ApplicationContext` 中.

>   `@DataJpaTest` 启用的自动配置设置列表可以是[found in the appendix](test-auto-configuration.html).

默认情况下，数据JPA测试是事务性的，并在每次测试结束时回滚.有关更多详细信息，请参阅Spring Framework参考文档中的[relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions).如果这不是您想要的，您可以为测试或整个类禁用事务管理，如下所示：

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

数据JPA测试也可以注入一个[TestEntityManager](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-test-autoconfigure/src/main/java/org/springframework/boot/test/autoconfigure/orm/jpa/TestEntityManager.java) bean，它提供了专门为测试设计的标准JPA  `EntityManager` 的替代方法.如果要在 `@DataJpaTest` 实例之外使用 `TestEntityManager` ，还可以使用 `@AutoConfigureTestEntityManager` 注释.如果您需要，也可以使用 `JdbcTemplate` .以下示例显示正在使用的 `@DataJpaTest` 注释：

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

内存中嵌入式数据库通常适用于测试，因为它们速度快且不需要任何安装.但是，如果您更喜欢对真实数据库运行测试，则可以使用 `@AutoConfigureTestDatabase` 注释，如以下示例所示：

```java
@RunWith(SpringRunner.class)
@DataJpaTest
@AutoConfigureTestDatabase(replace=Replace.NONE)
public class ExampleRepositoryTests {

	// ...

}
```

### 45.3.13自动配置的JDBC测试

`@JdbcTest` 类似于 `@DataJpaTest` ，但适用于仅需要 `DataSource` 并且不使用Spring Data JDBC的测试.默认情况下，它配置内存中的嵌入式数据库和 `JdbcTemplate` .常规 `@Component`  bean未加载到 `ApplicationContext` 中.

>   `@JdbcTest` 启用的自动配置列表可以是[found in the appendix](test-auto-configuration.html).

默认情况下，JDBC测试是事务性的，并在每次测试结束时回滚.有关更多详细信息，请参阅Spring Framework参考文档中的[relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions).如果这不是您想要的，您可以禁用测试或整个类的事务管理，如下所示：

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

如果您希望测试针对真实数据库运行，则可以使用 `@AutoConfigureTestDatabase` 注释，方法与 `DataJpaTest` 相同. （参见“[Section 45.3.12, “Auto-configured Data JPA Tests”](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test)”.）

### 45.3.14自动配置的数据JDBC测试

`@DataJdbcTest` 类似于 `@JdbcTest` ，但适用于使用Spring Data JDBC存储库的测试.默认情况下，它配置内存中的嵌入式数据库， `JdbcTemplate` 和Spring Data JDBC存储库.常规 `@Component`  bean未加载到 `ApplicationContext` 中.

>   `@DataJdbcTest` 启用的自动配置列表可以是[found in the appendix](test-auto-configuration.html).

默认情况下，数据JDBC测试是事务性的，并在每次测试结束时回滚.请参阅Spring Framework Reference中的[relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions)文档了解更多详情.如果这不是您想要的，您可以将测试或整个测试类的事务管理禁用为[shown in the JDBC example](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jdbc-test).

如果您希望测试针对真实数据库运行，则可以使用 `@AutoConfigureTestDatabase` 注释，方法与 `DataJpaTest` 相同. （参见“[Section 45.3.12, “Auto-configured Data JPA Tests”](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jpa-test)”.）

### 45.3.15自动配置的jOOQ测试

您可以使用与 `@JdbcTest` 类似的方式使用 `@JooqTest` ，但是对于与jOOQ相关的测试.由于jOOQ严重依赖于与数据库模式对应的基于Java的模式，因此使用现有的 `DataSource` .如果要将其替换为内存数据库，可以使用 `@AutoConfigureTestDatabase` 覆盖这些设置. （有关在Spring Boot中使用jOOQ的更多信息，请参阅本章前面的“[Section 30.6, “Using jOOQ”](boot-features-sql.html#boot-features-jooq)”.）常规的 `@Component` bean未加载到 `ApplicationContext` 中.

>   `@JooqTest` 启用的自动配置列表可以是[found in the appendix](test-auto-configuration.html).

`@JooqTest` 配置 `DSLContext` .常规 `@Component`  bean未加载到 `ApplicationContext` 中.以下示例显示正在使用的 `@JooqTest` 注释：

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

JOOQ测试是事务性的，默认情况下在每个测试结束时回滚.如果这不是您想要的，您可以将测试或整个测试类的事务管理禁用为[shown in the JDBC example](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-jdbc-test).

### 45.3.16自动配置的数据MongoDB测试

您可以使用 `@DataMongoTest` 来测试MongoDB应用程序.默认情况下，它配置内存中嵌入的MongoDB（如果可用），配置 `MongoTemplate` ，扫描 `@Document` 类，并配置Spring Data MongoDB存储库.常规 `@Component`  bean未加载到 `ApplicationContext` 中. （有关将MongoDB与Spring Boot一起使用的更多信息，请参阅本章前面的“[Section 31.2, “MongoDB”](boot-features-nosql.html#boot-features-mongodb)”.）

>   `@DataMongoTest` 启用的自动配置设置列表可以是[found in the appendix](test-auto-configuration.html).

以下类显示正在使用的 `@DataMongoTest` 注释：

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

内存中嵌入式MongoDB通常适用于测试，因为它速度快，不需要任何开发人员安装.但是，如果您更喜欢对真正的MongoDB服务器运行测试，则应排除嵌入式MongoDB自动配置，如以下示例所示：

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

### 45.3.17自动配置数据Neo4j测试

您可以使用 `@DataNeo4jTest` 来测试Neo4j应用程序.默认情况下，它使用内存中嵌入式Neo4j（如果嵌入式驱动程序可用），扫描 `@NodeEntity` 类，并配置Spring Data Neo4j存储库.常规 `@Component`  bean未加载到 `ApplicationContext` 中. （有关在Spring Boot中使用Neo4J的更多信息，请参阅本章前面的“[Section 31.3, “Neo4j”](boot-features-nosql.html#boot-features-neo4j)”.）

>   `@DataNeo4jTest` 启用的自动配置设置列表可以是[found in the appendix](test-auto-configuration.html).

以下示例显示了在Spring Boot中使用Neo4J测试的典型设置：

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

默认情况下，Data Neo4j测试是事务性的，并在每次测试结束时回滚.有关更多详细信息，请参阅Spring Framework Reference Documentation中的[relevant section](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/testing.html#testcontext-tx-enabling-transactions).如果这不是您想要的，您可以禁用测试或整个类的事务管理，如下所示：

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

### 45.3.18自动配置的数据Redis测试

您可以使用 `@DataRedisTest` 来测试Redis应用程序.默认情况下，它会扫描 `@RedisHash` 类并配置Spring Data Redis存储库.常规 `@Component`  bean未加载到 `ApplicationContext` 中. （有关使用带有Spring Boot的Redis的更多信息，请参阅本章前面的“[Section 31.1, “Redis”](boot-features-nosql.html#boot-features-redis)”.）

>   `@DataRedisTest` 启用的自动配置设置列表可以是[found in the appendix](test-auto-configuration.html).

以下示例显示正在使用的 `@DataRedisTest` 注释：

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

### 45.3.19自动配置的数据LDAP测试

您可以使用 `@DataLdapTest` 来测试LDAP应用程序.默认情况下，它配置内存中嵌入式LDAP（如果可用），配置 `LdapTemplate` ，扫描 `@Entry` 类，并配置Spring Data LDAP存储库.常规 `@Component`  bean未加载到 `ApplicationContext` 中. （有关在Spring Boot中使用LDAP的更多信息，请参阅本章前面的“[Section 31.9, “LDAP”](boot-features-nosql.html#boot-features-ldap)”.）

>   `@DataLdapTest` 启用的自动配置设置列表可以是[found in the appendix](test-auto-configuration.html).

以下示例显示正在使用的 `@DataLdapTest` 注释：

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

内存中嵌入式LDAP通常适用于测试，因为它速度快，不需要任何开发人员安装.但是，如果您希望针对真实LDAP服务器运行测试，则应排除嵌入式LDAP自动配置，如以下示例所示：

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

### 45.3.20自动配置的REST客户端

您可以使用 `@RestClientTest` 批注来测试REST客户端.默认情况下，它会自动配置Jackson，GSON和Jsonb支持，配置 `RestTemplateBuilder` ，并添加对 `MockRestServiceServer` 的支持.常规 `@Component`  bean未加载到 `ApplicationContext` 中.

>   `@RestClientTest` 启用的自动配置设置列表可以是[found in the appendix](test-auto-configuration.html).

应使用 `@RestClientTest` 的 `value` 或 `components` 属性指定要测试的特定bean，如以下示例所示：

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

### 45.3.21自动配置的Spring REST Docs测试

您可以使用 `@AutoConfigureRestDocs` 批注在Mock MVC或REST Assured的测试中使用[Spring REST Docs](https://projects.spring.io/spring-restdocs/).它消除了对Spring REST Docs中JUnit规则的需求.

`@AutoConfigureRestDocs` 可用于覆盖默认值输出目录（如果使用Maven则为 `target/generated-snippets` ，如果使用Gradle则为 `build/generated-snippets` ）.它还可用于配置出现在任何已记录的URI中的主机，方案和端口.

#### Auto配置的Spring REST Docs使用Mock MVC进行测试

`@AutoConfigureRestDocs` 自定义 `MockMvc`  bean以使用Spring REST Docs.您可以使用 `@Autowired` 注入它，并像在使用Mock MVC和Spring REST Docs时一样在测试中使用它，如以下示例所示：

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

如果您需要更多地控制Spring REST Docs配置，而不是 `@AutoConfigureRestDocs` 的属性，则可以使用 `RestDocsMockMvcConfigurationCustomizer`  bean，如以下示例所示：

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

如果要对参数化输出目录使用Spring REST Docs支持，可以创建 `RestDocumentationResultHandler`  bean.自动配置使用此结果处理程序调用 `alwaysDo` ，从而使每个 `MockMvc` 调用自动生成默认代码段.以下示例显示了正在定义的 `RestDocumentationResultHandler` ：

```java
@TestConfiguration
static class ResultHandlerConfiguration {

	@Bean
	public RestDocumentationResultHandler restDocumentation() {
		return MockMvcRestDocumentation.document("{method-name}");
	}

}
```

#### Auto配置的Spring REST Docs测试与REST Assured

`@AutoConfigureRestDocs` 创建一个 `RequestSpecification`  bean，预先配置为使用Spring REST Docs，可用于您的测试.您可以使用 `@Autowired` 注入它，并像在使用REST Assured和Spring REST Docs时一样在测试中使用它，如以下示例所示：

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

如果您需要更多地控制Spring REST Docs配置而不是 `@AutoConfigureRestDocs` 的属性，则可以使用 `RestDocsRestAssuredConfigurationCustomizer`  bean，如以下示例所示：

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

### 45.3.22其他自动配置和切片

每个切片提供一个或多个 `@AutoConfigure…` 注释，即定义应作为切片的一部分包括的自动配置.可以通过创建自定义 `@AutoConfigure…` 注释或仅通过将 `@ImportAutoConfiguration` 添加到测试来添加其他自动配置，如以下示例所示：

```java
@RunWith(SpringRunner.class)
@JdbcTest
@ImportAutoConfiguration(IntegrationAutoConfiguration.class)
public class ExampleJdbcTests {

}
```

> 确保不要使用常规 `@Import` 注释来导入自动配置，因为它们是由Spring Boot以特定方式处理的.

### 45.3.23用户配置和切片

如果你以合理的方式[structure your code](using-boot-structuring-your-code.html)，你的 `@SpringBootApplication` 类是[used by default](boot-features-testing.html#boot-features-testing-spring-boot-applications-detecting-config)作为测试的配置.

然后，重要的是不要使用特定于其功能的特定区域的配置设置来丢弃应用程序的主类.

假设您使用的是Spring Batch，并依赖于它的自动配置.您可以按如下方式定义 `@SpringBootApplication` ：

```java
@SpringBootApplication
@EnableBatchProcessing
public class SampleApplication { ... }
```

因为此类是测试的源配置，所以任何切片测试实际上都会尝试启动Spring Batch，这绝对不是您想要做的.建议的方法是将特定于区域的配置移动到与应用程序相同级别的单独 `@Configuration` 类，如以下示例所示：

```java
@Configuration
@EnableBatchProcessing
public class BatchConfiguration { ... }
```

> 根据应用程序的复杂程度，您可能只有一个 `@Configuration` 类用于自定义，或者每个域区域有一个类.后一种方法允许您在必要时使用 `@Import` 注释在其中一个测试中启用它.

混淆的另一个原因是类路径扫描.假设您以合理的方式构建代码，则需要扫描其他包.您的应用程序可能类似于以下代码：

```java
@SpringBootApplication
@ComponentScan({ "com.example.app", "org.acme.another" })
public class SampleApplication { ... }
```

这样做会有效地覆盖默认的组件扫描指令，无论您选择哪个切片，都会扫描这两个包.例如， `@DataJpaTest` 似乎突然扫描应用程序的组件和用户配置.同样，将自定义指令移动到单独的类是解决此问题的好方法.

> 如果这不是您的选项，您可以在测试的层次结构中的某处创建一个 `@SpringBootConfiguration` ，以便使用它.或者，您可以为测试指定源，这会禁用查找默认源的行为.

### 45.3.24使用Spock测试Spring Boot应用程序

如果您希望使用Spock来测试Spring Boot应用程序，您应该将Spock的 `spock-spring` 模块的依赖项添加到您的应用程序的构建中.  `spock-spring` 将Spring的测试框架集成到Spock中.建议您使用Spock 1.1或更高版本来受益于对Spock的Spring Framework和Spring Boot集成的一些改进.有关详细信息，请参阅[the documentation for Spock’s Spring module](http://spockframework.org/spock/docs/1.1/modules.html).

## 45.4测试实用程序

测试应用程序时通常有用的一些测试实用程序类打包为 `spring-boot` 的一部分.

### 45.4.1 ConfigFileApplicationContextInitializer

`ConfigFileApplicationContextInitializer` 是 `ApplicationContextInitializer` ，您可以将其应用于测试以加载Spring Boot  `application.properties` 文件.当您不需要 `@SpringBootTest` 提供的全部功能时，可以使用它，如以下示例所示：

```java
@ContextConfiguration(classes = Config.class,
	initializers = ConfigFileApplicationContextInitializer.class)
```

> 仅使用 `ConfigFileApplicationContextInitializer` 不支持 `@Value("${…}")` 注入.它唯一的工作是确保将 `application.properties` 文件加载到Spring的 `Environment` 中.对于 `@Value` 支持，您需要另外配置 `PropertySourcesPlaceholderConfigurer` 或使用 `@SpringBootTest` ，它会自动为您配置一个.

### 45.4.2 TestPropertyValues

`TestPropertyValues` 允许您快速添加属性 `ConfigurableEnvironment` 或 `ConfigurableApplicationContext` .您可以使用 `key=value` 字符串调用它，如下所示：

```java
TestPropertyValues.of("org=Spring", "name=Boot").applyTo(env);
```

### 45.4.3 OutputCapture

`OutputCapture` 是一个JUnit  `Rule` ，可用于捕获 `System.out` 和 `System.err` 输出.您可以将捕获声明为 `@Rule` ，然后使用 `toString()` 进行断言，如下所示：

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

> Spring Framework 5.0提供了一个适用于[WebFlux integration tests](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-autoconfigured-webflux-tests)和[WebFlux and MVC end-to-end testing](boot-features-testing.html#boot-features-testing-spring-boot-applications-testing-with-running-server)的新 `WebTestClient` .与 `TestRestTemplate` 不同，它为断言提供了流畅的API.

`TestRestTemplate` 是Spring的 `RestTemplate` 的便利替代品，在集成测试中很有用.您可以获得一个vanilla模板或一个发送基本HTTP身份验证（使用用户名和密码）的模板.在任何一种情况下，模板都以一种测试友好的方式运行，不会在服务器端错误上抛出异常.建议（但不是强制性的）使用Apache HTTP Client（版本4.3.2或更高版本）.如果您在类路径中有这个，则 `TestRestTemplate` 通过适当地配置客户端来响应.如果您确实使用Apache的HTTP客户端，则启用一些其他测试友好功能：

不遵循
- Redirects（因此您可以断言响应位置）.

- Cookies被忽略（因此模板是无状态的）.

`TestRestTemplate` 可以直接在集成测试中实例化，如以下示例所示：

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

或者，如果将 `@SpringBootTest` 注释与 `WebEnvironment.RANDOM_PORT` 或 `WebEnvironment.DEFINED_PORT` 一起使用，则可以注入完全配置的 `TestRestTemplate` 并开始使用它.如有必要，可以通过 `RestTemplateBuilder`  bean应用其他自定义.任何未指定主机和端口的URL都会自动连接到嵌入式服务器，如以下示例所示：

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


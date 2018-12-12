## 76. Spring Boot应用程序

本节包括与Spring Boot应用程序直接相关的主题.

## 76.1创建自己的失败分析器

[FailureAnalyzer](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/diagnostics/FailureAnalyzer.html)是一种在启动时拦截异常并将其转换为人类可读消息的好方法，包含在[FailureAnalysis](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/diagnostics/FailureAnalysis.html)中. Spring Boot为应用程序上下文相关异常，JSR-303验证等提供了这样的分析器.您也可以创建自己的.

`AbstractFailureAnalyzer` 是 `FailureAnalyzer` 的便捷扩展，用于检查要处理的异常中是否存在指定的异常类型.您可以从中进行扩展，以便您的实现只有在实际存在时才有机会处理异常.如果由于某种原因，您无法处理异常，请返回 `null` 以使另一个实现有机会处理异常.

`FailureAnalyzer` 实现必须在 `META-INF/spring.factories` 中注册.以下示例注册 `ProjectConstraintViolationFailureAnalyzer` ：

```java
org.springframework.boot.diagnostics.FailureAnalyzer=\
com.example.ProjectConstraintViolationFailureAnalyzer
```

> 如果您需要访问 `BeanFactory` 或 `Environment` ，您的 `FailureAnalyzer` 可以分别实现 `BeanFactoryAware` 或 `EnvironmentAware` .

## 76.2自动配置疑难解答

Spring Boot自动配置尽力“做正确的事”，但有时事情会失败，而且很难说清楚原因.

在任何Spring Boot  `ApplicationContext` 中都有一个非常有用的 `ConditionEvaluationReport` .如果启用 `DEBUG` 日志记录输出，则可以看到它.如果您使用 `spring-boot-actuator` （请参阅[the Actuator chapter]()），还有一个 `conditions` endpoints以JSON格式呈现报表.使用该endpoints调试应用程序，并查看Spring Boot在运行时添加了哪些功能（以及尚未添加的功能）.

通过查看源代码和Javadoc可以回答更多问题.阅读代码时，请记住以下经验法则：

- Look为名为 `*AutoConfiguration` 的类并读取它们的来源.请特别注意 `@Conditional*` 注释，以了解它们启用哪些功能以及何时启用.将 `--debug` 添加到命令行或系统属性 `-Ddebug` ，以在控制台上记录应用程序中所做的所有自动配置决策.在正在运行的Actuator应用程序中，查看 `conditions` endpoints（ `/actuator/conditions` 或JMX等效项）以获取相同的信息.

- Look为 `@ConfigurationProperties` （例如[ServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/ServerProperties.java)）的类，并从那里读取可用的外部配置选项.  `@ConfigurationProperties` 注释具有 `name` 属性，该属性充当外部属性的前缀.因此， `ServerProperties` 具有 `prefix="server"` ，其配置属性为 `server.port` ， `server.address` 等.在正在运行的Actuator应用程序中，查看 `configprops` endpoints.

- Look在 `Binder` 上使用 `bind` 方法以轻松的方式从 `Environment` 中明确地提取配置值.它通常与前缀一起使用.

- Look用于直接绑定到 `Environment` 的 `@Value` 注释.

- Look for  `@ConditionalOnExpression` 注释，用于响应SpEL表达式打开和关闭功能，通常使用从 `Environment` 解析的占位符进行评估.

## 76.3在开始之前自定义环境或ApplicationContext

`SpringApplication` 具有 `ApplicationListeners` 和 `ApplicationContextInitializers` ，用于将自定义应用于上下文或环境. Spring Boot加载了许多此类自定义，以便在 `META-INF/spring.factories` 内部使用.注册其他自定义项的方法不止一种：

- Programmatically，每个应用程序，通过在运行它之前调用 `SpringApplication` 上的 `addListeners` 和 `addInitializers` 方法.

- Declaratively，每个应用程序，通过设置 `context.initializer.classes` 或 `context.listener.classes` 属性.

- Declaratively，对于所有应用程序，通过添加 `META-INF/spring.factories` 并打包应用程序全部用作库的jar文件.

`SpringApplication` 向侦听器发送一些特殊的 `ApplicationEvents` （有些甚至在创建上下文之前），然后为 `ApplicationContext` 发布的事件注册侦听器.有关完整列表，请参阅“Spring Boot功能”部分中的“[Section 23.5, “Application Events and Listeners”](boot-features-spring-application.html#boot-features-application-events-and-listeners)”.

在使用 `EnvironmentPostProcessor` 刷新应用程序上下文之前，还可以自定义 `Environment` .每个实现都应在 `META-INF/spring.factories` 中注册，如以下示例所示：

```java
org.springframework.boot.env.EnvironmentPostProcessor=com.example.YourEnvironmentPostProcessor
```

该实现可以加载任意文件并将它们添加到 `Environment` .例如，以下示例从类路径加载YAML配置文件：

```java
public class EnvironmentPostProcessorExample implements EnvironmentPostProcessor {

	private final YamlPropertySourceLoader loader = new YamlPropertySourceLoader();

	@Override
	public void postProcessEnvironment(ConfigurableEnvironment environment,
			SpringApplication application) {
		Resource path = new ClassPathResource("com/example/myapp/config.yml");
		PropertySource<?> propertySource = loadYaml(path);
		environment.getPropertySources().addLast(propertySource);
	}

	private PropertySource<?> loadYaml(Resource path) {
		if (!path.exists()) {
			throw new IllegalArgumentException("Resource " + path + " does not exist");
		}
		try {
			return this.loader.load("custom-resource", path).get(0);
		}
		catch (IOException ex) {
			throw new IllegalStateException(
					"Failed to load yaml configuration from " + path, ex);
		}
	}

}
```

>   `Environment` 已经准备好了Spring Boot默认加载的所有常用属性.因此，可以从环境中获取文件的位置.上面的示例在列表末尾添加 `custom-resource` 属性源以便在任何通常的其他位置中定义的密钥优先.自定义实现可以定义另一个订单.

> 虽然在 `@SpringBootApplication` 上使用 `@PropertySource` 似乎是在 `Environment` 中加载自定义资源的一种方便而简单的方法，但我们不建议这样做，因为Spring Boot会在 `ApplicationContext` 刷新之前准备好 `Environment` .使用 `@PropertySource` 定义的任何键加载太晚，不会对自动配置产生任何影响.

## 76.4构建ApplicationContext层次结构（添加父或根上下文）

您可以使用 `ApplicationBuilder` 类创建父/子 `ApplicationContext` 层次结构.有关详细信息，请参阅“Spring Boot功能”部分中的“[Section 23.4, “Fluent Builder API”](boot-features-spring-application.html#boot-features-fluent-builder-api)”.

## 76.5创建非Web应用程序

并非所有Spring应用程序都必须是Web应用程序（或Web服务）.如果要在 `main` 方法中执行某些代码，而且还要引导Spring应用程序来设置要使用的基础结构，则可以使用Spring Boot的 `SpringApplication` 功能.  `SpringApplication` 更改其 `ApplicationContext` 类，具体取决于它是否认为它需要Web应用程序.您可以做的第一件事就是将与服务器相关的依赖项（例如servlet API）从类路径中删除.如果您不能这样做（例如，您从相同的代码库运行两个应用程序），那么您可以在 `SpringApplication` 实例上显式调用 `setWebApplicationType(WebApplicationType.NONE)` 或设置 `applicationContextClass` 属性（通过Java API或外部属性）.您希望作为业务逻辑运行的应用程序代码可以实现为 `CommandLineRunner` ，并作为 `@Bean` 定义放入上下文中.


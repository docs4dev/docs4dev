## 79. Spring MVC

Spring Boot有许多包含Spring MVC的启动器.请注意，一些启动器包含对Spring MVC的依赖，而不是直接包含它.本节回答有关Spring MVC和Spring Boot的常见问题.

## 79.1编写JSON REST服务

只要Jackson2在类路径中，Spring Boot应用程序中的任何Spring  `@RestController` 都应默认呈现JSON响应，如以下示例所示：

```java
@RestController
public class MyController {

	@RequestMapping("/thing")
	public MyThing thing() {
			return new MyThing();
	}

}
```

只要_877_可以被Jackson2序列化（对于普通的POJO或Groovy对象来说是真的），那么 `localhost:8080/thing` 默认为它提供JSON表示.请注意，在浏览器中，您有时可能会看到XML响应，因为浏览器倾向于发送更喜欢XML的接受标头.

## 79.2编写XML REST服务

如果在类路径上有Jackson XML扩展（ `jackson-dataformat-xml` ），则可以使用它来呈现XML响应.我们用于JSON的前一个示例可以使用.要使用Jackson XML渲染器，请将以下依赖项添加到项目中：

```xml
<dependency>
	<groupId>com.fasterxml.jackson.dataformat</groupId>
	<artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

您可能还想在Woodstox上添加依赖项.它比JDK提供的默认StAX实现更快，并且还增加了漂亮的打印支持和改进的命名空间处理.以下清单显示了如何在[Woodstox](https://github.com/FasterXML/woodstox)上包含依赖项：

```xml
<dependency>
	<groupId>org.codehaus.woodstox</groupId>
	<artifactId>woodstox-core-asl</artifactId>
</dependency>
```

如果Jackson的XML扩展不可用，则使用JAXB（默认情况下在JDK中提供），并附加要求 `MyThing` 注释为 `@XmlRootElement` ，如以下示例所示：

```java
@XmlRootElement
public class MyThing {
	private String name;
	// .. getters and setters
}
```

要使服务器呈现XML而不是JSON，您可能必须发送 `Accept: text/xml` 标头（或使用浏览器）.

## 79.3自定义Jackson ObjectMapper

Spring MVC（客户端和服务器端）使用 `HttpMessageConverters` 在HTTP交换中协商内容转换.如果Jackson在类路径上，您已经获得了 `Jackson2ObjectMapperBuilder` 提供的默认转换器，其中一个实例是为您自动配置的.

`ObjectMapper` （或用于Jackson XML转换器的 `XmlMapper` ）实例（默认情况下创建）具有以下自定义属性：

-  `MapperFeature.DEFAULT_VIEW_INCLUSION` 已禁用

-  `DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES` 已禁用

-  `SerializationFeature.WRITE_DATES_AS_TIMESTAMPS` 已禁用

Spring Boot还具有一些功能，可以更轻松地自定义此行为.

您可以使用环境配置 `ObjectMapper` 和 `XmlMapper` 实例. Jackson提供了一套广泛的简单开/关功能，可用于配置其处理的各个方面.这些功能在六个enum（在Jackson中）中描述，它们映射到环境中的属性：

|枚举|地产|Value观|
| ---- | ---- | ---- |
|  `com.fasterxml.jackson.databind.DeserializationFeature`  |  `spring.jackson.deserialization.<feature_name>`  |  `true` ， `false`  |
|  `com.fasterxml.jackson.core.JsonGenerator.Feature`  |  `spring.jackson.generator.<feature_name>`  |  `true` ， `false`  |
|  `com.fasterxml.jackson.databind.MapperFeature`  |  `spring.jackson.mapper.<feature_name>`  |  `true` ， `false`  |
|  `com.fasterxml.jackson.core.JsonParser.Feature`  |  `spring.jackson.parser.<feature_name>`  |  `true` ， `false`  |
|  `com.fasterxml.jackson.databind.SerializationFeature`  |  `spring.jackson.serialization.<feature_name>`  |  `true` ， `false`  |
|  `com.fasterxml.jackson.annotation.JsonInclude.Include`  |  `spring.jackson.default-property-inclusion`  |  `always` ， `non_null` ， `non_absent` ， `non_default` ， `non_empty`  |

例如，要启用漂亮打印，请设置 `spring.jackson.serialization.indent_output=true` .注意，由于使用[relaxed binding](boot-features-external-config.html#boot-features-external-config-relaxed-binding)， `indent_output` 的情况不必与相应的枚举常量（ `INDENT_OUTPUT` ）的情况相匹配.

此基于环境的配置应用于自动配置的 `Jackson2ObjectMapperBuilder`  bean，并应用于使用构建器创建的任何映射器，包括自动配置的 `ObjectMapper`  bean.

上下文的 `Jackson2ObjectMapperBuilder` 可以由一个或多个 `Jackson2ObjectMapperBuilderCustomizer`  bean自定义.可以订购这样的定制器bean（Boot自己的定制器的顺序为0），允许在Boot定制之前和之后应用其他定制.

任何 `com.fasterxml.jackson.databind.Module` 类型的bean都会自动注册自动配置的 `Jackson2ObjectMapperBuilder` ，并应用于它创建的任何 `ObjectMapper` 实例.这为您在应用程序中添加新功能时提供了一种全局机制，用于提供自定义模块.

如果要完全替换默认的 `ObjectMapper` ，请定义该类型的 `@Bean` 并将其标记为 `@Primary` ，或者，如果您更喜欢基于构建器的方法，请定义 `Jackson2ObjectMapperBuilder`   `@Bean` .请注意，在任何一种情况下，这样做都会禁用 `ObjectMapper` 的所有自动配置.

如果您提供 `MappingJackson2HttpMessageConverter` 类型的任何 `@Beans` ，它们将替换MVC配置中的默认值.此外，还提供了类型为 `HttpMessageConverters` 的便捷bean（如果使用默认的MVC配置，则始终可用）.它有一些有用的方法来访问默认和用户增强的消息转换器.

有关详细信息，请参阅“[Section 79.4, “Customize the @ResponseBody Rendering”](howto-spring-mvc.html#howto-customize-the-responsebody-rendering)”部分和[WebMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration.java)源代码.

## 79.4自定义@ResponseBody渲染

Spring使用 `HttpMessageConverters` 来渲染 `@ResponseBody` （或来自 `@RestController` 的响应）.您可以通过在Spring Boot上下文中添加适当类型的bean来提供其他转换器.如果您添加的bean是默认包含的类型（例如，对于JSON转换为 `MappingJackson2HttpMessageConverter` ），它将替换默认值.提供了类型为 `HttpMessageConverters` 的便捷bean，如果您使用默认的MVC配置，它始终可用.它有一些有用的方法来访问默认和用户增强的消息转换器（例如，如果您想将它们手动注入自定义_5953中，它会很有用）.

与正常的MVC用法一样，您提供的任何 `WebMvcConfigurer`  bean也可以通过覆盖 `configureMessageConverters` 方法来提供转换器.但是，与普通MVC不同，您只能提供所需的其他转换器（因为Spring Boot使用相同的机制来提供其默认值）.最后，如果您通过提供自己的 `@EnableWebMvc` 配置选择退出Spring Boot默认MVC配置，则可以完全控制并使用 `getMessageConverters` 从 `WebMvcConfigurationSupport` 手动完成所有操作.

有关更多详细信息，请参阅[WebMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration.java)源代码.

## 79.5处理多部分文件上传

Spring Boot包含Servlet 3  `javax.servlet.http.Part`  API以支持上传文件.默认情况下，Spring Boot配置Spring MVC，每个文件的最大大小为1MB，最大文件数据为10MB单一请求.您可以覆盖这些值，存储中间数据的位置（例如，到 `/tmp` 目录），以及使用 `MultipartProperties` 类中公开的属性将数据刷新到磁盘的阈值.例如，如果要指定文件不受限制，请将 `spring.servlet.multipart.max-file-size` 属性设置为 `-1` .

当您希望在Spring MVC控制器处理程序方法中将多部分编码的文件数据作为 `@RequestParam` 注释的 `MultipartFile` 类型的参数接收时，多部分支持非常有用.

有关详细信息，请参阅[MultipartAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/MultipartAutoConfiguration.java)源.

> 建议使用容器的内置支持进行分段上传，而不是引入额外的依赖项，例如Apache Commons File Upload.

## 79.6关闭Spring MVC DispatcherServlet

默认情况下，所有内容都是从应用程序的根目录（ `/` ）提供的.如果您希望映射到其他路径，可以按如下方式配置：

```java
spring.mvc.servlet.path=/acme
```

如果你有额外的servlet，你可以为每个声明 `@Bean` 类型 `Servlet` 或 `ServletRegistrationBean` ，Spring Boot会将它们透明地注册到容器中.因为servlet是以这种方式注册的，所以可以将它们映射到 `DispatcherServlet` 的子上下文而不调用它.

自己配置 `DispatcherServlet` 是不寻常的，但如果你真的需要这样做，还必须提供类型为 `DispatcherServletPath` 的 `@Bean` ，以提供自定义 `DispatcherServlet` 的路径.

## 79.7关闭默认MVC配置

完全控制MVC配置的最简单方法是使用 `@EnableWebMvc` 注释提供自己的 `@Configuration` .这样做会将所有MVC配置留在您的手中.

## 79.8自定义ViewResolvers

`ViewResolver` 是Spring MVC的核心组件，将 `@Controller` 中的视图名称转换为实际的 `View` 实现.请注意， `ViewResolvers` 主要用于UI应用程序，而不是REST样式的服务（ `View` 不用于呈现 `@ResponseBody` ）.有很多 `ViewResolver` 的实现可供选择，而Spring本身并不认为你应该使用哪些.另一方面，Spring Boot会为您安装一个或两个，具体取决于它在类路径和应用程序上下文中找到的内容.  `DispatcherServlet` 使用它在应用程序上下文中找到的所有解析器，依次尝试每个解析器直到得到结果，因此，如果添加自己的解析器，则必须知道顺序以及添加解析器的位置.

`WebMvcAutoConfiguration` 将以下 `ViewResolvers` 添加到您的上下文中：

- An  `InternalResourceViewResolver` 命名为'defaultViewResolver'.这个定位可以使用 `DefaultServlet` 呈现的物理资源（包括静态资源和JSP页面，如果您使用它们）.它将前缀和后缀应用于视图名称，然后在servlet上下文中查找具有该路径的物理资源（默认值为空，但可通过 `spring.mvc.view.prefix` 和 `spring.mvc.view.suffix` 进行外部配置）.您可以通过提供相同类型的bean来覆盖它.

- A  `BeanNameViewResolver` 命名为'beanNameViewResolver'.这是视图解析器链的一个有用成员，并获取与正在解析的 `View` 同名的任何bean.不必覆盖或替换它.
仅当 **are** 实际存在 `View` 类型的bean时，才会添加名为'viewResolver'的
- A  `ContentNegotiatingViewResolver` .这是一个“主”解析器，委托给所有其他人，并尝试找到与客户端发送的“Accept”HTTP头匹配的内容.有一个有用的[blog about ContentNegotiatingViewResolver](https://spring.io/blog/2013/06/03/content-negotiation-using-views)您可能想要学习以了解更多信息，您也可以查看源代码以获取详细信息.您可以通过定义名为“viewResolver”的bean来关闭自动配置的 `ContentNegotiatingViewResolver` .

- 如果您使用Thymeleaf，您还有 `ThymeleafViewResolver` 名为'thymeleafViewResolver'.它通过使用前缀和后缀包围视图名称来查找资源.前缀为 `spring.thymeleaf.prefix` ，后缀为 `spring.thymeleaf.suffix` .前缀和后缀的值分别默认为“classpath：/ templates /”和“.html”.您可以通过提供相同名称的bean来覆盖 `ThymeleafViewResolver` .

- 如果您使用FreeMarker，您还有 `FreeMarkerViewResolver` 名为'freeMarkerViewResolver'.它通过用前缀和后缀包围视图名称来查找加载器路径中的资源（外部化为 `spring.freemarker.templateLoaderPath` 并具有默认值'classpath：/ templates /'）.前缀外部化为 `spring.freemarker.prefix` ，后缀外部化为 `spring.freemarker.suffix` .前缀和后缀的默认值分别为空和“.ftl”.您可以通过提供相同名称的bean来覆盖 `FreeMarkerViewResolver` .

- 如果您使用Groovy模板（实际上，如果 `groovy-templates` 在您的类路径中），您还有一个名为'groovyMarkupViewResolver'的 `GroovyMarkupViewResolver` .它通过用前缀和后缀（外部化为 `spring.groovy.template.prefix` 和 `spring.groovy.template.suffix` ）包围视图名称来查找加载器路径中的资源.前缀和后缀分别具有“classpath：/ templates /”和“.tpl”的默认值.您可以通过提供相同名称的bean来覆盖 `GroovyMarkupViewResolver` .

有关更多详细信息，请参阅以下部分：

- [WebMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/WebMvcAutoConfiguration.java)

- [ThymeleafAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java)

- [FreeMarkerAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java)

- [GroovyTemplateAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java)


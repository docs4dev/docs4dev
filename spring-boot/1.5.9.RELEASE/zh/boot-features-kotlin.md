## 50. Kotlin支持

[Kotlin](https://kotlinlang.org)是一种定义JVM（和其他平台）的静态类型语言，它允许编写简洁而优雅的代码，同时为[interoperability](https://kotlinlang.org/docs/reference/java-interop.html)提供用Java编写的现有库.

Spring Boot通过利用其他Spring项目（如Spring Framework，Spring Data和Reactor）的支持来提供Kotlin支持.有关更多信息，请参阅[Spring Framework Kotlin support documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/languages.html#kotlin).

从Spring Boot和Kotlin开始的最简单方法是遵循[this comprehensive tutorial](https://spring.io/guides/tutorials/spring-boot-kotlin/).您可以通过[start.spring.io](https://start.spring.io/#!language=kotlin)创建新的Kotlin项目.如果您需要，请随意加入[Kotlin Slack](http://slack.kotlinlang.org/)的#springChannels或在[Stack Overflow](https://stackoverflow.com/questions/tagged/spring+kotlin)上使用 `spring` 和 `kotlin` 标签提问支持.

## 50.1要求

Spring Boot支持Kotlin 1.2.x.要使用Kotlin，类路径上必须存在 `org.jetbrains.kotlin:kotlin-stdlib` 和 `org.jetbrains.kotlin:kotlin-reflect` .也可以使用 `kotlin-stdlib` 变体 `kotlin-stdlib-jdk7` 和 `kotlin-stdlib-jdk8` .

从[Kotlin classes are final by default](https://discuss.kotlinlang.org/t/classes-final-by-default/166)开始，您可能想要配置[kotlin-spring](https://kotlinlang.org/docs/reference/compiler-plugins.html#spring-support)插件，以便自动打开Spring注释类，以便可以代理它们.

在Kotlin中序列化/反序列化JSON数据需要[Jackson’s Kotlin module](https://github.com/FasterXML/jackson-module-kotlin).在类路径中找到它时会自动注册.如果Jackson和Kotlin存在但Jackson Kotlin模块不存在，则会记录警告消息.

> 如果在[start.spring.io](https://start.spring.io/#!language=kotlin)上引导Kotlin项目，则默认提供这些依赖项和插件.

## 50.2无安全性

Kotlin的主要功能之一是[null-safety](https://kotlinlang.org/docs/reference/null-safety.html).它在编译时处理 `null` 值，而不是将问题推迟到运行时并遇到 `NullPointerException` .这有助于消除常见的错误来源，而无需支付像 `Optional` 这样的包装器的成本. Kotlin还允许使用具有可空值的函数结构，如[comprehensive guide to null-safety in Kotlin](http://www.baeldung.com/kotlin-null-safety)中所述.

尽管Java不允许在其类型系统中表达null安全性，但Spring Framework，Spring Data和Reactor现在通过易于使用工具的注释提供其API的安全性.默认情况下，Kotlin中使用的Java API类型被识别为[platform types](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types)，其中放宽了空检查. [Kotlin’s support for JSR 305 annotations](https://kotlinlang.org/docs/reference/java-interop.html#jsr-305-support)与可空性注释相结合，为Kotlin中的相关Spring API提供了空安全性.

可以通过添加带有以下选项的 `-Xjsr305` 编译器标志来配置JSR 305检查： `-Xjsr305={strict|warn|ignore}` .默认行为与 `-Xjsr305=warn` 相同.  `strict` 值需要在从Spring API推断的Kotlin类型中考虑空安全性，但应该使用Spring API可空性声明甚至可以在次要版本之间发展并且将来可能添加更多检查的知识.

> 尚未支持泛型类型参数，varargs和数组元素可为空性.有关最新信息，请参阅[SPR-15942](https://jira.spring.io/browse/SPR-15942).另请注意，Spring Boot自己的API是[not yet annotated](https://github.com/spring-projects/spring-boot/issues/10712).

## 50.3 Kotlin API

### 50.3.1 runApplication

Spring Boot提供了一种使用 `runApplication<MyApplication>(*args)` 运行应用程序的惯用方法，如以下示例所示：

```java
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
	runApplication<MyApplication>(*args)
}
```

这是 `SpringApplication.run(MyApplication::class.java, *args)` 的替代品.它还允许自定义应用程序，如以下示例所示：

```java
runApplication<MyApplication>(*args) {
	setBannerMode(OFF)
}
```

### 50.3.2扩展程序

Kotlin [extensions](https://kotlinlang.org/docs/reference/extensions.html)提供了使用附加功能扩展现有类的能力. Spring Boot Kotlin API利用这些扩展为现有API添加新的Kotlin特定便利.

提供了类似于Spring Framework中为 `RestOperations` 提供的扩展的 `TestRestTemplate` 扩展.除此之外，扩展使得可以利用Kotlin具体类型参数.

## 50.4依赖管理

为了避免在类路径上混合使用不同版本的Kotlin依赖项，提供了以下Kotlin依赖项的依赖项管理：

-  `kotlin-reflect` 

-  `kotlin-runtime` 

-  `kotlin-stdlib` 

-  `kotlin-stdlib-jdk7` 

-  `kotlin-stdlib-jdk8` 

-  `kotlin-stdlib-jre7` 

-  `kotlin-stdlib-jre8` 

使用Maven，可以通过 `kotlin.version` 属性自定义Kotlin版本，并为 `kotlin-maven-plugin` 提供插件管理.使用Gradle，Spring Boot插件会自动将 `kotlin.version` 与Kotlin插件的版本对齐.

## 50.5 @ConfigurationProperties

`@ConfigurationProperties` 目前仅适用于 `lateinit` 或可空的 `var` 属性（建议使用前者），因为构造函数初始化的不可变类是[not yet supported](https://github.com/spring-projects/spring-boot/issues/8762).

```java
@ConfigurationProperties("example.kotlin")
class KotlinExampleProperties {

	lateinit var name: String

	lateinit var description: String

	val myService = MyService()

	class MyService {

		lateinit var apiToken: String

		lateinit var uri: URI

	}

}
```

> 使用注释处理器生成[your own metadata](configuration-metadata.html#configuration-metadata-annotation-processor)，[kapt should be configured](https://kotlinlang.org/docs/reference/kapt.html)具有 `spring-boot-configuration-processor` 依赖性.

## 50.6测试

虽然可以使用JUnit 4（ `spring-boot-starter-test` 提供的默认值）来测试Kotlin代码，但建议使用JUnit 5. JUnit 5使测试类能够实例化一次并重用于所有类的测试.这使得在非静态方法上使用 `@BeforeAll` 和 `@AfterAll` 注释成为可能，这非常适合Kotlin.

要使用JUnit 5，请从 `spring-boot-starter-test` 中排除 `junit:junit` 依赖项，添加JUnit 5依赖项，并相应地配置Maven或Gradle插件.有关详细信息，请参阅[JUnit 5 documentation](https://junit.org/junit5/docs/current/user-guide/#dependency-metadata-junit-jupiter-samples).你还需要[switch test instance lifecycle to "per-class"](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle-changing-default).

## 50.7资源

### 50.7.1进一步阅读

- [Kotlin language reference](https://kotlinlang.org/docs/reference/)

- [Kotlin Slack](http://slack.kotlinlang.org/)（使用专用的#springChannels）

- [Stackoverflow with spring and kotlin tags](https://stackoverflow.com/questions/tagged/spring+kotlin)

- [Try Kotlin in your browser](https://try.kotlinlang.org/)

- [Kotlin blog](https://blog.jetbrains.com/kotlin/)

- [Awesome Kotlin](https://kotlin.link/)

- [Tutorial: building web applications with Spring Boot and Kotlin](https://spring.io/guides/tutorials/spring-boot-kotlin/)

- [Developing Spring Boot applications with Kotlin](https://spring.io/blog/2016/02/15/developing-spring-boot-applications-with-kotlin)

- [A Geospatial Messenger with Kotlin, Spring Boot and PostgreSQL](https://spring.io/blog/2016/03/20/a-geospatial-messenger-with-kotlin-spring-boot-and-postgresql)

- [Introducing Kotlin support in Spring Framework 5.0](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0)

- [Spring Framework 5 Kotlin APIs, the functional way](https://spring.io/blog/2017/08/01/spring-framework-5-kotlin-apis-the-functional-way)

### 50.7.2示例

- [spring-boot-kotlin-demo](https://github.com/sdeleuze/spring-boot-kotlin-demo)：常规Spring Boot Spring Data JPA项目

- [mixit](https://github.com/mixitconf/mixit)：Spring Boot 2 WebFlux Reactive Spring Data MongoDB

- [spring-kotlin-fullstack](https://github.com/sdeleuze/spring-kotlin-fullstack)：WebFlux Kotlin fullstack示例，其中Kotlin2js用于前端而不是JavaScript或TypeScript

- [spring-petclinic-kotlin](https://github.com/spring-petclinic/spring-petclinic-kotlin)：Spring PetClinic示例应用程序的Kotlin版本

- [spring-kotlin-deepdive](https://github.com/sdeleuze/spring-kotlin-deepdive)：将Boot 1.0 Java逐步迁移到Boot 2.0 Kotlin


## 50. Kotlin support

[Kotlin](https://kotlinlang.org) is a statically-typed language targeting the JVM (and other platforms) which allows writing concise and elegant code while providing [interoperability](https://kotlinlang.org/docs/reference/java-interop.html) with existing libraries written in Java.

Spring Boot provides Kotlin support by leveraging the support in other Spring projects such as Spring Framework, Spring Data, and Reactor. See the [Spring Framework Kotlin support documentation](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/languages.html#kotlin) for more information.

The easiest way to start with Spring Boot and Kotlin is to follow [this comprehensive tutorial](https://spring.io/guides/tutorials/spring-boot-kotlin/). You can create new Kotlin projects via [start.spring.io](https://start.spring.io/#!language=kotlin). Feel free to join the #spring channel of [Kotlin Slack](http://slack.kotlinlang.org/) or ask a question with the  `spring`  and  `kotlin`  tags on [Stack Overflow](https://stackoverflow.com/questions/tagged/spring+kotlin) if you need support.

## 50.1 Requirements

Spring Boot supports Kotlin 1.2.x. To use Kotlin,  `org.jetbrains.kotlin:kotlin-stdlib`  and  `org.jetbrains.kotlin:kotlin-reflect`  must be present on the classpath. The  `kotlin-stdlib`  variants  `kotlin-stdlib-jdk7`  and  `kotlin-stdlib-jdk8`  can also be used.

Since [Kotlin classes are final by default](https://discuss.kotlinlang.org/t/classes-final-by-default/166), you are likely to want to configure [kotlin-spring](https://kotlinlang.org/docs/reference/compiler-plugins.html#spring-support) plugin in order to automatically open Spring-annotated classes so that they can be proxied.

[Jackson’s Kotlin module](https://github.com/FasterXML/jackson-module-kotlin) is required for serializing / deserializing JSON data in Kotlin. It is automatically registered when found on the classpath. A warning message is logged if Jackson and Kotlin are present but the Jackson Kotlin module is not.

> These dependencies and plugins are provided by default if one bootstraps a Kotlin project on [start.spring.io](https://start.spring.io/#!language=kotlin).

## 50.2 Null-safety

One of Kotlin’s key features is [null-safety](https://kotlinlang.org/docs/reference/null-safety.html). It deals with  `null`  values at compile time rather than deferring the problem to runtime and encountering a  `NullPointerException` . This helps to eliminate a common source of bugs without paying the cost of wrappers like  `Optional` . Kotlin also allows using functional constructs with nullable values as described in this [comprehensive guide to null-safety in Kotlin](http://www.baeldung.com/kotlin-null-safety).

Although Java does not allow one to express null-safety in its type system, Spring Framework, Spring Data, and Reactor now provide null-safety of their API via tooling-friendly annotations. By default, types from Java APIs used in Kotlin are recognized as [platform types](https://kotlinlang.org/docs/reference/java-interop.html#null-safety-and-platform-types) for which null-checks are relaxed. [Kotlin’s support for JSR 305 annotations](https://kotlinlang.org/docs/reference/java-interop.html#jsr-305-support) combined with nullability annotations provide null-safety for the related Spring API in Kotlin.

The JSR 305 checks can be configured by adding the  `-Xjsr305`  compiler flag with the following options:  `-Xjsr305={strict|warn|ignore}` . The default behavior is the same as  `-Xjsr305=warn` . The  `strict`  value is required to have null-safety taken in account in Kotlin types inferred from Spring API but should be used with the knowledge that Spring API nullability declaration could evolve even between minor releases and more checks may be added in the future).

> Generic type arguments, varargs and array elements nullability are not yet supported. See [SPR-15942](https://jira.spring.io/browse/SPR-15942) for up-to-date information. Also be aware that Spring Boot’s own API is [not yet annotated](https://github.com/spring-projects/spring-boot/issues/10712).

## 50.3 Kotlin API

### 50.3.1 runApplication

Spring Boot provides an idiomatic way to run an application with  `runApplication<MyApplication>(*args)`  as shown in the following example:

```java
import org.springframework.boot.autoconfigure.SpringBootApplication
import org.springframework.boot.runApplication

@SpringBootApplication
class MyApplication

fun main(args: Array<String>) {
	runApplication<MyApplication>(*args)
}
```

This is a drop-in replacement for  `SpringApplication.run(MyApplication::class.java, *args)` . It also allows customization of the application as shown in the following example:

```java
runApplication<MyApplication>(*args) {
	setBannerMode(OFF)
}
```

### 50.3.2 Extensions

Kotlin [extensions](https://kotlinlang.org/docs/reference/extensions.html) provide the ability to extend existing classes with additional functionality. The Spring Boot Kotlin API makes use of these extensions to add new Kotlin specific conveniences to existing APIs.

`TestRestTemplate`  extensions, similar to those provided by Spring Framework for  `RestOperations`  in Spring Framework, are provided. Among other things, the extensions make it possible to take advantage of Kotlin reified type parameters.

## 50.4 Dependency management

In order to avoid mixing different version of Kotlin dependencies on the classpath, dependency management of the following Kotlin dependencies is provided:

-  `kotlin-reflect` 

-  `kotlin-runtime` 

-  `kotlin-stdlib` 

-  `kotlin-stdlib-jdk7` 

-  `kotlin-stdlib-jdk8` 

-  `kotlin-stdlib-jre7` 

-  `kotlin-stdlib-jre8` 

With Maven, the Kotlin version can be customized via the  `kotlin.version`  property and plugin management is provided for  `kotlin-maven-plugin` . With Gradle, the Spring Boot plugin automatically aligns the  `kotlin.version`  with the version of the Kotlin plugin.

## 50.5 @ConfigurationProperties

`@ConfigurationProperties`  currently only works with  `lateinit`  or nullable  `var`  properties (the former is recommended), since immutable classes initialized by constructors are [not yet supported](https://github.com/spring-projects/spring-boot/issues/8762).

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

> To generate [your own metadata](configuration-metadata.html#configuration-metadata-annotation-processor) using the annotation processor, [kapt should be configured](https://kotlinlang.org/docs/reference/kapt.html) with the  `spring-boot-configuration-processor`  dependency.

## 50.6 Testing

While it is possible to use JUnit 4 (the default provided by  `spring-boot-starter-test` ) to test Kotlin code, JUnit 5 is recommended. JUnit 5 enables a test class to be instantiated once and reused for all of the class’s tests. This makes it possible to use  `@BeforeAll`  and  `@AfterAll`  annotations on non-static methods, which is a good fit for Kotlin.

To use JUnit 5, exclude  `junit:junit`  dependency from  `spring-boot-starter-test` , add JUnit 5 dependencies, and configure the Maven or Gradle plugin accordingly. See the [JUnit 5 documentation](https://junit.org/junit5/docs/current/user-guide/#dependency-metadata-junit-jupiter-samples) for more details. You also need to [switch test instance lifecycle to "per-class"](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle-changing-default).

## 50.7 Resources

### 50.7.1 Further reading

- [Kotlin language reference](https://kotlinlang.org/docs/reference/)

- [Kotlin Slack](http://slack.kotlinlang.org/) (with a dedicated #spring channel)

- [Stackoverflow with spring and kotlin tags](https://stackoverflow.com/questions/tagged/spring+kotlin)

- [Try Kotlin in your browser](https://try.kotlinlang.org/)

- [Kotlin blog](https://blog.jetbrains.com/kotlin/)

- [Awesome Kotlin](https://kotlin.link/)

- [Tutorial: building web applications with Spring Boot and Kotlin](https://spring.io/guides/tutorials/spring-boot-kotlin/)

- [Developing Spring Boot applications with Kotlin](https://spring.io/blog/2016/02/15/developing-spring-boot-applications-with-kotlin)

- [A Geospatial Messenger with Kotlin, Spring Boot and PostgreSQL](https://spring.io/blog/2016/03/20/a-geospatial-messenger-with-kotlin-spring-boot-and-postgresql)

- [Introducing Kotlin support in Spring Framework 5.0](https://spring.io/blog/2017/01/04/introducing-kotlin-support-in-spring-framework-5-0)

- [Spring Framework 5 Kotlin APIs, the functional way](https://spring.io/blog/2017/08/01/spring-framework-5-kotlin-apis-the-functional-way)

### 50.7.2 Examples

- [spring-boot-kotlin-demo](https://github.com/sdeleuze/spring-boot-kotlin-demo): regular Spring Boot + Spring Data JPA project

- [mixit](https://github.com/mixitconf/mixit): Spring Boot 2 + WebFlux + Reactive Spring Data MongoDB

- [spring-kotlin-fullstack](https://github.com/sdeleuze/spring-kotlin-fullstack): WebFlux Kotlin fullstack example with Kotlin2js for frontend instead of JavaScript or TypeScript

- [spring-petclinic-kotlin](https://github.com/spring-petclinic/spring-petclinic-kotlin): Kotlin version of the Spring PetClinic Sample Application

- [spring-kotlin-deepdive](https://github.com/sdeleuze/spring-kotlin-deepdive): a step by step migration for Boot 1.0 + Java to Boot 2.0 + Kotlin


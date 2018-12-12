## 123. Introduction

Spring Cloud Function is a project with the following high-level goals:

- Promote the implementation of business logic via functions.

- Decouple the development lifecycle of business logic from any specific runtime target so that the same code can run as a web endpoint, a stream processor, or a task.

- Support a uniform programming model across serverless providers, as well as the ability to run standalone (locally or in a PaaS).

- Enable Spring Boot features (auto-configuration, dependency injection, metrics) on serverless providers.

It abstracts away all of the transport details and infrastructure, allowing the developer to keep all the familiar tools and processes, and focus firmly on business logic.

Here’s a complete, executable, testable Spring Boot application (implementing a simple string manipulation):

```java
@SpringBootApplication
public class Application {

@Bean
public Function<Flux<String>, Flux<String>> uppercase() {
return flux -> flux.map(value -> value.toUpperCase());
}

public static void main(String[] args) {
SpringApplication.run(Application.class, args);
}
}
```

It’s just a Spring Boot application, so it can be built, run and tested, locally and in a CI build, the same way as any other Spring Boot application. The  `Function`  is from  `java.util`  and  `Flux`  is a [Reactive Streams](http://www.reactive-streams.org/)  `Publisher`  from [Project Reactor](https://projectreactor.io/). The function can be accessed over HTTP or messaging.

Spring Cloud Function has 4 main features:

Wrappers for @Beans of type Function, Consumer and Supplier, exposing them to the outside world as either HTTP endpoints and/or message stream listeners/publishers with RabbitMQ, Kafka etc. Compiling strings which are Java function bodies into bytecode, and then turning them into @Beans that can be wrapped as above. Deploying a JAR file containing such an application context with an isolated classloader, so that you can pack them together in a single JVM. Adapters for AWS Lambda, Azure, Apache OpenWhisk and possibly other "serverless" service providers.

> Spring Cloud is released under the non-restrictive Apache 2.0 license. If you would like to contribute to this section of the documentation or if you find an error, please find the source code and issue trackers in the project at [github](https://github.com/spring-cloud/spring-cloud-function/tree/master/docs/src/main/asciidoc).


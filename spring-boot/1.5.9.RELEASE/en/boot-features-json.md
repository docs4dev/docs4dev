## 27. JSON

Spring Boot provides integration with three JSON mapping libraries:

- Gson

- Jackson

- JSON-B

Jackson is the preferred and default library.

## 27.1 Jackson

Auto-configuration for Jackson is provided and Jackson is part of  `spring-boot-starter-json` . When Jackson is on the classpath an  `ObjectMapper`  bean is automatically configured. Several configuration properties are provided for [customizing the configuration of the ObjectMapper](howto-spring-mvc.html#howto-customize-the-jackson-objectmapper).

## 27.2 Gson

Auto-configuration for Gson is provided. When Gson is on the classpath a  `Gson`  bean is automatically configured. Several  `spring.gson.*`  configuration properties are provided for customizing the configuration. To take more control, one or more  `GsonBuilderCustomizer`  beans can be used.

## 27.3 JSON-B

Auto-configuration for JSON-B is provided. When the JSON-B API and an implementation are on the classpath a  `Jsonb`  bean will be automatically configured. The preferred JSON-B implementation is Apache Johnzon for which dependency management is provided.


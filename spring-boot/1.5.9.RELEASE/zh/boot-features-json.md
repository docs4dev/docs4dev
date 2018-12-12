## 27. JSON

Spring Boot提供了与三个JSON映射库的集成：

- Gson

- Jackson

- JSON-B

Jackson是首选和默认的图书馆.

## 27.1Jackson

提供Jackson的自动配置，Jackson是 `spring-boot-starter-json` 的一部分.当Jackson在类路径上时，会自动配置 `ObjectMapper`  bean.为[customizing the configuration of the ObjectMapper](howto-spring-mvc.html#howto-customize-the-jackson-objectmapper)提供了几个配置属性.

## 27.2 Gson

提供Gson的自动配置.当Gson在类路径上时，会自动配置 `Gson`  bean.提供了几个 `spring.gson.*` 配置属性来自定义配置.为了获得更多控制，可以使用一个或多个 `GsonBuilderCustomizer`  bean.

## 27.3 JSON-B

提供了JSON-B的自动配置.当JSON-B API和实现在类路径上时，将自动配置 `Jsonb`  bean.首选的JSON-B实现是Apache Johnzon，它提供了依赖关系管理.


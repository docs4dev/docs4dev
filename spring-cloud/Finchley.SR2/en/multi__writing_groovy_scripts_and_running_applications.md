## 81. Writing Groovy Scripts and Running Applications

Spring Cloud CLI has support for most of the Spring Cloud declarative features, such as the  `@Enable*`  class of annotations. For example, here is a fully functional Eureka server

**app.groovy.**  

```java
@EnableEurekaServer
class Eureka {}
```

which you can run from the command line like this

```java
$ spring run app.groovy
```

To include additional dependencies, often it suffices just to add the appropriate feature-enabling annotation, e.g.  `@EnableConfigServer` ,  `@EnableOAuth2Sso`  or  `@EnableEurekaClient` . To manually include a dependency you can use a  `@Grab`  with the special "Spring Boot" short style artifact co-ordinates, i.e. with just the artifact ID (no need for group or version information), e.g. to set up a client app to listen on AMQP for management events from the Spring CLoud Bus:

**app.groovy.**  

```java
@Grab('spring-cloud-starter-bus-amqp')
@RestController
class Service {
@RequestMapping('/')
def home() { [message: 'Hello'] }
}
```


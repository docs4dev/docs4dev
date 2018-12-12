## 25. Profiles

Spring Profiles provide a way to segregate parts of your application configuration and make it be available only in certain environments. Any  `@Component`  or  `@Configuration`  can be marked with  `@Profile`  to limit when it is loaded, as shown in the following example:

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {

	// ...

}
```

You can use a  `spring.profiles.active`   `Environment`  property to specify which profiles are active. You can specify the property in any of the ways described earlier in this chapter. For example, you could include it in your  `application.properties` , as shown in the following example:

```java
spring.profiles.active=dev,hsqldb
```

You could also specify it on the command line by using the following switch:  `--spring.profiles.active=dev,hsqldb` .

## 25.1 Adding Active Profiles

The  `spring.profiles.active`  property follows the same ordering rules as other properties: The highest  `PropertySource`  wins. This means that you can specify active profiles in  `application.properties`  and then  **replace**  them by using the command line switch.

Sometimes, it is useful to have profile-specific properties that  **add**  to the active profiles rather than replace them. The  `spring.profiles.include`  property can be used to unconditionally add active profiles. The  `SpringApplication`  entry point also has a Java API for setting additional profiles (that is, on top of those activated by the  `spring.profiles.active`  property). See the  `setAdditionalProfiles()`  method in [SpringApplication](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/SpringApplication.html).

For example, when an application with the following properties is run by using the switch,  `--spring.profiles.active=prod` , the  `proddb`  and  `prodmq`  profiles are also activated:

```java
---
my.property: fromyamlfile
---
spring.profiles: prod
spring.profiles.include:
- proddb
- prodmq
```

> Remember that the  `spring.profiles`  property can be defined in a YAML document to determine when this particular document is included in the configuration. See [Section 77.7, “Change Configuration Depending on the Environment”](howto-properties-and-configuration.html#howto-change-configuration-depending-on-the-environment) for more details.

## 25.2 Programmatically Setting Profiles

You can programmatically set active profiles by calling  `SpringApplication.setAdditionalProfiles(…)`  before your application runs. It is also possible to activate profiles by using Spring’s  `ConfigurableEnvironment`  interface.

## 25.3 Profile-specific Configuration Files

Profile-specific variants of both  `application.properties`  (or  `application.yml` ) and files referenced through  `@ConfigurationProperties`  are considered as files and loaded. See "[Section 24.4, “Profile-specific Properties”](boot-features-external-config.html#boot-features-external-config-profile-specific-properties)" for details.


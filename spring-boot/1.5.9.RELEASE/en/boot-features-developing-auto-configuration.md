## 49. Creating Your Own Auto-configuration

If you work in a company that develops shared libraries, or if you work on an open-source or commercial library, you might want to develop your own auto-configuration. Auto-configuration classes can be bundled in external jars and still be picked-up by Spring Boot.

Auto-configuration can be associated to a “starter” that provides the auto-configuration code as well as the typical libraries that you would use with it. We first cover what you need to know to build your own auto-configuration and then we move on to the [typical steps required to create a custom starter](boot-features-developing-auto-configuration.html#boot-features-custom-starter).

> A [demo project](https://github.com/snicoll-demos/spring-boot-master-auto-configuration) is available to showcase how you can create a starter step-by-step.

## 49.1 Understanding Auto-configured Beans

Under the hood, auto-configuration is implemented with standard  `@Configuration`  classes. Additional  `@Conditional`  annotations are used to constrain when the auto-configuration should apply. Usually, auto-configuration classes use  `@ConditionalOnClass`  and  `@ConditionalOnMissingBean`  annotations. This ensures that auto-configuration applies only when relevant classes are found and when you have not declared your own  `@Configuration` .

You can browse the source code of [spring-boot-autoconfigure](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure) to see the  `@Configuration`  classes that Spring provides (see the [META-INF/spring.factories](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories) file).

## 49.2 Locating Auto-configuration Candidates

Spring Boot checks for the presence of a  `META-INF/spring.factories`  file within your published jar. The file should list your configuration classes under the  `EnableAutoConfiguration`  key, as shown in the following example:

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

You can use the [@AutoConfigureAfter](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.java) or [@AutoConfigureBefore](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.java) annotations if your configuration needs to be applied in a specific order. For example, if you provide web-specific configuration, your class may need to be applied after  `WebMvcAutoConfiguration` .

If you want to order certain auto-configurations that should not have any direct knowledge of each other, you can also use  `@AutoConfigureOrder` . That annotation has the same semantic as the regular  `@Order`  annotation but provides a dedicated order for auto-configuration classes.

> Auto-configurations must be loaded that way only. Make sure that they are defined in a specific package space and that, in particular, they are never the target of component scanning.

## 49.3 Condition Annotations

You almost always want to include one or more  `@Conditional`  annotations on your auto-configuration class. The  `@ConditionalOnMissingBean`  annotation is one common example that is used to allow developers to override auto-configuration if they are not happy with your defaults.

Spring Boot includes a number of  `@Conditional`  annotations that you can reuse in your own code by annotating  `@Configuration`  classes or individual  `@Bean`  methods. These annotations include:

- [Section 49.3.1, “Class Conditions”](boot-features-developing-auto-configuration.html#boot-features-class-conditions)

- [Section 49.3.2, “Bean Conditions”](boot-features-developing-auto-configuration.html#boot-features-bean-conditions)

- [Section 49.3.3, “Property Conditions”](boot-features-developing-auto-configuration.html#boot-features-property-conditions)

- [Section 49.3.4, “Resource Conditions”](boot-features-developing-auto-configuration.html#boot-features-resource-conditions)

- [Section 49.3.5, “Web Application Conditions”](boot-features-developing-auto-configuration.html#boot-features-web-application-conditions)

- [Section 49.3.6, “SpEL Expression Conditions”](boot-features-developing-auto-configuration.html#boot-features-spel-conditions)

### 49.3.1 Class Conditions

The  `@ConditionalOnClass`  and  `@ConditionalOnMissingClass`  annotations let configuration be included based on the presence or absence of specific classes. Due to the fact that annotation metadata is parsed by using [ASM](http://asm.ow2.org/), you can use the  `value`  attribute to refer to the real class, even though that class might not actually appear on the running application classpath. You can also use the  `name`  attribute if you prefer to specify the class name by using a  `String`  value.

> If you use  `@ConditionalOnClass`  or  `@ConditionalOnMissingClass`  as a part of a meta-annotation to compose your own composed annotations, you must use  `name`  as referring to the class in such a case is not handled.

### 49.3.2 Bean Conditions

The  `@ConditionalOnBean`  and  `@ConditionalOnMissingBean`  annotations let a bean be included based on the presence or absence of specific beans. You can use the  `value`  attribute to specify beans by type or  `name`  to specify beans by name. The  `search`  attribute lets you limit the  `ApplicationContext`  hierarchy that should be considered when searching for beans.

When placed on a  `@Bean`  method, the target type defaults to the return type of the method, as shown in the following example:

```java
@Configuration
public class MyAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public MyService myService() { ... }

}
```

In the preceding example, the  `myService`  bean is going to be created if no bean of type  `MyService`  is already contained in the  `ApplicationContext` .

> You need to be very careful about the order in which bean definitions are added, as these conditions are evaluated based on what has been processed so far. For this reason, we recommend using only  `@ConditionalOnBean`  and  `@ConditionalOnMissingBean`  annotations on auto-configuration classes (since these are guaranteed to load after any user-defined bean definitions have been added).

>  `@ConditionalOnBean`  and  `@ConditionalOnMissingBean`  do not prevent  `@Configuration`  classes from being created. The only difference between using these conditions at the class level and marking each contained  `@Bean`  method with the annotation is that the former prevents registration of the  `@Configuration`  class as a bean if the condition does not match.

### 49.3.3 Property Conditions

The  `@ConditionalOnProperty`  annotation lets configuration be included based on a Spring Environment property. Use the  `prefix`  and  `name`  attributes to specify the property that should be checked. By default, any property that exists and is not equal to  `false`  is matched. You can also create more advanced checks by using the  `havingValue`  and  `matchIfMissing`  attributes.

### 49.3.4 Resource Conditions

The  `@ConditionalOnResource`  annotation lets configuration be included only when a specific resource is present. Resources can be specified by using the usual Spring conventions, as shown in the following example:  `file:/home/user/test.dat` .

### 49.3.5 Web Application Conditions

The  `@ConditionalOnWebApplication`  and  `@ConditionalOnNotWebApplication`  annotations let configuration be included depending on whether the application is a “web application”. A web application is any application that uses a Spring  `WebApplicationContext` , defines a  `session`  scope, or has a  `StandardServletEnvironment` .

### 49.3.6 SpEL Expression Conditions

The  `@ConditionalOnExpression`  annotation lets configuration be included based on the result of a [SpEL expression](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#expressions).

## 49.4 Testing your Auto-configuration

An auto-configuration can be affected by many factors: user configuration ( `@Bean`  definition and  `Environment`  customization), condition evaluation (presence of a particular library), and others. Concretely, each test should create a well defined  `ApplicationContext`  that represents a combination of those customizations.  `ApplicationContextRunner`  provides a great way to achieve that.

`ApplicationContextRunner`  is usually defined as a field of the test class to gather the base, common configuration. The following example makes sure that  `UserServiceAutoConfiguration`  is always invoked:

```java
private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
		.withConfiguration(AutoConfigurations.of(UserServiceAutoConfiguration.class));
```

> If multiple auto-configurations have to be defined, there is no need to order their declarations as they are invoked in the exact same order as when running the application.

Each test can use the runner to represent a particular use case. For instance, the sample below invokes a user configuration ( `UserConfiguration` ) and checks that the auto-configuration backs off properly. Invoking  `run`  provides a callback context that can be used with  `Assert4J` .

```java
@Test
public void defaultServiceBacksOff() {
	this.contextRunner.withUserConfiguration(UserConfiguration.class)
			.run((context) -> {
				assertThat(context).hasSingleBean(UserService.class);
				assertThat(context.getBean(UserService.class)).isSameAs(
						context.getBean(UserConfiguration.class).myUserService());
			});
}

@Configuration
static class UserConfiguration {

	@Bean
	public UserService myUserService() {
		return new UserService("mine");
	}

}
```

It is also possible to easily customize the  `Environment` , as shown in the following example:

```java
@Test
public void serviceNameCanBeConfigured() {
	this.contextRunner.withPropertyValues("user.name=test123").run((context) -> {
		assertThat(context).hasSingleBean(UserService.class);
		assertThat(context.getBean(UserService.class).getName()).isEqualTo("test123");
	});
}
```

The runner can also be used to display the  `ConditionEvaluationReport` . The report can be printed at  `INFO`  or  `DEBUG`  level. The following example shows how to use the  `ConditionEvaluationReportLoggingListener`  to print the report in auto-configuration tests.

```java
@Test
public void autoConfigTest {
	ConditionEvaluationReportLoggingListener initializer = new ConditionEvaluationReportLoggingListener(
			LogLevel.INFO);
	ApplicationContextRunner contextRunner = new ApplicationContextRunner()
			.withInitializer(initializer).run((context) -> {
					// Do something...
			});
}
```

### 49.4.1 Simulating a Web Context

If you need to test an auto-configuration that only operates in a Servlet or Reactive web application context, use the  `WebApplicationContextRunner`  or  `ReactiveWebApplicationContextRunner`  respectively.

### 49.4.2 Overriding the Classpath

It is also possible to test what happens when a particular class and/or package is not present at runtime. Spring Boot ships with a  `FilteredClassLoader`  that can easily be used by the runner. In the following example, we assert that if  `UserService`  is not present, the auto-configuration is properly disabled:

```java
@Test
public void serviceIsIgnoredIfLibraryIsNotPresent() {
	this.contextRunner.withClassLoader(new FilteredClassLoader(UserService.class))
			.run((context) -> assertThat(context).doesNotHaveBean("userService"));
}
```

## 49.5 Creating Your Own Starter

A full Spring Boot starter for a library may contain the following components:

- The  `autoconfigure`  module that contains the auto-configuration code.

- The  `starter`  module that provides a dependency to the  `autoconfigure`  module as well as the library and any additional dependencies that are typically useful. In a nutshell, adding the starter should provide everything needed to start using that library.

> You may combine the auto-configuration code and the dependency management in a single module if you do not need to separate those two concerns.

### 49.5.1 Naming

You should make sure to provide a proper namespace for your starter. Do not start your module names with  `spring-boot` , even if you use a different Maven  `groupId` . We may offer official support for the thing you auto-configure in the future.

As a rule of thumb, you should name a combined module after the starter. For example, assume that you are creating a starter for "acme" and that you name the auto-configure module  `acme-spring-boot-autoconfigure`  and the starter  `acme-spring-boot-starter` . If you only have one module that combines the two, name it  `acme-spring-boot-starter` .

Also, if your starter provides configuration keys, use a unique namespace for them. In particular, do not include your keys in the namespaces that Spring Boot uses (such as  `server` ,  `management` ,  `spring` , and so on). If you use the same namespace, we may modify these namespaces in the future in ways that break your modules.

Make sure to [trigger meta-data generation](configuration-metadata.html#configuration-metadata-annotation-processor) so that IDE assistance is available for your keys as well. You may want to review the generated meta-data ( `META-INF/spring-configuration-metadata.json` ) to make sure your keys are properly documented.

### 49.5.2 autoconfigure Module

The  `autoconfigure`  module contains everything that is necessary to get started with the library. It may also contain configuration key definitions (such as  `@ConfigurationProperties` ) and any callback interface that can be used to further customize how the components are initialized.

> You should mark the dependencies to the library as optional so that you can include the  `autoconfigure`  module in your projects more easily. If you do it that way, the library is not provided and, by default, Spring Boot backs off.

Spring Boot uses an annotation processor to collect the conditions on auto-configurations in a metadata file ( `META-INF/spring-autoconfigure-metadata.properties` ). If that file is present, it is used to eagerly filter auto-configurations that do not match, which will improve startup time. It is recommended to add the following dependency in a module that contains auto-configurations:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-autoconfigure-processor</artifactId>
	<optional>true</optional>
</dependency>
```

With Gradle 4.5 and earlier, the dependency should be declared in the  `compileOnly`  configuration, as shown in the following example:

```java
dependencies {
	compileOnly "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

With Gradle 4.6 and later, the dependency should be declared in the  `annotationProcessor`  configuration, as shown in the following example:

```java
dependencies {
	annotationProcessor "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

### 49.5.3 Starter Module

The starter is really an empty jar. Its only purpose is to provide the necessary dependencies to work with the library. You can think of it as an opinionated view of what is required to get started.

Do not make assumptions about the project in which your starter is added. If the library you are auto-configuring typically requires other starters, mention them as well. Providing a proper set of default dependencies may be hard if the number of optional dependencies is high, as you should avoid including dependencies that are unnecessary for a typical usage of the library. In other words, you should not include optional dependencies.

> Either way, your starter must reference the core Spring Boot starter ( `spring-boot-starter` ) directly or indirectly (i.e. no need to add it if your starter relies on another starter). If a project is created with only your custom starter, Spring Boot’s core features will be honoured by the presence of the core starter.


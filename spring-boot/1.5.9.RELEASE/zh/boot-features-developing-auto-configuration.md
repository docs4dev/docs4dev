## 49.创建自己的自动配置

如果您在开发共享库的公司工作，或者您在开源或商业库中工作，则可能需要开发自己的自动配置.自动配置类可以捆绑在外部jar中，仍然可以通过Spring Boot获取.

自动配置可以与“启动器”相关联，该“启动器”提供自动配置代码以及您将使用它的典型库.我们首先介绍了构建自己的自动配置需要了解的内容，然后我们继续讨论[typical steps required to create a custom starter](boot-features-developing-auto-configuration.html#boot-features-custom-starter).

> A [demo project](https://github.com/snicoll-demos/spring-boot-master-auto-configuration)可用于展示如何逐步创建启动器.

## 49.1了解自动配置的Bean

在引擎盖下，使用标准 `@Configuration` 类实现自动配置.附加 `@Conditional` 注释用于约束何时应用自动配置.通常，自动配置类使用 `@ConditionalOnClass` 和 `@ConditionalOnMissingBean` 注释.这可确保仅在找到相关类时以及未声明自己的 `@Configuration` 时才应用自动配置.

您可以浏览[spring-boot-autoconfigure](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure)的源代码以查看Spring提供的 `@Configuration` 类（请参阅[META-INF/spring.factories](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/resources/META-INF/spring.factories)文件）.

## 49.2找到自动配置候选者

Spring Boot会检查已发布jar中是否存在 `META-INF/spring.factories` 文件.该文件应列出 `EnableAutoConfiguration` 键下的配置类，如以下示例所示：

```java
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.mycorp.libx.autoconfigure.LibXAutoConfiguration,\
com.mycorp.libx.autoconfigure.LibXWebAutoConfiguration
```

如果需要按特定顺序应用配置，则可以使用[@AutoConfigureAfter](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureAfter.java)或[@AutoConfigureBefore](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/AutoConfigureBefore.java)注释.例如，如果您提供特定于Web的配置，则可能需要在 `WebMvcAutoConfiguration` 之后应用您的类.

如果您想订购某些不应该彼此直接了解的自动配置，您也可以使用 `@AutoConfigureOrder` .该注释与常规 `@Order` 具有相同的语义注释，但为自动配置类提供专用顺序.

> Auto-configurations必须仅以这种方式加载.确保它们是在特定的包空间中定义的，特别是它们永远不是组件扫描的目标.

## 49.3条件注释

您几乎总是希望在自动配置类中包含一个或多个 `@Conditional` 注释.  `@ConditionalOnMissingBean` 注释是一个常见示例，用于允许开发人员如果对您的默认值不满意，则覆盖自动配置.

Spring Boot包含许多 `@Conditional` 注释，您可以通过注释 `@Configuration` 类或单独的 `@Bean` 方法在您自己的代码中重用它们.这些注释包括：

- [Section 49.3.1, “Class Conditions”](boot-features-developing-auto-configuration.html#boot-features-class-conditions)

- [Section 49.3.2, “Bean Conditions”](boot-features-developing-auto-configuration.html#boot-features-bean-conditions)

- [Section 49.3.3, “Property Conditions”](boot-features-developing-auto-configuration.html#boot-features-property-conditions)

- [Section 49.3.4, “Resource Conditions”](boot-features-developing-auto-configuration.html#boot-features-resource-conditions)

- [Section 49.3.5, “Web Application Conditions”](boot-features-developing-auto-configuration.html#boot-features-web-application-conditions)

- [Section 49.3.6, “SpEL Expression Conditions”](boot-features-developing-auto-configuration.html#boot-features-spel-conditions)

### 49.3.1类条件

`@ConditionalOnClass` 和 `@ConditionalOnMissingClass` 注释允许根据特定类的存在与否来配置.由于使用[ASM](http://asm.ow2.org/)解析了注释元数据这一事实，您可以使用 `value` 属性来引用真实类，即使该类实际上可能不会出现在正在运行的应用程序类路径中.如果您希望使用 `String` 值指定类名，也可以使用 `name` 属性.

> 如果您使用 `@ConditionalOnClass` 或 `@ConditionalOnMissingClass` 作为元注释的一部分来编写自己的组合注释，则必须使用 `name` 作为引用类，在这种情况下不处理.

### 49.3.2 Bean条件

`@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 注释允许根据特定bean的存在与否来包含bean.您可以使用 `value` 属性按类型指定bean，或使用 `name` 按名称指定bean.  `search` 属性允许您限制搜索bean时应考虑的 `ApplicationContext` 层次结构.

放置在 `@Bean` 方法上时，目标类型默认为方法的返回类型，如以下示例所示：

```java
@Configuration
public class MyAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public MyService myService() { ... }

}
```

在前面的示例中，如果 `ApplicationContext` 中不包含 `MyService` 类型的bean，则将创建 `myService`  bean.

> 您需要非常小心添加bean定义的顺序，因为这些条件是根据到目前为止已处理的内容进行评估的.因此，我们建议在自动配置类上仅使用 `@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 注释（因为这些注释保证在添加任何用户定义的bean定义后加载）.

>  `@ConditionalOnBean` 和 `@ConditionalOnMissingBean` 不会阻止创建 `@Configuration` 类.在类级别使用这些条件和使用注释标记每个包含 `@Bean` 方法的唯一区别是，如果条件不匹配，前者会阻止将 `@Configuration` 类注册为bean.

### 49.3.3properties条件

`@ConditionalOnProperty` 注释允许基于Spring Environment属性包含配置.使用 `prefix` 和 `name` 属性指定应检查的属性.默认情况下，匹配存在且不等于 `false` 的任何属性.您还可以使用 `havingValue` 和 `matchIfMissing` 属性创建更高级的检查.

### 49.3.4资源条件

`@ConditionalOnResource` 注释仅允许在存在特定资源时包含配置.可以使用常用的Spring约定来指定资源，如以下示例所示： `file:/home/user/test.dat` .

### 49.3.5网络应用条件

`@ConditionalOnWebApplication` 和 `@ConditionalOnNotWebApplication` 注释允许配置，具体取决于应用程序是否为“Web应用程序”. Web应用程序是使用Spring  `WebApplicationContext` ，定义 `session` 范围或具有 `StandardServletEnvironment` 的任何应用程序.

### 49.3.6 SpEL表达条件

`@ConditionalOnExpression` 注释允许根据[SpEL expression](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#expressions)的结果包含配置.

## 49.4测试自动配置

自动配置可能受许多因素的影响：用户配置（ `@Bean` 定义和 `Environment` 自定义），条件评估（存在特定库）等.具体而言，每个测试都应创建一个定义良好的 `ApplicationContext` ，它代表这些自定义的组合.  `ApplicationContextRunner` 提供了实现这一目标的好方法.

`ApplicationContextRunner` 通常被定义为测试类的一个字段，用于收集基本的通用配置.以下示例确保始终调用 `UserServiceAutoConfiguration` ：

```java
private final ApplicationContextRunner contextRunner = new ApplicationContextRunner()
		.withConfiguration(AutoConfigurations.of(UserServiceAutoConfiguration.class));
```

> 如果必须定义多个自动配置，则无需按照与运行应用程序时完全相同的顺序调用它们的声明.

每个测试都可以使用运行器来表示特定的用例.例如，下面的示例调用用户配置（ `UserConfiguration` ）并检查自动配置是否正确退回.调用 `run` 提供了一个可以与 `Assert4J` 一起使用的回调上下文.

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

也可以轻松自定义 `Environment` ，如以下示例所示：

```java
@Test
public void serviceNameCanBeConfigured() {
	this.contextRunner.withPropertyValues("user.name=test123").run((context) -> {
		assertThat(context).hasSingleBean(UserService.class);
		assertThat(context.getBean(UserService.class).getName()).isEqualTo("test123");
	});
}
```

跑步者也可以用来显示 `ConditionEvaluationReport` .报告可以在 `INFO` 或 `DEBUG` 级别打印.以下示例显示了如何使用 `ConditionEvaluationReportLoggingListener` 在自动配置测试中打印报告.

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

### 49.4.1模拟Web上下文

如果需要测试仅在Servlet或Reactive Web应用程序上下文中运行的自动配置，请分别使用 `WebApplicationContextRunner` 或 `ReactiveWebApplicationContextRunner` .

### 49.4.2重写类路径

还可以测试在运行时不存在特定类和/或包时发生的情况. Spring Boot随附 `FilteredClassLoader` ，跑步者可以轻松使用.在以下示例中，我们声明如果 `UserService` 不存在，则会自动禁用自动配置：

```java
@Test
public void serviceIsIgnoredIfLibraryIsNotPresent() {
	this.contextRunner.withClassLoader(new FilteredClassLoader(UserService.class))
			.run((context) -> assertThat(context).doesNotHaveBean("userService"));
}
```

## 49.5创建自己的初学者

库的完整Spring Boot启动程序可能包含以下组件：

- 包含自动配置代码的 `autoconfigure` 模块.

-   `starter` 模块，它提供对 `autoconfigure` 模块以及库的依赖关系以及通常有用的任何其他依赖项.简而言之，添加启动器应该提供开始使用该库所需的一切.

> 如果您不需要将这两个问题分开，您可以将自动配置代码和依赖关系管理合并到一个模块中.

### 49.5.1命名

您应该确保为您的启动器提供适当的命名空间.即使您使用不同的Maven  `groupId` ，也不要使用 `spring-boot` 启动模块名称.我们可能会为您将来自动配置的内容提供官方支持.

根据经验，您应该在启动后命名组合模块.例如，假设您正在为"acme"创建启动器，并且您将自动配置模块 `acme-spring-boot-autoconfigure` 和启动器 `acme-spring-boot-starter` 命名为.如果您只有一个组合了两个的模块，请将其命名为 `acme-spring-boot-starter` .

此外，如果您的启动器提供配置密钥，请为它们使用唯一的命名空间.特别是，不要将您的密钥包含在Spring Boot使用的命名空间中（例如 `server` ， `management` ， `spring` 等）.如果您使用相同的命名空间，我们将来可能会以破坏您的模块的方式修改这些命名空间.

确保[trigger meta-data generation](configuration-metadata.html#configuration-metadata-annotation-processor)也可以为您的密钥提供IDE帮助.您可能希望查看生成的元数据（ `META-INF/spring-configuration-metadata.json` ）以确保正确记录您的密钥.

### 49.5.2 autoconfigure模块

`autoconfigure` 模块包含开始使用库所需的所有内容.它还可以包含配置键定义（例如 `@ConfigurationProperties` ）和任何可用于进一步自定义组件初始化方式的回调接口.

> 您应该将库的依赖项标记为可选，以便您可以更轻松地在项目中包含 `autoconfigure` 模块.如果以这种方式执行，则不提供库，默认情况下，Spring Boot会退出.

Spring Boot使用注释处理器来收集元数据文件（ `META-INF/spring-autoconfigure-metadata.properties` ）中自动配置的条件.如果该文件存在，则用于热切过滤不匹配的自动配置，这将缩短启动时间.建议在包含自动配置的模块中添加以下依赖项：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-autoconfigure-processor</artifactId>
	<optional>true</optional>
</dependency>
```

使用Gradle 4.5及更早版本时，应在 `compileOnly` 配置中声明依赖项，如以下示例所示：

```java
dependencies {
	compileOnly "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

使用Gradle 4.6及更高版本时，应在 `annotationProcessor` 配置中声明依赖项，如以下示例所示：

```java
dependencies {
	annotationProcessor "org.springframework.boot:spring-boot-autoconfigure-processor"
}
```

### 49.5.3入门模块

起动器真的是一个空jar.它的唯一目的是提供必要的依赖项来使用库.您可以将其视为对入门所需内容的一种看法.

不要对添加启动器的项目做出假设.如果您自动配置的库通常需要其他启动器，请同时提及它们.如果可选依赖项的数量很高，则提供一组适当的默认依赖项可能很难，因为您应该避免包含对典型库的使用不必要的依赖项.换句话说，您不应该包含可选的依赖项.

> 无论哪种方式，您的启动器必须直接或间接引用核心Spring Boot启动器（ `spring-boot-starter` ）（即如果您的启动器依赖于另一个启动器，则无需添加它）.如果只使用自定义启动器创建项目，则Spring Boot的核心功能将通过核心启动器的存在来实现.


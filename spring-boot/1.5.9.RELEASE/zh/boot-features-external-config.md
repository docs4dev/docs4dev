## 24.外部化配置

Spring Boot允许您外部化配置，以便您可以在不同的环境中使用相同的应用程序代码.您可以使用属性文件，YAML文件，环境变量和命令行参数来外部化配置.可以使用 `@Value` 注释直接将属性值注入到bean中，通过Spring的 `Environment` 抽象访问，或者[bound to structured objects](boot-features-external-config.html#boot-features-external-config-typesafe-configuration-properties)到 `@ConfigurationProperties` .

Spring Boot使用一个非常特殊的 `PropertySource` 命令，旨在允许合理地覆盖值.按以下顺序考虑属性：

在您的主目录上开发全局设置属性（当devtools处于活动状态时，〜/ .spring-boot-devtools.properties）.测试中的@TestPropertySource注释.测试中的属性属性.可在@SpringBootTest上使用，以及用于测试应用程序特定片段的测试注释.命令行参数. SPRING_APPLICATION_JSON中的属性（嵌入在环境变量或系统属性中的内联JSON）. ServletConfig初始化参数. ServletContext init参数.来自java：comp / env的JNDI属性.Java系统属性（System.getProperties（））. OS环境变量. RandomValuePropertySource，只具有随机属性.*.特定于配置文件的应用程序属性在打包的jar之外（application- {profile} .properties和YAML变体）.打包在jar中的特定于配置文件的应用程序属性（application- {profile} .properties和YAML变体）.打包jar之外的应用程序属性（application.properties和YAML变体）.打包在jar中的应用程序属性（application.properties和YAML变体）. @Configuration类上的@PropertySource注释.默认属性（通过设置SpringApplication.setDefaultProperties指定）.

要提供一个具体示例，假设您开发了一个使用 `name` 属性的 `@Component` ，如以下示例所示：

```java
import org.springframework.stereotype.*;
import org.springframework.beans.factory.annotation.*;

@Component
public class MyBean {

@Value("${name}")
private String name;

// ...

}
```

在应用程序类路径上（例如，在jar中），您可以拥有 `application.properties` 文件，为 `name` 提供合理的默认属性值.在新环境中运行时，可以在jar外部提供覆盖 `name` 的 `application.properties` 文件.对于一次性测试，您可以使用特定的命令行开关启动（例如， `java -jar app.jar --name="Spring"` ）.

> 可以在命令行上使用环境变量提供 `SPRING_APPLICATION_JSON` 属性.例如，您可以在UN * X shell中使用以下行：

## 24.1配置随机值

`RandomValuePropertySource` 对于注入随机值非常有用（例如，注入秘密或测试用例）.它可以生成整数，长整数，uuids或字符串，如以下示例所示：

```java
my.secret=${random.value}
my.number=${random.int}
my.bignumber=${random.long}
my.uuid=${random.uuid}
my.number.less.than.ten=${random.int(10)}
my.number.in.range=${random.int[1024,65536]}
```

`random.int*` 语法是 `OPEN value (,max) CLOSE` ，其中 `OPEN,CLOSE` 是任何字符， `value,max` 是整数.如果提供 `max` ，则 `value` 是最小值， `max` 是最大值（不包括）.

## 24.2访问命令行属性

默认情况下， `SpringApplication` 将任何命令行选项参数（即以 `--` 开头的参数，例如 `--server.port=9000` ）转换为 `property` ，并将它们添加到Spring  `Environment` .如前所述，命令行属性始终优先于其他属性源.

如果您不希望将命令行属性添加到 `Environment` ，则可以使用 `SpringApplication.setAddCommandLineProperties(false)` 禁用它们.

## 24.3应用程序属性文件

`SpringApplication` 从以下位置的 `application.properties` 文件加载属性并将它们添加到Spring  `Environment` ：

当前目录的A / config子目录当前目录classpath / config包类路径根目录

列表按优先级排序（在列表中较高位置定义的属性将覆盖在较低位置中定义的属性）.

> 您也可以[use YAML ('.yml') files](boot-features-external-config.html#boot-features-external-config-yaml)替代'.properties'.

如果您不喜欢 `application.properties` 作为配置文件名，则可以通过指定 `spring.config.name` 环境属性切换到另一个文件名.您还可以使用 `spring.config.location` 环境属性（以逗号分隔的目录位置或文件路径列表）来引用显式位置.以下示例显示如何指定其他文件名：

```java
$ java -jar myproject.jar --spring.config.name=myproject
```

以下示例显示如何指定两个位置：

```java
$ java -jar myproject.jar --spring.config.location=classpath:/default.properties,classpath:/override.properties
```

很早就会使用
>  `spring.config.name` 和 `spring.config.location` 来确定必须加载哪些文件，因此必须将它们定义为环境属性（通常是OS环境变量，系统属性或命令行参数）.

如果 `spring.config.location` 包含目录（而不是文件），则它们应以 `/` 结尾（并且在运行时，在加载之前附加从 `spring.config.name` 生成的名称，包括特定于配置文件的文件名）.  `spring.config.location` 中指定的文件按原样使用，不支持特定于配置文件的变体，并且被任何特定于配置文件的属性覆盖.

以相反的顺序搜索配置位置.默认情况下，配置的位置为 `classpath:/,classpath:/config/,file:./,file:./config/` .生成的搜索顺序如下：

file：./ config / file：./ classpath：/ config / classpath：/

使用 `spring.config.location` 配置自定义配置位置时，它们会替换默认位置.例如，如果 `spring.config.location` 配置了值 `classpath:/custom-config/,file:./custom-config/` ，则搜索顺序将变为：

file：./ custom-config / classpath：custom-config /

或者，当使用 `spring.config.additional-location` 配置自定义配置位置时，除默认位置外，还会使用它们.在默认位置之前搜索其他位置.例如，如果配置了 `classpath:/custom-config/,file:./custom-config/` 的其他位置，则搜索顺序将变为：

file：./ custom-config / classpath：custom-config / file：./ config / file：./ classpath：/ config / classpath：/

此搜索顺序允许您在一个配置文件中指定默认值，然后有选择地覆盖另一个配置文件中的值.您可以在 `application.properties` （或您使用 `spring.config.name` 选择的任何其他基本名称）中的某个默认位置为应用程序提供默认值.然后，可以在运行时使用位于其中一个自定义位置的不同文件覆盖这些默认值.

> 如果您使用的是环境变量而不是系统属性操作系统不允许使用句点分隔的键名，但您可以使用下划线（例如， `SPRING_CONFIG_NAME` 而不是 `spring.config.name` ）.

> 如果您的应用程序在容器中运行，则可以使用JNDI属性（在 `java:comp/env` 中）或servlet上下文初始化参数来代替环境变量或系统属性.

## 24.4特定于配置文件的属性

除 `application.properties` 文件外，还可以使用以下命名约定定义特定于配置文件的属性： `application-{profile}.properties` .  `Environment` 具有一组默认配置文件（默认情况下为 `[default]` ），如果未设置活动配置文件，则使用这些配置文件.换句话说，如果没有显式激活配置文件，则会加载 `application-default.properties` 中的属性.

特定于配置文件的属性从与标准 `application.properties` 相同的位置加载，特定于配置文件的文件始终覆盖非特定文件，无论特定于配置文件的文件是在打包的jar内部还是外部.

如果指定了多个配置文件，则应用last-wins策略.例如， `spring.profiles.active` 属性指定的配置文件将添加到通过 `SpringApplication`  API配置的配置文件之后，因此优先.

> 如果您在 `spring.config.location` 中指定了任何文件，则不会考虑这些文件的特定于配置文件的变体.如果您还想使用特定于配置文件的属性，请使用 `spring.config.location` 中的目录.

## 24.5属性中的占位符

`application.properties` 中的值在使用时通过现有的 `Environment` 进行过滤，因此您可以返回先前定义的值（例如，从系统属性中）.

```java
app.name=MyApp
app.description=${app.name} is a Spring Boot application
```

> 您还可以使用此技术创建现有Spring Boot属性的“简短”变体.有关详细信息，请参阅[Section 77.4, “Use ‘Short’ Command Line Arguments”](howto-properties-and-configuration.html#howto-use-short-command-line-arguments)操作方法.

## 24.6加密属性

Spring Boot没有为加密属性值提供任何内置支持，但是，它确实提供了修改Spring  `Environment` 中包含的值所必需的钩子点.  `EnvironmentPostProcessor` 接口允许您在应用程序启动之前操作 `Environment` .有关详细信息，请参阅[Section 76.3, “Customize the Environment or ApplicationContext Before It Starts”](howto-spring-boot-application.html#howto-customize-the-environment-or-application-context).

如果您正在寻找一种存储凭据和密码的安全方法，[Spring Cloud Vault](https://cloud.spring.io/spring-cloud-vault/)项目支持在[HashiCorp Vault](https://www.vaultproject.io/)中存储外部化配置.

## 24.7使用YAML而不是属性

[YAML](http://yaml.org)是JSON的超集，因此是用于指定分层配置数据的便捷格式.只要在类路径上有[SnakeYAML](http://www.snakeyaml.org/)库， `SpringApplication` 类就会自动支持YAML作为属性的替代.

> 如果您使用“Starters”，则 `spring-boot-starter` 会自动提供SnakeYAML.

### 24.7.1加载YAML

Spring Framework提供了两个方便的类，可用于加载YAML文档.  `YamlPropertiesFactoryBean` 将YAML加载为 `Properties` ， `YamlMapFactoryBean` 将YAML加载为 `Map` .

例如，请考虑以下YAML文档：

```java
environments:
	dev:
		url: http://dev.example.com
		name: Developer Setup
	prod:
		url: http://another.example.com
		name: My Cool App
```

前面的示例将转换为以下属性：

```java
environments.dev.url=http://dev.example.com
environments.dev.name=Developer Setup
environments.prod.url=http://another.example.com
environments.prod.name=My Cool App
```

YAML列表用 `[index]`  dereferencers表示为属性键.例如，考虑以下YAML：

```java
my:
servers:
	- dev.example.com
	- another.example.com
```

前面的示例将转换为这些属性：

```java
my.servers[0]=dev.example.com
my.servers[1]=another.example.com
```

要使用Spring Boot的 `Binder` 实用程序（这是 `@ConfigurationProperties` 所做的）绑定到这样的属性，你需要在 `java.util.List` （或 `Set` ）类型的目标bean中有一个属性，你需要提供一个setter或者用一个setter初始化它.可变Value.例如，以下示例绑定到前面显示的属性：

```java
@ConfigurationProperties(prefix="my")
public class Config {

	private List<String> servers = new ArrayList<String>();

	public List<String> getServers() {
		return this.servers;
	}
}
```

### 24.7.2在Spring环境中将YAML公开为属性

`YamlPropertySourceLoader` 类可用于在Spring  `Environment` 中将YAML公开为 `PropertySource` .这样做可以让您使用带占位符语法的 `@Value` 注释来访问YAML属性.

### 24.7.3多个档案的YAML文件

您可以使用 `spring.profiles` 键指定单个文件中的多个特定于配置文件的YAML文档，以指示文档何时应用，如以下示例所示：

```java
server:
	address: 192.168.1.100
---
spring:
	profiles: development
server:
	address: 127.0.0.1
---
spring:
	profiles: production & eu-central
server:
	address: 192.168.1.120
```

在前面的示例中，如果 `development` 配置文件处于活动状态，则 `server.address` 属性为 `127.0.0.1` .同样，如果 `production`   **and**   `eu-central` 配置文件处于活动状态，则 `server.address` 属性为 `192.168.1.120` .如果 `development` ， `production` 和 `eu-central` 配置文件已启用 **not** ，则属性的值为 `192.168.1.100` .

因此，
>  `spring.profiles` 可以包含简单的配置文件名称（例如 `production` ）或配置文件表达式.概要表达式允许表达更复杂的概要逻辑，例如 `production & (eu-central | eu-west)` .查看[reference guide](https://docs.spring.io/spring/docs/5.1.2.RELEASE/spring-framework-reference/core.html#beans-definition-profiles-java)了解更多详情.

如果在应用程序上下文启动时没有显式激活，则激活默认配置文件.因此，在以下YAML中，我们在"default"配置文件中为 `spring.security.user.password` 设置了一个可用的值 **only** ：

```java
server:
port: 8000
---
spring:
profiles: default
security:
user:
password: weak
```

然而，在以下示例中，始终设置密码，因为它未附加到任何配置文件，并且必须在必要时在所有其他配置文件中显式重置：

```java
server:
port: 8000
spring:
security:
user:
password: weak
```

使用 `!` 元素指定的spring配置文件可以选择使用 `!` 字符来取消.如果为单个指定了否定和非否定的配置文件文档，至少一个非否定的配置文件必须匹配，并且没有否定的配置文件可能匹配.

### 24.7.4 YAML缺点

无法使用 `@PropertySource` 注释加载YAML文件.因此，如果您需要以这种方式加载值，则需要使用属性文件.

## 24.8类型安全的配置属性

使用 `@Value("${property}")` 注释来注入配置属性有时会很麻烦，特别是如果您正在处理多个属性或者您的数据本质上是分层的. Spring Boot提供了一种使用属性的替代方法，该方法允许强类型bean管理和验证应用程序的配置，如以下示例所示：

```java
package com.example;

import java.net.InetAddress;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("acme")
public class AcmeProperties {

	private boolean enabled;

	private InetAddress remoteAddress;

	private final Security security = new Security();

	public boolean isEnabled() { ... }

	public void setEnabled(boolean enabled) { ... }

	public InetAddress getRemoteAddress() { ... }

	public void setRemoteAddress(InetAddress remoteAddress) { ... }

	public Security getSecurity() { ... }

	public static class Security {

		private String username;

		private String password;

		private List<String> roles = new ArrayList<>(Collections.singleton("USER"));

		public String getUsername() { ... }

		public void setUsername(String username) { ... }

		public String getPassword() { ... }

		public void setPassword(String password) { ... }

		public List<String> getRoles() { ... }

		public void setRoles(List<String> roles) { ... }

	}
}
```

前面的POJO定义了以下属性：

-  `acme.enabled` ，默认值为 `false` .

-  `acme.remote-address` ，其类型可以从 `String` 强制执行.

-  `acme.security.username` ，带有嵌套的"security"对象，其名称由属性名称决定.特别是，返回类型根本没有使用，可能是 `SecurityProperties` .

-  `acme.security.password` .

-  `acme.security.roles` ，收集了 `String` .

> Getters和setter通常是必需的，因为绑定是通过标准的Java Beans属性描述符，就像在Spring MVC中一样.在下列情况下可以省略setter：

> 参见[differences between @Value and @ConfigurationProperties](boot-features-external-config.html#boot-features-external-config-vs-value).

您还需要列出要在 `@EnableConfigurationProperties` 注释中注册的属性类，如以下示例所示：

```java
@Configuration
@EnableConfigurationProperties(AcmeProperties.class)
public class MyConfiguration {
}
```

> 当 `@ConfigurationProperties`  bean以这种方式注册时，bean具有常规名称： `<prefix>-<fqn>` ，其中 `<prefix>` 是 `@ConfigurationProperties` 注释中指定的环境键前缀， `<fqn>` 是bean的完全限定名称.如果注释未提供任何前缀，则仅使用bean的完全限定名称.

即使前面的配置为 `AcmeProperties` 创建了一个常规bean，我们也建议 `@ConfigurationProperties` 只处理环境，特别是不会从上下文中注入其他bean.话虽如此， `@EnableConfigurationProperties` 注释也会自动应用于您的项目，以便从 `Environment` 配置任何使用 `@ConfigurationProperties` 注释的现有bean.您可以通过确保 `AcmeProperties` 已经是bean来快捷 `MyConfiguration` ，如以下示例所示：

```java
@Component
@ConfigurationProperties(prefix="acme")
public class AcmeProperties {

	// ... see the preceding example

}
```

这种配置样式特别适用于 `SpringApplication` 外部YAML配置，如以下示例所示：

```java
# application.yml

acme:
	remote-address: 192.168.1.1
	security:
		username: admin
		roles:
		  - USER
		  - ADMIN

# additional configuration as required
```

要使用 `@ConfigurationProperties`  beans，您可以使用与任何其他bean相同的方式注入它们，如以下示例所示：

```java
@Service
public class MyService {

	private final AcmeProperties properties;

	@Autowired
	public MyService(AcmeProperties properties) {
	    this.properties = properties;
	}

	//...

	@PostConstruct
	public void openConnection() {
		Server server = new Server(this.properties.getRemoteAddress());
		// ...
	}

}
```

> 使用 `@ConfigurationProperties` 还允许您生成元数据文件，IDE可以使用这些文件为您自己的密钥提供自动完成功能.有关详细信息，请参阅[Appendix B, Configuration Metadata](configuration-metadata.html)附录.

### 24.8.1第三方配置

除了使用 `@ConfigurationProperties` 注释类之外，您还可以在公共 `@Bean` 方法上使用它.当您想要将属性绑定到控件之外的第三方组件时，这样做会特别有用.

要从 `Environment` 属性配置bean，请将 `@ConfigurationProperties` 添加到其bean注册中，如以下示例所示：

```java
@ConfigurationProperties(prefix = "another")
@Bean
public AnotherComponent anotherComponent() {
	...
}
```

使用 `another` 前缀定义的任何属性都以与前面的 `AcmeProperties` 示例类似的方式映射到该 `AnotherComponent`  bean.

### 24.8.2轻松装订

Spring Boot使用一些宽松的规则将 `Environment` 属性绑定到 `@ConfigurationProperties`  beans，因此 `Environment` 属性名称和bean属性名称之间不需要完全匹配.这有用的常见示例包括破折号分隔的环境属性（例如， `context-path` 绑定到 `contextPath` ）和大写环境属性（例如， `PORT` 绑定到 `port` ）.

例如，请考虑以下 `@ConfigurationProperties` 类：

```java
@ConfigurationProperties(prefix="acme.my-project.person")
public class OwnerProperties {

	private String firstName;

	public String getFirstName() {
		return this.firstName;
	}

	public void setFirstName(String firstName) {
		this.firstName = firstName;
	}

}
```

在前面的示例中，可以使用以下属性名称：

**Table 24.1. relaxed binding** 

|地产|注意|
| ---- | ---- |
|  `acme.my-project.person.first-name`  | Kebab案例，建议在 `.properties` 和 `.yml` 文件中使用. |
|  `acme.myProject.person.firstName`  |标准的驼峰案例语法. |
|  `acme.my_project.person.first_name`  |下划线表示法，这是在 `.properties` 和 `.yml` 文件中使用的替代格式. |
|  `ACME_MYPROJECT_PERSON_FIRSTNAME`  |大写格式，使用系统环境变量时建议使用. |

> 注释的 `prefix` 值必须为kebab大小写（小写且由 `-` 分隔，例如 `acme.my-project.person` ）.

**Table 24.2. relaxed binding rules per property source** 

|properties来源|简单|列表|
| ---- | ---- | ---- |
|属性文件|驼峰大小写，烤肉串案例或下划线表示法|使用 `[ ]` 或逗号分隔值的标准列表语法|
| YAML文件| Camel案例，烤肉串案例或下划线表示法|标准YAML列表语法或逗号分隔值|
|环境变量|大写格式，下划线作为分隔符.  `_` 不应在属性名称中使用|由下划线包围的数字值，例如 `MY_ACME_1_OTHER = my.acme[1].other`  |
|系统属性|驼峰大小写，烤肉串案例或下划线表示法|使用 `[ ]` 或逗号分隔值的标准列表语法|

> We建议在可能的情况下，以小写的烤肉串格式存储属性，例如 `my.property-name=acme` .

绑定到 `Map` 属性时，如果 `key` 包含其他任何内容与小写字母数字字符或 `-` 相比，您需要使用括号表示法，以便保留原始值.如果该键未被 `[]` 包围，则删除任何非字母数字或 `-` 的字符.例如，考虑将以下属性绑定到 `Map` ：

```java
acme:
map:
"[/key1]": value1
"[/key2]": value2
/key3: value3
```

上面的属性将绑定到 `Map` ， `/key1` ， `/key2` 和 `key3` 作为Map中的键.

### 24.8.3合并复杂类型

当列表在多个位置配置时，覆盖通过替换整个列表来工作.

例如，假设 `MyPojo` 对象具有 `name` 和 `description` 属性，默认情况下为 `null` .以下示例从 `AcmeProperties` 公开 `MyPojo` 对象的列表：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final List<MyPojo> list = new ArrayList<>();

	public List<MyPojo> getList() {
		return this.list;
	}

}
```

请考虑以下配置：

```java
acme:
list:
- name: my name
description: my description
---
spring:
profiles: dev
acme:
list:
- name: my another name
```

如果 `dev` 配置文件未激活，则 `AcmeProperties.list` 包含一个 `MyPojo` 条目，如先前定义的那样.但是，如果启用了 `dev` 配置文件，则 `list` 仍然只包含一个条目（名称为 `my another name` ，描述为 `null` ）.此配置不会向列表中添加第二个 `MyPojo` 实例，也不会合并项目.

在多个配置文件中指定 `List` 时，将使用具有最高优先级（并且仅具有该优先级）的配置文件.请考虑以下示例：

```java
acme:
list:
- name: my name
description: my description
- name: another name
description: another description
---
spring:
profiles: dev
acme:
list:
- name: my another name
```

在前面的示例中，如果 `dev` 配置文件处于活动状态， `AcmeProperties.list` 包含一个 `MyPojo` 条目（名称为 `my another name` ，描述为 `null` ）.对于YAML，逗号分隔列表和YAML列表都可用于完全覆盖列表的内容.

对于 `Map` 属性，您可以绑定从多个源中提取的属性值.但是，对于多个源中的相同属性，使用具有最高优先级的属性.以下示例从 `AcmeProperties` 公开 `Map<String, MyPojo>` ：

```java
@ConfigurationProperties("acme")
public class AcmeProperties {

	private final Map<String, MyPojo> map = new HashMap<>();

	public Map<String, MyPojo> getMap() {
		return this.map;
	}

}
```

请考虑以下配置：

```java
acme:
map:
key1:
name: my name 1
description: my description 1
---
spring:
profiles: dev
acme:
map:
key1:
name: dev name 1
key2:
name: dev name 2
description: dev description 2
```

如果 `dev` 配置文件未激活， `AcmeProperties.map` 包含一个带有键 `key1` 的条目（名称为 `my name 1` ，描述为 `my description 1` ）.但是，如果启用了 `dev` 配置文件，则 `map` 包含两个条目，其中键 `key1` （名称为 `dev name 1` ，描述为 `my description 1` ）和 `key2` （名称为 `dev name 2` ，描述为 `dev description 2` ）.

> 前面的合并规则适用于所有属性来源的属性，而不仅仅是YAML文件.

### 24.8.4属性转换

当Spring绑定到 `@ConfigurationProperties`  bean时，Spring Boot会尝试将外部应用程序属性强制转换为正确的类型.如果需要自定义类型转换，可以提供 `ConversionService`  bean（名为 `conversionService` 的bean）或自定义属性编辑器（通过 `CustomEditorConfigurer`  bean）或自定义 `Converters` （bean定义注释为 `@ConfigurationPropertiesBinding` ）.

> 由于在应用程序生命周期中很早就请求了此bean，因此请确保限制 `ConversionService` 正在使用的依赖项.通常，您在创建时可能无法完全初始化所需的任何依赖项.如果配置密钥强制不需要，您可能需要重命名自定义 `ConversionService` ，并且只依赖于使用 `@ConfigurationPropertiesBinding` 限定的自定义转换器.

#### 转换持续时间

Spring Boot专门支持表达持续时间.如果公开 `java.time.Duration` 属性，则可以使用应用程序属性中的以下格式：

- A常规 `long` 表示（除非指定了 `@DurationUnit` ，否则使用毫秒作为默认单位）

- 标准ISO-8601格式[used by java.util.Duration](https://docs.oracle.com/javase/8/docs/api//java/time/Duration.html#parse-java.lang.CharSequence-)

- 更可读的格式，其中值和单位耦合（例如 `10s` 表示10秒）

请考虑以下示例：

```java
@ConfigurationProperties("app.system")
public class AppSystemProperties {

	@DurationUnit(ChronoUnit.SECONDS)
	private Duration sessionTimeout = Duration.ofSeconds(30);

	private Duration readTimeout = Duration.ofMillis(1000);

	public Duration getSessionTimeout() {
		return this.sessionTimeout;
	}

	public void setSessionTimeout(Duration sessionTimeout) {
		this.sessionTimeout = sessionTimeout;
	}

	public Duration getReadTimeout() {
		return this.readTimeout;
	}

	public void setReadTimeout(Duration readTimeout) {
		this.readTimeout = readTimeout;
	}

}
```

要指定30秒的会话超时， `30` ， `PT30S` 和 `30s` 都是等效的.可以使用以下任何一种形式指定500ms的读取超时： `500` ， `PT0.5S` 和 `500ms` .

您也可以使用任何支持的单位.这些是：

-  `ns` 为纳秒

-  `us` 微秒

-  `ms` 毫秒

-  `s` 秒

-  `m` 分钟

-  `h` 几个小时

-  `d` 天

默认单位是毫秒，可以使用 `@DurationUnit` 覆盖，如上面的示例所示.

> 如果您从以前只使用 `Long` 来表示持续时间的版本升级，请确保定义单位（使用 `@DurationUnit` ），如果它不是切换到 `Duration` 旁边的毫秒.这样做可以提供透明的升级路径，同时支持更丰富的格式.

#### Converting Data Sizes

Spring Framework有一个 `DataSize` 值类型，允许以字节为单位表示大小.如果公开 `DataSize` 属性，则可以使用应用程序属性中的以下格式：

- A常规 `long` 表示（除非指定了 `@DataSizeUnit` ，否则使用字节作为默认单位）

- 更可读的格式，其中值和单位耦合（例如 `10MB` 表示10兆字节）

请考虑以下示例：

```java
@ConfigurationProperties("app.io")
public class AppIoProperties {

	@DataSizeUnit(DataUnit.MEGABYTES)
	private DataSize bufferSize = DataSize.ofMegabytes(2);

	private DataSize sizeThreshold = DataSize.ofBytes(512);

	public DataSize getBufferSize() {
		return this.bufferSize;
	}

	public void setBufferSize(DataSize bufferSize) {
		this.bufferSize = bufferSize;
	}

	public DataSize getSizeThreshold() {
		return this.sizeThreshold;
	}

	public void setSizeThreshold(DataSize sizeThreshold) {
		this.sizeThreshold = sizeThreshold;
	}

}
```

要指定10兆字节的缓冲区大小， `10` 和 `10MB` 是等效的.可以将大小阈值256字节指定为 `256` 或 `256B` .

您也可以使用任何支持的单位.这些是：

-  `B` 表示字节数

-  `KB`  for千字节

-  `MB` ，兆字节

-  `GB` 为千兆字节

-  `TB` 对于太字节

默认单位是字节，可以使用 `@DataSizeUnit` 覆盖，如上面的示例所示.

> 如果要从以前只使用 `Long` 表示大小的版本进行升级，请确保定义单位（使用 `@DataSizeUnit` ），如果它不是切换到 `DataSize` 旁边的字节.这样做可以提供透明的升级路径，同时支持更丰富的格式.

### 24.8.5 @ConfigurationProperties验证

只要使用Spring的 `@Validated` 注释注释，Spring Boot就会尝试验证 `@ConfigurationProperties` 类.您可以直接在配置类上使用JSR-303  `javax.validation` 约束注释.为此，请确保符合条件的JSR-303实现位于类路径上，然后将约束注释添加到字段中，如以下示例所示：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

	@NotNull
	private InetAddress remoteAddress;

	// ... getters and setters

}
```

> 您还可以通过使用 `@Validated` 注释创建配置属性的 `@Bean` 方法来触发验证.

虽然嵌套属性也会在绑定时进行验证，但最好还是将关联字段注释为 `@Valid` .这可确保即使未找到嵌套属性也会触发验证.以下示例基于前面的 `AcmeProperties` 示例构建：

```java
@ConfigurationProperties(prefix="acme")
@Validated
public class AcmeProperties {

	@NotNull
	private InetAddress remoteAddress;

	@Valid
	private final Security security = new Security();

	// ... getters and setters

	public static class Security {

		@NotEmpty
		public String username;

		// ... getters and setters

	}

}
```

您还可以通过创建名为 `configurationPropertiesValidator` 的bean定义来添加自定义Spring  `Validator` .应将 `@Bean` 方法声明为 `static` .配置属性验证器是在应用程序生命周期的早期创建的，并且将 `@Bean` 方法声明为static可以创建bean而无需实例化 `@Configuration` 类.这样做可以避免早期实例化可能导致的任何问题.有一个[property validation sample](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/spring-boot-sample-property-validation)显示如何设置.

>   `spring-boot-actuator` 模块包含一个公开所有 `@ConfigurationProperties`  bean的endpoints.将Web浏览器指向 `/actuator/configprops` 或使用等效的JMXendpoints.有关详细信息，请参阅“[Production ready features](production-ready-endpoints.html)”部分.

### 24.8.6 @ConfigurationProperties vs. @Value

`@Value` 注释是核心容器功能，它不提供与类型安全配置属性相同的功能.下表总结了 `@ConfigurationProperties` 和 `@Value` 支持的功能：

|功能|  `@ConfigurationProperties`  |  `@Value`  |
| ---- | ---- | ---- |
| [Relaxed binding](boot-features-external-config.html#boot-features-external-config-relaxed-binding) |是|否|
| [Meta-data support](configuration-metadata.html) |是|否|
|  `SpEL` 评估|否|是|

如果为自己的组件定义一组配置键，我们建议您将它们分组到使用 `@ConfigurationProperties` 注释的POJO中.您还应该知道，因为 `@Value` 不支持宽松绑定，所以如果您需要使用环境变量来提供值，则它不是一个好的候选者.

最后，虽然您可以在 `@Value` 中编写 `SpEL` 表达式，但不会从[application property files](boot-features-external-config.html#boot-features-external-config-application-property-files)处理此类表达式.


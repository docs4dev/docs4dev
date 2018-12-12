## Appendix B.配置元数据

Spring Boot jar包含元数据文件，提供所有支持的配置属性的详细信息.这些文件旨在让IDE开发人员在用户使用 `application.properties` 或 `application.yml` 文件时提供上下文帮助和“代码完成”.

通过处理所有使用 `@ConfigurationProperties` 注释的项目，大多数元数据文件在编译时自动生成.但是，对于极端情况或更高级的用例，可以[write part of the metadata manually](configuration-metadata.html#configuration-metadata-additional-metadata).

## B.1元数据格式

配置元数据文件位于 `META-INF/spring-configuration-metadata.json` 下的jar中.它们使用简单的JSON格式，其中的项目分类在“groups”或“properties”下，其他值提示分类在"hints"下，如以下示例所示：

```java
{"groups": [
	{
		"name": "server",
		"type": "org.springframework.boot.autoconfigure.web.ServerProperties",
		"sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
	},
	{
		"name": "spring.jpa.hibernate",
		"type": "org.springframework.boot.autoconfigure.orm.jpa.JpaProperties$Hibernate",
		"sourceType": "org.springframework.boot.autoconfigure.orm.jpa.JpaProperties",
		"sourceMethod": "getHibernate()"
	}
	...
],"properties": [
	{
		"name": "server.port",
		"type": "java.lang.Integer",
		"sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
	},
	{
		"name": "server.address",
		"type": "java.net.InetAddress",
		"sourceType": "org.springframework.boot.autoconfigure.web.ServerProperties"
	},
	{
		  "name": "spring.jpa.hibernate.ddl-auto",
		  "type": "java.lang.String",
		  "description": "DDL mode. This is actually a shortcut for the \"hibernate.hbm2ddl.auto\" property.",
		  "sourceType": "org.springframework.boot.autoconfigure.orm.jpa.JpaProperties$Hibernate"
	}
	...
],"hints": [
	{
		"name": "spring.jpa.hibernate.ddl-auto",
		"values": [
			{
				"value": "none",
				"description": "Disable DDL handling."
			},
			{
				"value": "validate",
				"description": "Validate the schema, make no changes to the database."
			},
			{
				"value": "update",
				"description": "Update the schema if necessary."
			},
			{
				"value": "create",
				"description": "Create the schema and destroy previous data."
			},
			{
				"value": "create-drop",
				"description": "Create and then destroy the schema at the end of the session."
			}
		]
	}
]}
```

每个“属性”是用户使用给定值指定的配置项.例如，可以在 `application.properties` 中指定 `server.port` 和 `server.address` ，如下所示：

```java
server.port=9090
server.address=127.0.0.1
```

“组”是更高级别的项目，它们本身不指定值，而是为属性提供上下文分组.例如， `server.port` 和 `server.address` 属性是 `server` 组的一部分.

> 并不要求每个“property”都有一个“组”.某些属性可能本身就存在.

最后，“提示”是用于帮助用户配置给定属性的附加信息.例如，当开发人员配置 `spring.jpa.hibernate.ddl-auto` 属性时，工具可以使用提示为 `none` ， `validate` ， `update` ， `create` 和 `create-drop` 值提供一些自动完成帮助.

### B.1.1组属性

`groups` 数组中包含的JSON对象可以包含下表中显示的属性：

|名称|类型|用途|
| ---- | ---- | ---- |
|  `name`  | String |组的全名.该属性是必需的. |
|  `type`  | String |组的数据类型的类名.例如，如果该组基于使用 `@ConfigurationProperties` 注释的类，则该属性将包含该类的完全限定名称.如果它基于 `@Bean` 方法，则它将是该方法的返回类型.如果类型未知，则可以省略该属性. |
|  `description`  | String |可以向用户显示的组的简短描述.如果没有描述，则可以省略.建议描述为简短段落，第一行提供简明摘要.描述中的最后一行应以句点（ `.` ）结尾. |
|  `sourceType`  | String |贡献此组的源的类名.例如，如果该组基于 `@ConfigurationProperties` 注释的 `@Bean` 方法，则此属性将包含包含该方法的 `@Configuration` 类的完全限定名称.如果源类型未知，则可以省略该属性. |
|  `sourceMethod`  | String |贡献此组的方法的全名（包括括号和参数类型）（例如， `@ConfigurationProperties` 带注释的 `@Bean` 方法的名称）.如果源方法未知，则可以省略. |

### B.1.2属性属性

`properties` 数组中包含的JSON对象可以包含下表中描述的属性：

|名称|类型|用途|
| ---- | ---- | ---- |
|  `name`  | String |属性的全名.名称采用小写的句点分隔形式（例如， `server.address` ）.该属性是必需的. |
|  `type`  | String |属性的数据类型的完整签名（例如， `java.lang.String` ），但也是完整的泛型类型（例如 `java.util.Map<java.util.String,acme.MyEnum>` ）.您可以使用此属性来指导用户可以输入的值类型.为了保持一致性，通过使用其包装对应项来指定基元的类型（例如， `boolean` 变为 `java.lang.Boolean` ）.请注意，此类可能是一个复杂类型，在绑定值时会从 `String` 转换.如果类型未知，则可以省略. |
|  `description`  | String |可以向用户显示的组的简短描述.如果没有可用的描述，则可以省略.建议描述为简短段落，第一行提供简明摘要.描述中的最后一行应以句点（ `.` ）结尾. |
|  `sourceType`  | String |提供此属性的源的类名.例如，如果属性来自使用 `@ConfigurationProperties` 注释的类，则此属性将包含该类的完全限定名称.如果源类型未知，则可以省略. |
|  `defaultValue`  | Object |默认值，如果未指定属性，则使用该值.如果属性的类型是数组，则它可以是值数组.如果默认值未知，则可以省略. |
|  `deprecation`  | Deprecation |指定是否弃用该属性.如果该字段未被弃用或者该信息未知，则可以省略该字段.下表提供了有关 `deprecation` 属性的更多详细信息. |

每个 `properties` 元素的 `deprecation` 属性中包含的JSON对象可以包含以下属性：

|名称|类型|用途|
| ---- | ---- | ---- |
|  `level`  | String |弃用级别，可以是 `warning` （默认值）或 `error` .当属性具有 `warning` 弃用级别时，它仍应绑定在环境中.但是，当它具有 `error` 弃用级别时，该属性不再受管理且未绑定. |
|  `reason`  | String |不推荐使用该属性的简短描述.如果没有可用的原因，可以省略.建议描述为简短段落，第一行提供简明摘要.描述中的最后一行应以句点（ `.` ）结尾. |
|  `replacement`  | String |替换此不推荐使用的属性的属性的全名.如果此属性没有替换，则可以省略. |

> Prior到Spring Boot 1.3之前，可以使用单个 `deprecated` 布尔属性代替 `deprecation` 元素.仍以不推荐的方式支持此功能，不应再使用它.如果没有可用的原因和替换，则应设置空的 `deprecation` 对象.

通过将 `@DeprecatedConfigurationProperty` 注释添加到公开不推荐使用的属性的getter中，也可以在代码中以声明方式指定弃用.例如，假设 `app.acme.target` 属性令人困惑，并重命名为 `app.acme.name` .以下示例显示了如何处理这种情况：

```java
@ConfigurationProperties("app.acme")
public class AcmeProperties {

	private String name;

	public String getName() { ... }

	public void setName(String name) { ... }

	@DeprecatedConfigurationProperty(replacement = "app.acme.name")
	@Deprecated
	public String getTarget() {
		return getName();
	}

	@Deprecated
	public void setTarget(String target) {
		setName(target);
	}
}
```

> 无法设置 `level` .始终假设 `warning` ，因为代码仍在处理属性.

上面的代码确保deprecated属性仍然有效（在后台委托给 `name` 属性）.一旦可以从公共API中删除 `getTarget` 和 `setTarget` 方法，元数据中的自动弃用提示也会消失.如果要保留提示，添加具有 `error` 弃用级别的手动元数据可确保用户仍然了解该属性.当提供 `replacement` 时，这样做特别有用.

### B.1.3提示属性

`hints` 数组中包含的JSON对象可以包含下表中显示的属性：

|名称|类型|用途|
| ---- | ---- | ---- |
|  `name`  | String |此提示引用的属性的全名.名称采用小写的句点分隔形式（例如 `spring.mvc.servlet.path` ）.如果属性引用Map（例如 `system.contexts` ），则提示要么应用于Map的键（ `system.context.keys` ），要么应用于Map的值（ `system.context.values` ）.该属性是必需的. |
|  `values`  | ValueHint [] |  `ValueHint` 对象定义的有效值列表（在下表中描述）.每个条目定义值并可能有描述. |
|  `providers`  | ValueProvider [] |  `ValueProvider` 对象定义的提供程序列表（本文档稍后介绍）.每个条目定义提供者的名称及其参数（如果有）. |

每个 `hint` 元素的 `values` 属性中包含的JSON对象可以包含下表中描述的属性：

|名称|类型|用途|
| ---- | ---- | ---- |
|  `value`  | Object |提示引用的元素的有效值.如果属性的类型是数组，则它也可以是值数组.该属性是必需的. |
|  `description`  | String |可以向用户显示的值的简短描述.如果没有可用的描述，则可以省略.建议描述为简短段落，第一行提供简明摘要.描述中的最后一行应以句点（ `.` ）结尾. |

每个 `hint` 元素的 `providers` 属性中包含的JSON对象可以包含下表中描述的属性：

|名称|类型|用途|
| ---- | ---- | ---- |
|  `name`  | String |用于为提示引用的元素提供其他内容帮助的提供程序的名称. |
|  `parameters`  | JSON对象|提供程序支持的任何其他参数（有关更多详细信息，请查看提供程序的文档）. |

### B.1.4重复的元数据项

具有相同“属性”和“组”名称的对象可以在元数据文件中多次出现.例如，您可以将两个单独的类绑定到同一个前缀，每个类都有可能重叠的属性名称.虽然多次出现在元数据中的相同名称不应该是常见的，但元数据的使用者应该注意确保他们支持它.

## B.2提供手动提示

要改善用户体验并进一步帮助用户配置给定属性，您可以提供以下其他元数据：

- 描述属性的潜在值列表.

- Associates一个提供者，将一个定义良好的语义附加到属性，以便工具可以根据项目的上下文发现潜在值列表.

### B.2.1值提示

每个提示的 `name` 属性引用属性的 `name` .在[initial example shown earlier](configuration-metadata.html#configuration-metadata-format)中，我们为 `spring.jpa.hibernate.ddl-auto` 属性提供了五个值： `none` ， `validate` ， `update` ， `create` 和 `create-drop` .每个值也可以有描述.

如果您的属性类型为 `Map` ，则可以提供键和值的提示（但不能提供Map本身的提示）.特殊的 `.keys` 和 `.values` 后缀必须分别引用键和值.

假设 `sample.contexts` 将magic  `String` 值映射为整数，如以下示例所示：

```java
@ConfigurationProperties("sample")
public class SampleProperties {

	private Map<String,Integer> contexts;
	// getters and setters
}
```

神奇的值是（在这个例子中）是 `sample1` 和 `sample2` .为了为密钥提供额外的内容帮助，您可以将以下JSON添加到[the manual metadata of the module](configuration-metadata.html#configuration-metadata-additional-metadata)：

```java
{"hints": [
	{
		"name": "sample.contexts.keys",
		"values": [
			{
				"value": "sample1"
			},
			{
				"value": "sample2"
			}
		]
	}
]}
```

> We建议您使用 `Enum` 代替这两个值.如果您的IDE支持它，这是迄今为止最有效的自动完成方法.

### B.2.2Value提供者

提供程序是将语义附加到属性的强大方法.在本节中，我们定义了可用于您自己的提示的官方提供程序.但是，您最喜欢的IDE可能会实现其中一些或不实现.此外，它最终可以提供自己的.

> 由于这是一项新功能，IDE供应商必须了解它的工作原理.采用时间自然会有所不同.

下表总结了支持的提供程序列表：

|名称|说明|
| ---- | ---- |
|  `any`  |允许提供任何其他值. |
|  `class-reference`  |自动完成项目中可用的类.通常由 `target` 参数指定的基类约束. |
|  `handle-as`  |处理属性，就好像它是由强制 `target` 参数定义的类型定义的一样. |
|  `logger-name`  |自动完成有效的Logger名称和[logger groups](boot-features-logging.html#boot-features-custom-log-groups).通常，当前项目中可用的包名和类名可以自动完成，也可以自定义组. |
|  `spring-bean-reference`  |自动完成当前项目中的可用Bean名称.通常由 `target` 参数指定的基类约束. |
|  `spring-profile-name`  |自动完成项目中可用的Spring配置文件名称. |

> 只有一个提供者可以为给定的属性处于活动状态，但如果他们都能以某种方式管理属性，则可以指定多个提供者.确保首先放置最强大的提供程序，因为IDE必须使用它可以处理的JSON部分中的第一个.如果不支持给定属性的提供者，则也不提供特殊内容帮助.

#### Any

特殊的 **any** 提供程序值允许提供任何其他值.如果支持，则应应用基于属性类型的常规值验证.

如果您有值列表并且任何额外值仍应被视为有效，则通常使用此提供程序.

以下示例提供 `on` 和 `off` 作为 `system.state` 的自动完成值：

```java
{"hints": [
	{
		"name": "system.state",
		"values": [
			{
				"value": "on"
			},
			{
				"value": "off"
			}
		],
		"providers": [
			{
				"name": "any"
			}
		]
	}
]}
```

请注意，在前面的示例中，还允许任何其他值.

#### Class参考

**class-reference** 提供程序自动完成项目中可用的类.此提供程序支持以下参数：

|参数|类型|默认值|描述|
| ---- | ---- | ---- | ---- |
|  `target`  |  `String` （ `Class` ）|无|应分配给所选值的类的完全限定名称.通常用于过滤非候选类.请注意，通过公开具有适当上限的类，可以通过类型本身提供此信息. |
|  `concrete`  |  `boolean`  | true |指定是否仅将具体类视为有效候选. |

以下元数据片段对应于标准 `server.servlet.jsp.class-name` 属性，该属性定义要使用的 `JspServlet` 类名称：

```java
{"hints": [
	{
		"name": "server.servlet.jsp.class-name",
		"providers": [
			{
				"name": "class-reference",
				"parameters": {
					"target": "javax.servlet.http.HttpServlet"
				}
			}
		]
	}
]}
```

#### Handle As

**handle-as** 提供程序允许您将属性的类型替换为更高级别的类型.当属性具有 `java.lang.String` 类型时，通常会发生这种情况，因为您不希望配置类依赖于可能不在类路径上的类.此提供程序支持以下参数：

|参数|类型|默认值|描述|
| ---- | ---- | ---- | ---- |
|  **target**  |  `String` （ `Class` ）|无|要为属性考虑的类型的完全限定名称.此参数是必需的. |

可以使用以下类型：

- Any  `java.lang.Enum` ：列出属性的可能值. （我们建议使用 `Enum` 类型定义属性，因为IDE不需要进一步提示来自动完成值.）

-  `java.nio.charset.Charset` ：支持自动完成字符集/编码值（例如 `UTF-8` ）

-  `java.util.Locale` ：自动完成语言环境（例如 `en_US` ）

-  `org.springframework.util.MimeType` ：支持自动完成内容类型值（例如 `text/plain` ）

-  `org.springframework.core.io.Resource` ：支持Spring的资源抽象的自动完成，以引用文件系统或类路径上的文件. （例如 `classpath:/sample.properties` ）

> 如果可以提供多个值，请使用 `Collection` 或Array类型向IDE讲授它.

以下元数据片段对应于标准 `spring.liquibase.change-log` 属性，该属性定义要使用的更改日志的路径.它实际上在内部用作 `org.springframework.core.io.Resource` 但不能这样暴露，因为我们需要保留原始的String值以将其传递给Liquibase API.

```java
{"hints": [
	{
		"name": "spring.liquibase.change-log",
		"providers": [
			{
				"name": "handle-as",
				"parameters": {
					"target": "org.springframework.core.io.Resource"
				}
			}
		]
	}
]}
```

#### Logger名称

**logger-name** 提供程序自动完成有效的Logger名称和[logger groups](boot-features-logging.html#boot-features-custom-log-groups).通常，可以自动完成当前项目中可用的包名和类名.如果启用了组（默认），并且在配置中标识了自定义Logger组，则应提供自动完成组.特定框架可能还有额外的魔术Logger名称，也可以支持.

此提供程序支持以下参数：

|参数|类型|默认值|描述|
| ---- | ---- | ---- | ---- |
|  `group`  |  `boolean`  |  `true`  |指定是否应考虑已知组. |

由于Logger名称可以是任意名称，因此此提供程序应允许任何值，但可以突出显示项目类路径中不可用的有效包名和类名.

以下元数据片段对应于标准 `logging.level` 属性.键是Logger名称，值对应于标准日志级别或任何自定义级别.由于Spring Boot定义了几个开箱即用的Logger组，因此为这些组件添加了专用值提示.

```java
{"hints": [
	{
		"name": "logging.level.keys",
		"values": [
			{
				"value": "root",
				"description": "Root logger used to assign the default logging level."
			},
			{
				"value": "sql",
				"description": "SQL logging group including Hibernate SQL logger."
			},
			{
				"value": "web",
				"description": "Web logging group including codecs."
			}
		],
		"providers": [
			{
				"name": "logger-name"
			}
		]
	},
	{
		"name": "logging.level.values",
		"values": [
			{
				"value": "trace"
			},
			{
				"value": "debug"
			},
			{
				"value": "info"
			},
			{
				"value": "warn"
			},
			{
				"value": "error"
			},
			{
				"value": "fatal"
			},
			{
				"value": "off"
			}

		],
		"providers": [
			{
				"name": "any"
			}
		]
	}
]}
```

#### Spring Bean参考

**spring-bean-reference** 提供程序自动完成在当前项目的配置中定义的bean.此提供程序支持以下参数：

|参数|类型|默认值|描述|
| ---- | ---- | ---- | ---- |
|  `target`  |  `String` （ `Class` ）|无|应分配给候选者的bean类的完全限定名称.通常用于过滤掉非候选bean. |

以下元数据片段对应于标准 `spring.jmx.server` 属性，该属性定义要使用的 `MBeanServer`  bean的名称：

```java
{"hints": [
	{
		"name": "spring.jmx.server",
		"providers": [
			{
				"name": "spring-bean-reference",
				"parameters": {
					"target": "javax.management.MBeanServer"
				}
			}
		]
	}
]}
```

> Binders不知道元数据.如果提供该提示，则仍需要使用 `ApplicationContext` 将bean名称转换为实际的Bean引用.

#### Spring个人资料名称

**spring-profile-name** 提供程序自动完成在当前项目的配置中定义的Spring配置文件.

以下元数据片段对应于标准 `spring.profiles.active` 属性，该属性定义要启用的Spring配置文件的名称：

```java
{"hints": [
	{
		"name": "spring.profiles.active",
		"providers": [
			{
				"name": "spring-profile-name"
			}
		]
	}
]}
```

## B.3使用注释处理器生成自己的元数据

您可以使用 `spring-boot-configuration-processor`  jar从使用 `@ConfigurationProperties` 注释的项目轻松生成自己的配置元数据文件. jar包含一个Java注释处理器，在您编译项目时调用该处理器.要使用处理器，请在 `spring-boot-configuration-processor` 上包含依赖项.

使用Maven时，依赖项应声明为可选，如以下示例所示：

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>
```

使用Gradle 4.5及更早版本时，应在 `compileOnly` 配置中声明依赖项，如以下示例所示：

```java
dependencies {
	compileOnly "org.springframework.boot:spring-boot-configuration-processor"
}
```

使用Gradle 4.6及更高版本时，应在 `annotationProcessor` 配置中声明依赖项，如以下示例所示：

```java
dependencies {
	annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"
}
```

如果您使用 `additional-spring-configuration-metadata.json` 文件，则应将 `compileJava` 任务配置为依赖于 `processResources` 任务，如以下示例所示：

```java
compileJava.dependsOn(processResources)
```

此依赖关系确保在编译期间注释处理器运行时可以使用其他元数据.

处理器选择使用 `@ConfigurationProperties` 注释的类和方法.配置类中的字段值的Javadoc用于填充 `description` 属性.

> 您应该只使用带有 `@ConfigurationProperties` 字段Javadoc的简单文本，因为它们在添加到JSON之前不会被处理.

通过存在标准的getter和setter来发现属性，这些getter和setter具有对集合类型的特殊处理（即使只有getter存在也会检测到）.注释处理器还支持使用 `@Data` ， `@Getter` 和 `@Setter`  lombok注释.

> 如果您在项目中使用AspectJ，则需要确保注释处理器只运行一次.有几种方法可以做到这一点.使用Maven，您可以显式配置 `maven-apt-plugin` 并仅在那里将依赖项添加到注释处理器.您还可以让AspectJ插件在 `maven-compiler-plugin` 配置中运行所有处理并禁用注释处理，如下所示：

### B.3.1嵌套属性

注释处理器自动将内部类视为嵌套属性.考虑以下课程：

```java
@ConfigurationProperties(prefix="server")
public class ServerProperties {

	private String name;

	private Host host;

	// ... getter and setters

	public static class Host {

		private String ip;

		private int port;

		// ... getter and setters

	}

}
```

上面的示例为 `server.name` ， `server.host.ip` 和 `server.host.port` 属性生成元数据信息.您可以在字段上使用 `@NestedConfigurationProperty` 注释来指示应将常规（非内部）类视为嵌套.

> 这对集合和映射没有影响，因为这些类型是自动标识的，并且为每个类型生成单个元数据属性.

### B.3.2添加其他元数据

Spring Boot的配置文件处理非常灵活，通常情况下可能存在未绑定到a的属性 `@ConfigurationProperties`  beans.您可能还需要调整现有密钥的某些属性.为支持此类情况并允许您提供自定义"hints"，注释处理器会自动将 `META-INF/additional-spring-configuration-metadata.json` 中的项目合并到主元数据文件中.

如果引用自动检测到的属性，则会覆盖描述，默认值和弃用信息（如果已指定）.如果在当前模块中未标识手动属性声明，则将其添加为新属性.

`additional-spring-configuration-metadata.json` 文件的格式与常规 `spring-configuration-metadata.json` 完全相同.附加属性文件是可选的.如果您没有任何其他属性，请不要添加该文件.


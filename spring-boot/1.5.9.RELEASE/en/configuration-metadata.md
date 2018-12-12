## Appendix B. Configuration Metadata

Spring Boot jars include metadata files that provide details of all supported configuration properties. The files are designed to let IDE developers offer contextual help and “code completion” as users are working with  `application.properties`  or  `application.yml`  files.

The majority of the metadata file is generated automatically at compile time by processing all items annotated with  `@ConfigurationProperties` . However, it is possible to [write part of the metadata manually](configuration-metadata.html#configuration-metadata-additional-metadata) for corner cases or more advanced use cases.

## B.1 Metadata Format

Configuration metadata files are located inside jars under  `META-INF/spring-configuration-metadata.json`  They use a simple JSON format with items categorized under either “groups” or “properties” and additional values hints categorized under "hints", as shown in the following example:

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

Each “property” is a configuration item that the user specifies with a given value. For example,  `server.port`  and  `server.address`  might be specified in  `application.properties` , as follows:

```java
server.port=9090
server.address=127.0.0.1
```

The “groups” are higher level items that do not themselves specify a value but instead provide a contextual grouping for properties. For example, the  `server.port`  and  `server.address`  properties are part of the  `server`  group.

> It is not required that every “property” has a “group”. Some properties might exist in their own right.

Finally, “hints” are additional information used to assist the user in configuring a given property. For example, when a developer is configuring the  `spring.jpa.hibernate.ddl-auto`  property, a tool can use the hints to offer some auto-completion help for the  `none` ,  `validate` ,  `update` ,  `create` , and  `create-drop`  values.

### B.1.1 Group Attributes

The JSON object contained in the  `groups`  array can contain the attributes shown in the following table:

|Name|Type|Purpose|
|----|----|----|
| `name`  |String |The full name of the group. This attribute is mandatory. |
| `type`  |String |The class name of the data type of the group. For example, if the group were based on a class annotated with  `@ConfigurationProperties` , the attribute would contain the fully qualified name of that class. If it were based on a  `@Bean`  method, it would be the return type of that method. If the type is not known, the attribute may be omitted. |
| `description`  |String |A short description of the group that can be displayed to users. If not description is available, it may be omitted. It is recommended that descriptions be short paragraphs, with the first line providing a concise summary. The last line in the description should end with a period ( `.` ). |
| `sourceType`  |String |The class name of the source that contributed this group. For example, if the group were based on a  `@Bean`  method annotated with  `@ConfigurationProperties` , this attribute would contain the fully qualified name of the  `@Configuration`  class that contains the method. If the source type is not known, the attribute may be omitted. |
| `sourceMethod`  |String |The full name of the method (include parenthesis and argument types) that contributed this group (for example, the name of a  `@ConfigurationProperties`  annotated  `@Bean`  method). If the source method is not known, it may be omitted. |

### B.1.2 Property Attributes

The JSON object contained in the  `properties`  array can contain the attributes described in the following table:

|Name|Type|Purpose|
|----|----|----|
| `name`  |String |The full name of the property. Names are in lower-case period-separated form (for example,  `server.address` ). This attribute is mandatory. |
| `type`  |String |The full signature of the data type of the property (for example,  `java.lang.String` ) but also a full generic type (such as  `java.util.Map<java.util.String,acme.MyEnum>` ). You can use this attribute to guide the user as to the types of values that they can enter. For consistency, the type of a primitive is specified by using its wrapper counterpart (for example,  `boolean`  becomes  `java.lang.Boolean` ). Note that this class may be a complex type that gets converted from a  `String`  as values are bound. If the type is not known, it may be omitted. |
| `description`  |String |A short description of the group that can be displayed to users. If no description is available, it may be omitted. It is recommended that descriptions be short paragraphs, with the first line providing a concise summary. The last line in the description should end with a period ( `.` ). |
| `sourceType`  |String |The class name of the source that contributed this property. For example, if the property were from a class annotated with  `@ConfigurationProperties` , this attribute would contain the fully qualified name of that class. If the source type is unknown, it may be omitted. |
| `defaultValue`  |Object |The default value, which is used if the property is not specified. If the type of the property is an array, it can be an array of value(s). If the default value is unknown, it may be omitted. |
| `deprecation`  |Deprecation |Specify whether the property is deprecated. If the field is not deprecated or if that information is not known, it may be omitted. The next table offers more detail about the  `deprecation`  attribute. |

The JSON object contained in the  `deprecation`  attribute of each  `properties`  element can contain the following attributes:

|Name|Type|Purpose|
|----|----|----|
| `level`  |String |The level of deprecation, which can be either  `warning`  (the default) or  `error` . When a property has a  `warning`  deprecation level, it should still be bound in the environment. However, when it has an  `error`  deprecation level, the property is no longer managed and is not bound. |
| `reason`  |String |A short description of the reason why the property was deprecated. If no reason is available, it may be omitted. It is recommended that descriptions be short paragraphs, with the first line providing a concise summary. The last line in the description should end with a period ( `.` ). |
| `replacement`  |String |The full name of the property that replaces this deprecated property. If there is no replacement for this property, it may be omitted. |

> Prior to Spring Boot 1.3, a single  `deprecated`  boolean attribute can be used instead of the  `deprecation`  element. This is still supported in a deprecated fashion and should no longer be used. If no reason and replacement are available, an empty  `deprecation`  object should be set.

Deprecation can also be specified declaratively in code by adding the  `@DeprecatedConfigurationProperty`  annotation to the getter exposing the deprecated property. For instance, assume that the  `app.acme.target`  property was confusing and was renamed to  `app.acme.name` . The following example shows how to handle that situation:

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

> There is no way to set a  `level` .  `warning`  is always assumed, since code is still handling the property.

The preceding code makes sure that the deprecated property still works (delegating to the  `name`  property behind the scenes). Once the  `getTarget`  and  `setTarget`  methods can be removed from your public API, the automatic deprecation hint in the metadata goes away as well. If you want to keep a hint, adding manual metadata with an  `error`  deprecation level ensures that users are still informed about that property. Doing so is particularly useful when a  `replacement`  is provided.

### B.1.3 Hint Attributes

The JSON object contained in the  `hints`  array can contain the attributes shown in the following table:

|Name|Type|Purpose|
|----|----|----|
| `name`  |String |The full name of the property to which this hint refers. Names are in lower-case period-separated form (such as  `spring.mvc.servlet.path` ). If the property refers to a map (such as  `system.contexts` ), the hint either applies to the keys of the map ( `system.context.keys` ) or the values ( `system.context.values` ) of the map. This attribute is mandatory. |
| `values`  |ValueHint[] |A list of valid values as defined by the  `ValueHint`  object (described in the next table). Each entry defines the value and may have a description. |
| `providers`  |ValueProvider[] |A list of providers as defined by the  `ValueProvider`  object (described later in this document). Each entry defines the name of the provider and its parameters, if any. |

The JSON object contained in the  `values`  attribute of each  `hint`  element can contain the attributes described in the following table:

|Name|Type|Purpose|
|----|----|----|
| `value`  |Object |A valid value for the element to which the hint refers. If the type of the property is an array, it can also be an array of value(s). This attribute is mandatory. |
| `description`  |String |A short description of the value that can be displayed to users. If no description is available, it may be omitted . It is recommended that descriptions be short paragraphs, with the first line providing a concise summary. The last line in the description should end with a period ( `.` ). |

The JSON object contained in the  `providers`  attribute of each  `hint`  element can contain the attributes described in the following table:

|Name|Type|Purpose|
|----|----|----|
| `name`  |String |The name of the provider to use to offer additional content assistance for the element to which the hint refers. |
| `parameters`  |JSON object |Any additional parameter that the provider supports (check the documentation of the provider for more details). |

### B.1.4 Repeated Metadata Items

Objects with the same “property” and “group” name can appear multiple times within a metadata file. For example, you could bind two separate classes to the same prefix, with each having potentially overlapping property names. While the same names appearing in the metadata multiple times should not be common, consumers of metadata should take care to ensure that they support it.

## B.2 Providing Manual Hints

To improve the user experience and further assist the user in configuring a given property, you can provide additional metadata that:

- Describes the list of potential values for a property.

- Associates a provider, to attach a well defined semantic to a property, so that a tool can discover the list of potential values based on the project’s context.

### B.2.1 Value Hint

The  `name`  attribute of each hint refers to the  `name`  of a property. In the [initial example shown earlier](configuration-metadata.html#configuration-metadata-format), we provide five values for the  `spring.jpa.hibernate.ddl-auto`  property:  `none` ,  `validate` ,  `update` ,  `create` , and  `create-drop` . Each value may have a description as well.

If your property is of type  `Map` , you can provide hints for both the keys and the values (but not for the map itself). The special  `.keys`  and  `.values`  suffixes must refer to the keys and the values, respectively.

Assume a  `sample.contexts`  maps magic  `String`  values to an integer, as shown in the following example:

```java
@ConfigurationProperties("sample")
public class SampleProperties {

	private Map<String,Integer> contexts;
	// getters and setters
}
```

The magic values are (in this example) are  `sample1`  and  `sample2` . In order to offer additional content assistance for the keys, you could add the following JSON to [the manual metadata of the module](configuration-metadata.html#configuration-metadata-additional-metadata):

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

> We recommend that you use an  `Enum`  for those two values instead. If your IDE supports it, this is by far the most effective approach to auto-completion.

### B.2.2 Value Providers

Providers are a powerful way to attach semantics to a property. In this section, we define the official providers that you can use for your own hints. However, your favorite IDE may implement some of these or none of them. Also, it could eventually provide its own.

> As this is a new feature, IDE vendors must catch up with how it works. Adoption times naturally vary.

The following table summarizes the list of supported providers:

|Name|Description|
|----|----|
| `any`  |Permits any additional value to be provided. |
| `class-reference`  |Auto-completes the classes available in the project. Usually constrained by a base class that is specified by the  `target`  parameter. |
| `handle-as`  |Handles the property as if it were defined by the type defined by the mandatory  `target`  parameter. |
| `logger-name`  |Auto-completes valid logger names and [logger groups](boot-features-logging.html#boot-features-custom-log-groups). Typically, package and class names available in the current project can be auto-completed as well as defined groups. |
| `spring-bean-reference`  |Auto-completes the available bean names in the current project. Usually constrained by a base class that is specified by the  `target`  parameter. |
| `spring-profile-name`  |Auto-completes the available Spring profile names in the project. |

> Only one provider can be active for a given property, but you can specify several providers if they can all manage the property in some way. Make sure to place the most powerful provider first, as the IDE must use the first one in the JSON section that it can handle. If no provider for a given property is supported, no special content assistance is provided, either.

#### Any

The special  **any**  provider value permits any additional values to be provided. Regular value validation based on the property type should be applied if this is supported.

This provider is typically used if you have a list of values and any extra values should still be considered as valid.

The following example offers  `on`  and  `off`  as auto-completion values for  `system.state` :

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

Note that, in the preceding example, any other value is also allowed.

#### Class Reference

The  **class-reference**  provider auto-completes classes available in the project. This provider supports the following parameters:

|Parameter|Type|Default value|Description|
|----|----|----|----|
| `target`  | `String`  ( `Class` ) |none |The fully qualified name of the class that should be assignable to the chosen value. Typically used to filter out-non candidate classes. Note that this information can be provided by the type itself by exposing a class with the appropriate upper bound. |
| `concrete`  | `boolean`  |true |Specify whether only concrete classes are to be considered as valid candidates. |

The following metadata snippet corresponds to the standard  `server.servlet.jsp.class-name`  property that defines the  `JspServlet`  class name to use:

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

The  **handle-as**  provider lets you substitute the type of the property to a more high-level type. This typically happens when the property has a  `java.lang.String`  type, because you do not want your configuration classes to rely on classes that may not be on the classpath. This provider supports the following parameters:

|Parameter|Type|Default value|Description|
|----|----|----|----|
| **target**  | `String`  ( `Class` ) |none |The fully qualified name of the type to consider for the property. This parameter is mandatory. |

The following types can be used:

- Any  `java.lang.Enum` : Lists the possible values for the property. (We recommend defining the property with the  `Enum`  type, as no further hint should be required for the IDE to auto-complete the values.)

-  `java.nio.charset.Charset` : Supports auto-completion of charset/encoding values (such as  `UTF-8` )

-  `java.util.Locale` : auto-completion of locales (such as  `en_US` )

-  `org.springframework.util.MimeType` : Supports auto-completion of content type values (such as  `text/plain` )

-  `org.springframework.core.io.Resource` : Supports auto-completion of Spring’s Resource abstraction to refer to a file on the filesystem or on the classpath. (such as  `classpath:/sample.properties` )

> If multiple values can be provided, use a  `Collection`  or Array type to teach the IDE about it.

The following metadata snippet corresponds to the standard  `spring.liquibase.change-log`  property that defines the path to the changelog to use. It is actually used internally as a  `org.springframework.core.io.Resource`  but cannot be exposed as such, because we need to keep the original String value to pass it to the Liquibase API.

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

#### Logger Name

The  **logger-name**  provider auto-completes valid logger names and [logger groups](boot-features-logging.html#boot-features-custom-log-groups). Typically, package and class names available in the current project can be auto-completed. If groups are enabled (default) and if a custom logger group is identified in the configuration, auto-completion for it should be provided. Specific frameworks may have extra magic logger names that can be supported as well.

This provider supports the following parameters:

|Parameter|Type|Default value|Description|
|----|----|----|----|
| `group`  | `boolean`  | `true`  |Specify whether known groups should be considered. |

Since a logger name can be any arbitrary name, this provider should allow any value but could highlight valid package and class names that are not available in the project’s classpath.

The following metadata snippet corresponds to the standard  `logging.level`  property. Keys are logger names, and values correspond to the standard log levels or any custom level. As Spring Boot defines a few logger groups out-of-the-box, dedicated value hints have been added for those.

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

#### Spring Bean Reference

The  **spring-bean-reference**  provider auto-completes the beans that are defined in the configuration of the current project. This provider supports the following parameters:

|Parameter|Type|Default value|Description|
|----|----|----|----|
| `target`  | `String`  ( `Class` ) |none |The fully qualified name of the bean class that should be assignable to the candidate. Typically used to filter out non-candidate beans. |

The following metadata snippet corresponds to the standard  `spring.jmx.server`  property that defines the name of the  `MBeanServer`  bean to use:

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

> The binder is not aware of the metadata. If you provide that hint, you still need to transform the bean name into an actual Bean reference using by the  `ApplicationContext` .

#### Spring Profile Name

The  **spring-profile-name**  provider auto-completes the Spring profiles that are defined in the configuration of the current project.

The following metadata snippet corresponds to the standard  `spring.profiles.active`  property that defines the name of the Spring profile(s) to enable:

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

## B.3 Generating Your Own Metadata by Using the Annotation Processor

You can easily generate your own configuration metadata file from items annotated with  `@ConfigurationProperties`  by using the  `spring-boot-configuration-processor`  jar. The jar includes a Java annotation processor which is invoked as your project is compiled. To use the processor, include a dependency on  `spring-boot-configuration-processor` .

With Maven the dependency should be declared as optional, as shown in the following example:

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-configuration-processor</artifactId>
	<optional>true</optional>
</dependency>
```

With Gradle 4.5 and earlier, the dependency should be declared in the  `compileOnly`  configuration, as shown in the following example:

```java
dependencies {
	compileOnly "org.springframework.boot:spring-boot-configuration-processor"
}
```

With Gradle 4.6 and later, the dependency should be declared in the  `annotationProcessor`  configuration, as shown in the following example:

```java
dependencies {
	annotationProcessor "org.springframework.boot:spring-boot-configuration-processor"
}
```

If you are using an  `additional-spring-configuration-metadata.json`  file, the  `compileJava`  task should be configured to depend on the  `processResources`  task, as shown in the following example:

```java
compileJava.dependsOn(processResources)
```

This dependency ensures that the additional metadata is available when the annotation processor runs during compilation.

The processor picks up both classes and methods that are annotated with  `@ConfigurationProperties` . The Javadoc for field values within configuration classes is used to populate the  `description`  attribute.

> You should only use simple text with  `@ConfigurationProperties`  field Javadoc, since they are not processed before being added to the JSON.

Properties are discovered through the presence of standard getters and setters with special handling for collection types (that is detected even if only a getter is present). The annotation processor also supports the use of the  `@Data` ,  `@Getter` , and  `@Setter`  lombok annotations.

> If you are using AspectJ in your project, you need to make sure that the annotation processor runs only once. There are several ways to do this. With Maven, you can configure the  `maven-apt-plugin`  explicitly and add the dependency to the annotation processor only there. You could also let the AspectJ plugin run all the processing and disable annotation processing in the  `maven-compiler-plugin`  configuration, as follows:

### B.3.1 Nested Properties

The annotation processor automatically considers inner classes as nested properties. Consider the following class:

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

The preceding example produces metadata information for  `server.name` ,  `server.host.ip` , and  `server.host.port`  properties. You can use the  `@NestedConfigurationProperty`  annotation on a field to indicate that a regular (non-inner) class should be treated as if it were nested.

> This has no effect on collections and maps, as those types are automatically identified, and a single metadata property is generated for each of them.

### B.3.2 Adding Additional Metadata

Spring Boot’s configuration file handling is quite flexible, and it is often the case that properties may exist that are not bound to a  `@ConfigurationProperties`  bean. You may also need to tune some attributes of an existing key. To support such cases and let you provide custom "hints", the annotation processor automatically merges items from  `META-INF/additional-spring-configuration-metadata.json`  into the main metadata file.

If you refer to a property that has been detected automatically, the description, default value, and deprecation information are overridden, if specified. If the manual property declaration is not identified in the current module, it is added as a new property.

The format of the  `additional-spring-configuration-metadata.json`  file is exactly the same as the regular  `spring-configuration-metadata.json` . The additional properties file is optional. If you do not have any additional properties, do not add the file.


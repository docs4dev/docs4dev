## 96.自定义

|图片/ important.png |重要|
| ---- | ---- |
|此部分仅适用于Groovy DSL |

您可以通过扩展DSL来自定义Spring Cloud Contract Verifier，如本节其余部分所示.

## 96.1扩展DSL

您可以为DSL提供自己的功能.此功能的关键要求是保持静态兼容性.在本文档的后面部分，您可以看到以下示例：

- 使用可重用的类创建JAR.

- 在DSL中引用这些类.

你可以找到完整的例子[here](https://github.com/spring-cloud-samples/spring-cloud-contract-samples).

### 96.1.1普通JAR

以下示例显示了可在DSL中重用的三个类.

**PatternUtils** 包含 **consumer** 和 **producer** 使用的函数.

```xml
package com.example;

import java.util.regex.Pattern;

/**
* If you want to use {@link Pattern} directly in your tests
* then you can create a class resembling this one. It can
* contain all the {@link Pattern} you want to use in the DSL.
*
* <pre>
* {@code
* request {
*     body(
*         [ age: $(c(PatternUtils.oldEnough()))]
*     )
* }
* </pre>
*
* Notice that we're using both {@code $()} for dynamic values
* and {@code c()} for the consumer side.
*
* @author Marcin Grzejszczak
*/
//tag::impl[]
public class PatternUtils {

	public static String tooYoung() {
		//remove::start[]
		return "[0-1][0-9]";
		//remove::end[return]
	}

	public static Pattern oldEnough() {
		//remove::start[]
		return Pattern.compile("[2-9][0-9]");
		//remove::end[return]
	}

	/**
	 * Makes little sense but it's just an example ;)
	 */
	public static Pattern ok() {
		//remove::start[]
		return Pattern.compile("OK");
		//remove::end[return]
	}
}
//end::impl[]
```

**ConsumerUtils** 包含 **consumer** 使用的函数.

```xml
package com.example;

import org.springframework.cloud.contract.spec.internal.ClientDslProperty;

/**
* DSL Properties passed to the DSL from the consumer's perspective.
* That means that on the input side {@code Request} for HTTP
* or {@code Input} for messaging you can have a regular expression.
* On the {@code Response} for HTTP or {@code Output} for messaging
* you have to have a concrete value.
*
* @author Marcin Grzejszczak
*/
//tag::impl[]
public class ConsumerUtils {
	/**
	 * Consumer side property. By using the {@link ClientDslProperty}
	 * you can omit most of boilerplate code from the perspective
	 * of dynamic values. Example
	 *
	 * <pre>
	 * {@code
	 * request {
	 *     body(
	 *         [ age: $(ConsumerUtils.oldEnough())]
	 *     )
	 * }
	 * </pre>
	 *
	 * That way it's in the implementation that we decide what value we will pass to the consumer
	 * and which one to the producer.
	 *
	 * @author Marcin Grzejszczak
	 */
	public static ClientDslProperty oldEnough() {
		//remove::start[]
		// this example is not the best one and
		// theoretically you could just pass the regex instead of `ServerDslProperty` but
		// it's just to show some new tricks :)
		return new ClientDslProperty(PatternUtils.oldEnough(), 40);
		//remove::end[return]
	}

}
//end::impl[]
```

**ProducerUtils** 包含 **producer** 使用的函数.

```xml
package com.example;

import org.springframework.cloud.contract.spec.internal.ServerDslProperty;

/**
* DSL Properties passed to the DSL from the producer's perspective.
* That means that on the input side {@code Request} for HTTP
* or {@code Input} for messaging you have to have a concrete value.
* On the {@code Response} for HTTP or {@code Output} for messaging
* you can have a regular expression.
*
* @author Marcin Grzejszczak
*/
//tag::impl[]
public class ProducerUtils {

	/**
	 * Producer side property. By using the {@link ProducerUtils}
	 * you can omit most of boilerplate code from the perspective
	 * of dynamic values. Example
	 *
	 * <pre>
	 * {@code
	 * response {
	 *     body(
	 *         [ status: $(ProducerUtils.ok())]
	 *     )
	 * }
	 * </pre>
	 *
	 * That way it's in the implementation that we decide what value we will pass to the consumer
	 * and which one to the producer.
	 */
	public static ServerDslProperty ok() {
		// this example is not the best one and
		// theoretically you could just pass the regex instead of `ServerDslProperty` but
		// it's just to show some new tricks :)
		return new ServerDslProperty( PatternUtils.ok(), "OK");
	}
}
//end::impl[]
```

### 96.1.2将依赖项添加到项目中

为了使插件和IDE能够引用公共JAR类，您需要将依赖项传递给项目.

### 96.1.3测试项目依赖关系中的依赖关系

首先，将公共jar依赖项添加为测试依赖项.由于您的Contract文件在测试资源路径中可用，因此常见的jar类会自动显示在Groovy文件中.以下示例显示如何测试依赖项：

**Maven.** 

```xml
<dependency>
	<groupId>com.example</groupId>
	<artifactId>beer-common</artifactId>
	<version>${project.version}</version>
	<scope>test</scope>
</dependency>
```

**Gradle.** 

```java
testCompile("com.example:beer-common:0.0.1-SNAPSHOT")
```

### 96.1.4测试插件依赖关系中的依赖关系

现在，您必须添加插件的依赖项以在运行时重用，如以下示例所示：

**Maven.** 

```xml
<plugin>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-contract-maven-plugin</artifactId>
	<version>${spring-cloud-contract.version}</version>
	<extensions>true</extensions>
	<configuration>
		<packageWithBaseClasses>com.example</packageWithBaseClasses>
		<baseClassMappings>
			<baseClassMapping>
				<contractPackageRegex>.*intoxication.*</contractPackageRegex>
				<baseClassFQN>com.example.intoxication.BeerIntoxicationBase</baseClassFQN>
			</baseClassMapping>
		</baseClassMappings>
	</configuration>
	<dependencies>
		<dependency>
			<groupId>com.example</groupId>
			<artifactId>beer-common</artifactId>
			<version>${project.version}</version>
			<scope>compile</scope>
		</dependency>
	</dependencies>
</plugin>
```

**Gradle.** 

```java
classpath "com.example:beer-common:0.0.1-SNAPSHOT"
```

### 96.1.5在DSL中引用类

您现在可以在DSL中引用您的类，如以下示例所示：

```java
package contracts.beer.rest

import com.example.ConsumerUtils
import com.example.ProducerUtils
import org.springframework.cloud.contract.spec.Contract

Contract.make {
	description("""
Represents a successful scenario of getting a beer

```
given:
	client is old enough
when:
	he applies for a beer
then:
	we'll grant him the beer
```

""")
	request {
		method 'POST'
		url '/check'
		body(
				age: $(ConsumerUtils.oldEnough())
		)
		headers {
			contentType(applicationJson())
		}
	}
	response {
		status 200
		body("""
			{
				"status": "${value(ProducerUtils.ok())}"
			}
			""")
		headers {
			contentType(applicationJson())
		}
	}
}
```


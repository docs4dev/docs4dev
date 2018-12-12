## 96. Customization

|images/important.png|Important|
|----|----|
|This section is valid only for Groovy DSL |

You can customize the Spring Cloud Contract Verifier by extending the DSL, as shown in the remainder of this section.

## 96.1 Extending the DSL

You can provide your own functions to the DSL. The key requirement for this feature is to maintain the static compatibility. Later in this document, you can see examples of:

- Creating a JAR with reusable classes.

- Referencing of these classes in the DSLs.

You can find the full example [here](https://github.com/spring-cloud-samples/spring-cloud-contract-samples).

### 96.1.1 Common JAR

The following examples show three classes that can be reused in the DSLs.

**PatternUtils**  contains functions used by both the  **consumer**  and the  **producer** .

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

**ConsumerUtils**  contains functions used by the  **consumer** .

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

**ProducerUtils**  contains functions used by the  **producer** .

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

### 96.1.2 Adding the Dependency to the Project

In order for the plugins and IDE to be able to reference the common JAR classes, you need to pass the dependency to your project.

### 96.1.3 Test the Dependency in the Project’s Dependencies

First, add the common jar dependency as a test dependency. Because your contracts files are available on the test resources path, the common jar classes automatically become visible in your Groovy files. The following examples show how to test the dependency:

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

### 96.1.4 Test a Dependency in the Plugin’s Dependencies

Now, you must add the dependency for the plugin to reuse at runtime, as shown in the following example:

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

### 96.1.5 Referencing classes in DSLs

You can now reference your classes in your DSL, as shown in the following example:

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


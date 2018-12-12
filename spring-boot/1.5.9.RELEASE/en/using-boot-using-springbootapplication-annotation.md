## 18. Using the @SpringBootApplication Annotation

Many Spring Boot developers like their apps to use auto-configuration, component scan and be able to define extra configuration on their "application class". A single  `@SpringBootApplication`  annotation can be used to enable those three features, that is:

-  `@EnableAutoConfiguration` : enable [Spring Boot’s auto-configuration mechanism](using-boot-auto-configuration.html)

-  `@ComponentScan` : enable  `@Component`  scan on the package where the application is located (see [the best practices](using-boot-structuring-your-code.html))

-  `@Configuration` : allow to register extra beans in the context or import additional configuration classes

The  `@SpringBootApplication`  annotation is equivalent to using  `@Configuration` ,  `@EnableAutoConfiguration` , and  `@ComponentScan`  with their default attributes, as shown in the following example:

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication // same as @Configuration @EnableAutoConfiguration @ComponentScan
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```

>  `@SpringBootApplication`  also provides aliases to customize the attributes of  `@EnableAutoConfiguration`  and  `@ComponentScan` .

> None of these features are mandatory and you may choose to replace this single annotation by any of the features that it enables. For instance, you may not want to use component scan in your application:


## 14. Structuring Your Code

Spring Boot does not require any specific code layout to work. However, there are some best practices that help.

## 14.1 Using the “default” Package

When a class does not include a  `package`  declaration, it is considered to be in the “default package”. The use of the “default package” is generally discouraged and should be avoided. It can cause particular problems for Spring Boot applications that use the  `@ComponentScan` ,  `@EntityScan` , or  `@SpringBootApplication`  annotations, since every class from every jar is read.

> We recommend that you follow Java’s recommended package naming conventions and use a reversed domain name (for example,  `com.example.project` ).

## 14.2 Locating the Main Application Class

We generally recommend that you locate your main application class in a root package above other classes. The [@SpringBootApplication annotation](using-boot-using-springbootapplication-annotation.html) is often placed on your main class, and it implicitly defines a base “search package” for certain items. For example, if you are writing a JPA application, the package of the  `@SpringBootApplication`  annotated class is used to search for  `@Entity`  items. Using a root package also allows component scan to apply only on your project.

> If you don’t want to use  `@SpringBootApplication` , the  `@EnableAutoConfiguration`  and  `@ComponentScan`  annotations that it imports defines that behaviour so you can also use that instead.

The following listing shows a typical layout:

```java
com
+- example
+- myapplication
+- Application.java
|
+- customer
|   +- Customer.java
|   +- CustomerController.java
|   +- CustomerService.java
|   +- CustomerRepository.java
|
+- order
+- Order.java
+- OrderController.java
+- OrderService.java
+- OrderRepository.java
```

The  `Application.java`  file would declare the  `main`  method, along with the basic  `@SpringBootApplication` , as follows:

```java
package com.example.myapplication;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class Application {

	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}

}
```


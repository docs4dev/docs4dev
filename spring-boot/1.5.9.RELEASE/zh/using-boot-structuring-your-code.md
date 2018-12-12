## 14.构建您的代码

Spring Boot不需要任何特定的代码布局.但是，有一些最佳实践可以提供帮助.

## 14.1使用“默认”包

当一个类不包含 `package` 声明时，它被认为是在“默认包”中.通常不鼓励使用“默认包”，应该避免使用.对于使用 `@ComponentScan` ， `@EntityScan` 或 `@SpringBootApplication` 注释的Spring Boot应用程序，它可能会导致特定问题，因为每个jar中的每个类都被读取.

> We建议您遵循Java推荐的程序包命名约定并使用反向域名（例如， `com.example.project` ）.

## 14.2找到主应用程序类

我们通常建议您将主应用程序类放在其他类之上的根包中. [@SpringBootApplication annotation](using-boot-using-springbootapplication-annotation.html)通常放在您的主类上，它隐含地为某些项定义了一个基础“搜索包”.例如，如果您正在编写JPA应用程序，则 `@SpringBootApplication`  annotated类的包用于搜索 `@Entity` 项.使用根包还允许组件扫描仅应用于您的项目.

> 如果您不想使用 `@SpringBootApplication` ，它导入的 `@EnableAutoConfiguration` 和 `@ComponentScan` 注释会定义该行为，因此您也可以使用它.

以下清单显示了典型的布局：

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

`Application.java` 文件将声明 `main` 方法以及基本 `@SpringBootApplication` ，如下所示：

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


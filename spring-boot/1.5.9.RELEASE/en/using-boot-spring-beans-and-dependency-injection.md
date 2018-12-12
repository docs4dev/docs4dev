## 17. Spring Beans and Dependency Injection

You are free to use any of the standard Spring Framework techniques to define your beans and their injected dependencies. For simplicity, we often find that using  `@ComponentScan`  (to find your beans) and using  `@Autowired`  (to do constructor injection) works well.

If you structure your code as suggested above (locating your application class in a root package), you can add  `@ComponentScan`  without any arguments. All of your application components ( `@Component` ,  `@Service` ,  `@Repository` ,  `@Controller`  etc.) are automatically registered as Spring Beans.

The following example shows a  `@Service`  Bean that uses constructor injection to obtain a required  `RiskAssessor`  bean:

```java
package com.example.service;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	@Autowired
	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```

If a bean has one constructor, you can omit the  `@Autowired` , as shown in the following example:

```java
@Service
public class DatabaseAccountService implements AccountService {

	private final RiskAssessor riskAssessor;

	public DatabaseAccountService(RiskAssessor riskAssessor) {
		this.riskAssessor = riskAssessor;
	}

	// ...

}
```

> Notice how using constructor injection lets the  `riskAssessor`  field be marked as  `final` , indicating that it cannot be subsequently changed.


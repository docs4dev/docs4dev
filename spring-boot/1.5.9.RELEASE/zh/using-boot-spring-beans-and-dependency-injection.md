## 17.spring beans和依赖注射

您可以自由地使用任何标准的Spring Framework技术来定义bean及其注入的依赖项.为简单起见，我们经常发现使用 `@ComponentScan` （找到你的bean）并使用 `@Autowired` （做构造函数注入）效果很好.

如果按照上面的建议构建代码（在根包中定位应用程序类），则可以添加 `@ComponentScan` 而不带任何参数.所有应用程序组件（ `@Component` ， `@Service` ， `@Repository` ， `@Controller` 等）都会自动注册为Spring Beans.

以下示例显示了 `@Service`  Bean，它使用构造函数注入来获取所需的 `RiskAssessor`  bean：

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

如果bean有一个构造函数，则可以省略 `@Autowired` ，如以下示例所示：

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

> 注意如何使用构造函数注入将 `riskAssessor` 字段标记为 `final` ，表示无法随后更改它.


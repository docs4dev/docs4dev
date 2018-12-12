## 68.使用Groovy Bean DSL开发应用程序

Spring Framework 4.0本机支持 `beans{}` “DSL”（从[Grails](http://grails.org/)借用），您可以使用相同的格式在Groovy应用程序脚本中嵌入bean定义.这有时是包含中间件声明等外部功能的好方法，如以下示例所示：

```java
@Configuration
class Application implements CommandLineRunner {

	@Autowired
	SharedService service

	@Override
	void run(String... args) {
		println service.message
	}

}

import my.company.SharedService

beans {
	service(SharedService) {
		message = "Hello World"
	}
}
```

您可以将类声明与 `beans{}` 混合在同一个文件中，只要它们保持在顶层，或者，如果您愿意，可以将bean DSL放在单独的文件中.

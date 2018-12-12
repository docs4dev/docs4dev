## 36.验证

只要JSR-303实现（例如Hibernate验证器）在类路径上，就会自动启用Bean Validation 1.1支持的方法验证功能.这允许bean方法在其参数和/或返回值上使用 `javax.validation` 约束进行注释.具有此类带注释方法的目标类需要在类型级别使用 `@Validated` 注释进行注释，以便搜索其内联约束注释的方法.

例如，以下服务触发第一个参数的验证，确保其大小在8到10之间：

```java
@Service
@Validated
public class MyBean {

	public Archive findByCodeAndAuthor(@Size(min = 8, max = 10) String code,
			Author author) {
		...
	}

}
```

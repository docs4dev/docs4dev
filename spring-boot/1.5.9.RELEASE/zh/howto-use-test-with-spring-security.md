## 80.测试使用Spring Security

Spring Security支持以特定用户身份运行测试.例如，下面代码段中的测试将与具有 `ADMIN` 角色的经过身份验证的用户一起运行.

```java
@Test
@WithMockUser(roles="ADMIN")
public void requestProtectedUrlWithUser() throws Exception {
	mvc
		.perform(get("/"))
		...
}
```

Spring Security提供了与Spring MVC Test的全面集成，这也可以在使用 `@WebMvcTest`  slice和 `MockMvc` 测试控制器时使用.

有关Spring Security的测试支持的其他详细信息，请参阅Spring Security的[reference documentation](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#test)）.

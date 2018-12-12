## 80. Testing With Spring Security

Spring Security provides support for running tests as a specific user. For example, the test in the snippet below will run with an authenticated user that has the  `ADMIN`  role.

```java
@Test
@WithMockUser(roles="ADMIN")
public void requestProtectedUrlWithUser() throws Exception {
	mvc
		.perform(get("/"))
		...
}
```

Spring Security provides comprehensive integration with Spring MVC Test and this can also be used when testing controllers using the  `@WebMvcTest`  slice and  `MockMvc` .

For additional details on Spring Security’s testing support, refer to Spring Security’s [reference documentation](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#test)).

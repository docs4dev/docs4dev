## 89.安全

本节介绍使用Spring Boot时的安全性问题，包括使用Spring Boot with Spring Boot时出现的问题.

有关Spring Security的更多信息，请参阅[Spring Security project page](https://projects.spring.io/spring-security/).

## 89.1关闭Spring Boot安全配置

如果在应用程序中使用 `WebSecurityConfigurerAdapter` 定义 `@Configuration` ，则会关闭Spring Boot中的默认Webapp安全设置.

## 89.2更改UserDetailsService并添加用户帐户

如果提供 `AuthenticationManager` ， `AuthenticationProvider` 或 `UserDetailsService` 类型的 `@Bean` ，则不会创建 `InMemoryUserDetailsManager` 的默认 `@Bean` ，因此您可以使用Spring Security的完整功能集（例如[various authentication options](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication)）.

添加用户帐户的最简单方法是提供自己的 `UserDetailsService`  bean.

## 89.3在代理服务器后运行时启用HTTPS

确保所有主要endpoints仅通过HTTPS可用是任何应用程序的重要工作.如果你使用Tomcat作为servlet容器，那么Spring Boot会自动添加Tomcat自己的 `RemoteIpValve` ，如果它检测到一些环境设置，你应该能够依赖 `HttpServletRequest` 来报告它是否安全（甚至是代理服务器的下游）处理真正的SSL终止）.标准行为由某些请求标头（ `x-forwarded-for` 和 `x-forwarded-proto` ）的存在与否决定，这些标头的名称是常规的，因此它应该适用于大多数前端代理.您可以通过向 `application.properties` 添加一些条目来打开阀门，如以下示例所示：

```java
server.tomcat.remote-ip-header=x-forwarded-for
server.tomcat.protocol-header=x-forwarded-proto
```

（其中任何一个属性的存在都会打开阀门.或者，您可以通过添加 `TomcatServletWebServerFactory`  bean来添加 `RemoteIpValve` .）

要将Spring Security配置为要求所有（或某些）请求的安全通道，请考虑添加自己的 `WebSecurityConfigurerAdapter` ，以添加以下 `HttpSecurity` 配置：

```java
@Configuration
public class SslWebSecurityConfigurerAdapter extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		// Customize the application security
		http.requiresChannel().anyRequest().requiresSecure();
	}

}
```


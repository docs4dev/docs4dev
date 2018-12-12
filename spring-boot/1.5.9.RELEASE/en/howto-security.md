## 89. Security

This section addresses questions about security when working with Spring Boot, including questions that arise from using Spring Security with Spring Boot.

For more about Spring Security, see the [Spring Security project page](https://projects.spring.io/spring-security/).

## 89.1 Switch off the Spring Boot Security Configuration

If you define a  `@Configuration`  with a  `WebSecurityConfigurerAdapter`  in your application, it switches off the default webapp security settings in Spring Boot.

## 89.2 Change the UserDetailsService and Add User Accounts

If you provide a  `@Bean`  of type  `AuthenticationManager` ,  `AuthenticationProvider` , or  `UserDetailsService` , the default  `@Bean`  for  `InMemoryUserDetailsManager`  is not created, so you have the full feature set of Spring Security available (such as [various authentication options](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#jc-authentication)).

The easiest way to add user accounts is to provide your own  `UserDetailsService`  bean.

## 89.3 Enable HTTPS When Running behind a Proxy Server

Ensuring that all your main endpoints are only available over HTTPS is an important chore for any application. If you use Tomcat as a servlet container, then Spring Boot adds Tomcat’s own  `RemoteIpValve`  automatically if it detects some environment settings, and you should be able to rely on the  `HttpServletRequest`  to report whether it is secure or not (even downstream of a proxy server that handles the real SSL termination). The standard behavior is determined by the presence or absence of certain request headers ( `x-forwarded-for`  and  `x-forwarded-proto` ), whose names are conventional, so it should work with most front-end proxies. You can switch on the valve by adding some entries to  `application.properties` , as shown in the following example:

```java
server.tomcat.remote-ip-header=x-forwarded-for
server.tomcat.protocol-header=x-forwarded-proto
```

(The presence of either of those properties switches on the valve. Alternatively, you can add the  `RemoteIpValve`  by adding a  `TomcatServletWebServerFactory`  bean.)

To configure Spring Security to require a secure channel for all (or some) requests, consider adding your own  `WebSecurityConfigurerAdapter`  that adds the following  `HttpSecurity`  configuration:

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


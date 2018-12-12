## 29.安全

如果[Spring Security](https://projects.spring.io/spring-security/)在类路径上，则默认情况下Web应用程序是安全的. Spring Boot依赖于Spring Security的内容协商策略来确定是使用 `httpBasic` 还是 `formLogin` .要向Web应用程序添加方法级安全性，还可以使用所需设置添加 `@EnableGlobalMethodSecurity` .其他信息可在[Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/htmlsingle#jc-method)中找到.

默认 `UserDetailsService` 只有一个用户.用户名是 `user` ，密码是随机的，在应用程序启动时以INFO级别打印，如以下示例所示：

```java
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

> 如果您对日志记录配置进行微调，请确保将 `org.springframework.boot.autoconfigure.security` 类别设置为记录 `INFO` 级别的消息.否则，不会打印默认密码.

您可以通过提供 `spring.security.user.name` 和 `spring.security.user.password` 来更改用户名和密码.

您在Web应用程序中默认获得的基本功能包括：

- A  `UserDetailsService` （如果是WebFlux应用程序，则为 `ReactiveUserDetailsService` ）具有内存存储的bean和具有生成密码的单个用户（有关用户属性，请参阅[SecurityProperties.User](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/autoconfigure/security/SecurityProperties.User.html)）.
基于
- Form的登录或整个应用程序的HTTP基本安全性（取决于Content-Type）（如果Actuator在类路径上，则包括Actuatorendpoints）.

- A  `DefaultAuthenticationEventPublisher` 用于发布身份验证事件.

您可以通过为其添加bean来提供不同的 `AuthenticationEventPublisher` .

## 29.1 MVC安全

默认安全配置在 `SecurityAutoConfiguration` 和 `UserDetailsServiceAutoConfiguration` 中实现.  `SecurityAutoConfiguration` 导入 `SpringBootWebSecurityConfiguration` 用于Web安全性和 `UserDetailsServiceAutoConfiguration` 配置身份验证，这在非Web应用程序中也很相关.要完全关闭默认Web应用程序安全性配置，可以添加 `WebSecurityConfigurerAdapter` 类型的bean（这样做不会禁用 `UserDetailsService` 配置或Actuator的安全性）.

要同时关闭 `UserDetailsService` 配置，可以添加 `UserDetailsService` ， `AuthenticationProvider` 或 `AuthenticationManager` 类型的bean. [Spring Boot samples](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/)中有几个安全应用程序可以帮助您开始使用常见用例.

可以通过添加自定义 `WebSecurityConfigurerAdapter` 来覆盖访问规则. Spring Boot提供了便捷方法，可用于覆盖Actuatorendpoints和静态资源的访问规则.  `EndpointRequest` 可用于创建基于 `management.endpoints.web.base-path` 属性的 `RequestMatcher` .  `PathRequest` 可用于为常用位置中的资源创建 `RequestMatcher` .

## 29.2 WebFlux安全

与Spring MVC应用程序类似，您可以通过添加 `spring-boot-starter-security` 依赖项来保护您的WebFlux应用程序.默认安全配置在 `ReactiveSecurityAutoConfiguration` 和 `UserDetailsServiceAutoConfiguration` 中实现.  `ReactiveSecurityAutoConfiguration` 导入 `WebFluxSecurityConfiguration` 用于Web安全性和 `UserDetailsServiceAutoConfiguration` 配置身份验证，这在非Web应用程序中也很相关.要完全关闭默认Web应用程序安全性配置，可以添加 `WebFilterChainProxy` 类型的bean（这样做不会禁用 `UserDetailsService` 配置或Actuator的安全性）.

要关闭 `UserDetailsService` 配置，可以添加 `ReactiveUserDetailsService` 或 `ReactiveAuthenticationManager` 类型的bean.

可以通过添加自定义 `SecurityWebFilterChain` 来配置访问规则. Spring Boot提供了便捷方法，可用于覆盖Actuatorendpoints和静态资源的访问规则.  `EndpointRequest` 可用于创建基于 `management.endpoints.web.base-path` 属性的 `ServerWebExchangeMatcher` .

`PathRequest` 可用于为常用位置的资源创建 `ServerWebExchangeMatcher` .

例如，您可以通过添加以下内容来自定义安全配置：

```java
@Bean
public SecurityWebFilterChain springSecurityFilterChain(ServerHttpSecurity http) {
	return http
		.authorizeExchange()
			.matchers(PathRequest.toStaticResources().atCommonLocations()).permitAll()
			.pathMatchers("/foo", "/bar")
				.authenticated().and()
			.formLogin().and()
		.build();
}
```

## 29.3 OAuth2

[OAuth2](https://oauth.net/2/)是Spring支持的一种广泛使用的授权框架.

### 29.3.1客户

如果您的类路径上有 `spring-security-oauth2-client` ，则可以利用某些自动配置来轻松设置OAuth2 / Open ID Connect客户端.此配置使用 `OAuth2ClientProperties` 下的属性.相同的属性适用于servlet和反应应用程序.

您可以在 `spring.security.oauth2.client` 前缀下注册多个OAuth2客户端和提供程序，如以下示例所示：

```java
spring.security.oauth2.client.registration.my-client-1.client-id=abcd
spring.security.oauth2.client.registration.my-client-1.client-secret=password
spring.security.oauth2.client.registration.my-client-1.client-name=Client for user scope
spring.security.oauth2.client.registration.my-client-1.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-1.scope=user
spring.security.oauth2.client.registration.my-client-1.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-1.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-1.authorization-grant-type=authorization_code

spring.security.oauth2.client.registration.my-client-2.client-id=abcd
spring.security.oauth2.client.registration.my-client-2.client-secret=password
spring.security.oauth2.client.registration.my-client-2.client-name=Client for email scope
spring.security.oauth2.client.registration.my-client-2.provider=my-oauth-provider
spring.security.oauth2.client.registration.my-client-2.scope=email
spring.security.oauth2.client.registration.my-client-2.redirect-uri-template=http://my-redirect-uri.com
spring.security.oauth2.client.registration.my-client-2.client-authentication-method=basic
spring.security.oauth2.client.registration.my-client-2.authorization-grant-type=authorization_code

spring.security.oauth2.client.provider.my-oauth-provider.authorization-uri=http://my-auth-server/oauth/authorize
spring.security.oauth2.client.provider.my-oauth-provider.token-uri=http://my-auth-server/oauth/token
spring.security.oauth2.client.provider.my-oauth-provider.user-info-uri=http://my-auth-server/userinfo
spring.security.oauth2.client.provider.my-oauth-provider.user-info-authentication-method=header
spring.security.oauth2.client.provider.my-oauth-provider.jwk-set-uri=http://my-auth-server/token_keys
spring.security.oauth2.client.provider.my-oauth-provider.user-name-attribute=name
```

对于支持[OpenID Connect discovery](https://openid.net/specs/openid-connect-discovery-1_0.html)的OpenID Connect提供程序，可以进一步简化配置.提供程序需要配置 `issuer-uri` ，它是它声明为其颁发者标识符的URI.例如，如果 `issuer-uri` 提供的是"https://example.com"，则 `OpenID Provider Configuration Request` 将被设置为"https://example.com/.well-known/openid-configuration".结果预计是 `OpenID Provider Configuration Response` .以下示例显示如何使用 `issuer-uri` 配置OpenID Connect Provider：

```java
spring.security.oauth2.client.provider.oidc-provider.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

默认情况下，Spring Security的 `OAuth2LoginAuthenticationFilter` 仅处理与 `/login/oauth2/code/*` 匹配的URL.如果要自定义 `redirect-uri` 以使用其他模式，则需要提供配置以处理该自定义模式.例如，对于servlet应用程序，您可以添加类似于以下内容的 `WebSecurityConfigurerAdapter` ：

```java
public class OAuth2LoginSecurityConfig extends WebSecurityConfigurerAdapter {

	@Override
	protected void configure(HttpSecurity http) throws Exception {
		http
			.authorizeRequests()
				.anyRequest().authenticated()
				.and()
			.oauth2Login()
				.redirectionEndpoint()
					.baseUri("/custom-callback");
	}
}
```

#### OAuth2常用提供商的客户注册

对于常见的OAuth2和OpenID提供商，包括Google，Github，Facebook和Okta，我们提供了一组提供程序默认值（分别为 `google` ， `github` ， `facebook` 和 `okta` ）.

如果您不需要自定义这些提供程序，则可以将 `provider` 属性设置为需要推断默认值的属性.此外，如果客户端注册的密钥与默认支持的提供程序匹配，则Spring Boot也会推断出.

换句话说，以下示例中的两个配置使用Google提供程序：

```java
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```

### 29.3.2资源服务器

如果您的类路径上有 `spring-security-oauth2-resource-server` ，只要指定了JWK集URI或OIDC颁发者URI，Spring Boot就可以设置OAuth2资源服务器，如以下示例所示：

```java
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://example.com/oauth2/default/v1/keys
```

```java
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

相同的属性适用于servlet和反应应用程序.

或者，您可以为servlet应用程序定义自己的 `JwtDecoder`  bean，或者为响应式应用程序定义 `ReactiveJwtDecoder` .

### 29.3.3授权服务器

目前，Spring Security不支持实施OAuth 2.0授权服务器.但是，此功能可从[Spring Security OAuth](https://projects.spring.io/spring-security-oauth/)项目获得，该项目最终将完全被Spring Security取代.在此之前，您可以使用 `spring-security-oauth2-autoconfigure` 模块轻松设置OAuth 2.0授权服务器;请参阅[documentation](https://docs.spring.io/spring-security-oauth2-boot)获取说明.

## 29.4Actuator安全性

出于安全考虑，默认情况下禁用 `/health` 和 `/info` 以外的所有Actuator.  `management.endpoints.web.exposure.include` 属性可用于启用Actuator.

如果Spring Security位于类路径上且没有其他WebSecurityConfigurerAdapter，则 `/health` 和 `/info` 以外的所有Actuator都由Spring Boot自动配置保护.如果您定义了自定义 `WebSecurityConfigurerAdapter` ，Spring Boot自动配置将退回，您将完全控制Actuator访问规则.

> B在设置 `management.endpoints.web.exposure.include` 之前，确保暴露的Actuator不包含敏感信息和/或通过将它们放在防火墙后面或通过Spring Security之类的东西来保护.

### 29.4.1跨站点请求伪造保护

由于Spring Boot依赖于Spring Security的默认值，因此默认情况下会启用CSRF保护.这意味着当使用默认安全配置时，需要 `POST` （关闭和Loggerendpoints）， `PUT` 或 `DELETE` 的Actuatorendpoints将获得403禁止错误.

> We建议仅在创建非浏览器客户端使用的服务时完全禁用CSRF保护.

有关CSRF保护的更多信息，请参见[Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/htmlsingle#csrf).


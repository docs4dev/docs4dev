## 29. Security

If [Spring Security](https://projects.spring.io/spring-security/) is on the classpath, then web applications are secured by default. Spring Boot relies on Spring Security’s content-negotiation strategy to determine whether to use  `httpBasic`  or  `formLogin` . To add method-level security to a web application, you can also add  `@EnableGlobalMethodSecurity`  with your desired settings. Additional information can be found in the [Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/htmlsingle#jc-method).

The default  `UserDetailsService`  has a single user. The user name is  `user` , and the password is random and is printed at INFO level when the application starts, as shown in the following example:

```java
Using generated security password: 78fa095d-3f4c-48b1-ad50-e24c31d5cf35
```

> If you fine-tune your logging configuration, ensure that the  `org.springframework.boot.autoconfigure.security`  category is set to log  `INFO` -level messages. Otherwise, the default password is not printed.

You can change the username and password by providing a  `spring.security.user.name`  and  `spring.security.user.password` .

The basic features you get by default in a web application are:

- A  `UserDetailsService`  (or  `ReactiveUserDetailsService`  in case of a WebFlux application) bean with in-memory store and a single user with a generated password (see [SecurityProperties.User](https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/api/org/springframework/boot/autoconfigure/security/SecurityProperties.User.html) for the properties of the user).

- Form-based login or HTTP Basic security (depending on Content-Type) for the entire application (including actuator endpoints if actuator is on the classpath).

- A  `DefaultAuthenticationEventPublisher`  for publishing authentication events.

You can provide a different  `AuthenticationEventPublisher`  by adding a bean for it.

## 29.1 MVC Security

The default security configuration is implemented in  `SecurityAutoConfiguration`  and  `UserDetailsServiceAutoConfiguration` .  `SecurityAutoConfiguration`  imports  `SpringBootWebSecurityConfiguration`  for web security and  `UserDetailsServiceAutoConfiguration`  configures authentication, which is also relevant in non-web applications. To switch off the default web application security configuration completely, you can add a bean of type  `WebSecurityConfigurerAdapter`  (doing so does not disable the  `UserDetailsService`  configuration or Actuator’s security).

To also switch off the  `UserDetailsService`  configuration, you can add a bean of type  `UserDetailsService` ,  `AuthenticationProvider` , or  `AuthenticationManager` . There are several secure applications in the [Spring Boot samples](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-samples/) to get you started with common use cases.

Access rules can be overridden by adding a custom  `WebSecurityConfigurerAdapter` . Spring Boot provides convenience methods that can be used to override access rules for actuator endpoints and static resources.  `EndpointRequest`  can be used to create a  `RequestMatcher`  that is based on the  `management.endpoints.web.base-path`  property.  `PathRequest`  can be used to create a  `RequestMatcher`  for resources in commonly used locations.

## 29.2 WebFlux Security

Similar to Spring MVC applications, you can secure your WebFlux applications by adding the  `spring-boot-starter-security`  dependency. The default security configuration is implemented in  `ReactiveSecurityAutoConfiguration`  and  `UserDetailsServiceAutoConfiguration` .  `ReactiveSecurityAutoConfiguration`  imports  `WebFluxSecurityConfiguration`  for web security and  `UserDetailsServiceAutoConfiguration`  configures authentication, which is also relevant in non-web applications. To switch off the default web application security configuration completely, you can add a bean of type  `WebFilterChainProxy`  (doing so does not disable the  `UserDetailsService`  configuration or Actuator’s security).

To also switch off the  `UserDetailsService`  configuration, you can add a bean of type  `ReactiveUserDetailsService`  or  `ReactiveAuthenticationManager` .

Access rules can be configured by adding a custom  `SecurityWebFilterChain` . Spring Boot provides convenience methods that can be used to override access rules for actuator endpoints and static resources.  `EndpointRequest`  can be used to create a  `ServerWebExchangeMatcher`  that is based on the  `management.endpoints.web.base-path`  property.

`PathRequest`  can be used to create a  `ServerWebExchangeMatcher`  for resources in commonly used locations.

For example, you can customize your security configuration by adding something like:

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

[OAuth2](https://oauth.net/2/) is a widely used authorization framework that is supported by Spring.

### 29.3.1 Client

If you have  `spring-security-oauth2-client`  on your classpath, you can take advantage of some auto-configuration to make it easy to set up an OAuth2/Open ID Connect clients. This configuration makes use of the properties under  `OAuth2ClientProperties` . The same properties are applicable to both servlet and reactive applications.

You can register multiple OAuth2 clients and providers under the  `spring.security.oauth2.client`  prefix, as shown in the following example:

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

For OpenID Connect providers that support [OpenID Connect discovery](https://openid.net/specs/openid-connect-discovery-1_0.html), the configuration can be further simplified. The provider needs to be configured with an  `issuer-uri`  which is the URI that the it asserts as its Issuer Identifier. For example, if the  `issuer-uri`  provided is "https://example.com", then an  `OpenID Provider Configuration Request`  will be made to "https://example.com/.well-known/openid-configuration". The result is expected to be an  `OpenID Provider Configuration Response` . The following example shows how an OpenID Connect Provider can be configured with the  `issuer-uri` :

```java
spring.security.oauth2.client.provider.oidc-provider.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

By default, Spring Security’s  `OAuth2LoginAuthenticationFilter`  only processes URLs matching  `/login/oauth2/code/*` . If you want to customize the  `redirect-uri`  to use a different pattern, you need to provide configuration to process that custom pattern. For example, for servlet applications, you can add your own  `WebSecurityConfigurerAdapter`  that resembles the following:

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

#### OAuth2 client registration for common providers

For common OAuth2 and OpenID providers, including Google, Github, Facebook, and Okta, we provide a set of provider defaults ( `google` ,  `github` ,  `facebook` , and  `okta` , respectively).

If you do not need to customize these providers, you can set the  `provider`  attribute to the one for which you need to infer defaults. Also, if the key for the client registration matches a default supported provider, Spring Boot infers that as well.

In other words, the two configurations in the following example use the Google provider:

```java
spring.security.oauth2.client.registration.my-client.client-id=abcd
spring.security.oauth2.client.registration.my-client.client-secret=password
spring.security.oauth2.client.registration.my-client.provider=google

spring.security.oauth2.client.registration.google.client-id=abcd
spring.security.oauth2.client.registration.google.client-secret=password
```

### 29.3.2 Resource Server

If you have  `spring-security-oauth2-resource-server`  on your classpath, Spring Boot can set up an OAuth2 Resource Server as long as a JWK Set URI or OIDC Issuer URI is specified, as shown in the following examples:

```java
spring.security.oauth2.resourceserver.jwt.jwk-set-uri=https://example.com/oauth2/default/v1/keys
```

```java
spring.security.oauth2.resourceserver.jwt.issuer-uri=https://dev-123456.oktapreview.com/oauth2/default/
```

The same properties are applicable for both servlet and reactive applications.

Alternatively, you can define your own  `JwtDecoder`  bean for servlet applications or a  `ReactiveJwtDecoder`  for reactive applications.

### 29.3.3 Authorization Server

Currently, Spring Security does not provide support for implementing an OAuth 2.0 Authorization Server. However, this functionality is available from the [Spring Security OAuth](https://projects.spring.io/spring-security-oauth/) project, which will eventually be superseded by Spring Security completely. Until then, you can use the  `spring-security-oauth2-autoconfigure`  module to easily set up an OAuth 2.0 authorization server; see its [documentation](https://docs.spring.io/spring-security-oauth2-boot) for instructions.

## 29.4 Actuator Security

For security purposes, all actuators other than  `/health`  and  `/info`  are disabled by default. The  `management.endpoints.web.exposure.include`  property can be used to enable the actuators.

If Spring Security is on the classpath and no other WebSecurityConfigurerAdapter is present, all actuators other than  `/health`  and  `/info`  are secured by Spring Boot auto-configuration. If you define a custom  `WebSecurityConfigurerAdapter` , Spring Boot auto-configuration will back off and you will be in full control of actuator access rules.

> Before setting the  `management.endpoints.web.exposure.include` , ensure that the exposed actuators do not contain sensitive information and/or are secured by placing them behind a firewall or by something like Spring Security.

### 29.4.1 Cross Site Request Forgery Protection

Since Spring Boot relies on Spring Security’s defaults, CSRF protection is turned on by default. This means that the actuator endpoints that require a  `POST`  (shutdown and loggers endpoints),  `PUT`  or  `DELETE`  will get a 403 forbidden error when the default security configuration is in use.

> We recommend disabling CSRF protection completely only if you are creating a service that is used by non-browser clients.

Additional information about CSRF protection can be found in the [Spring Security Reference Guide](https://docs.spring.io/spring-security/site/docs/5.1.1.RELEASE/reference/htmlsingle#csrf).


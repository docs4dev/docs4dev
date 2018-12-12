## 84. More Detail

## 84.1 Single Sign On

> All of the OAuth2 SSO and resource server features moved to Spring Boot in version 1.3. You can find documentation in the [Spring Boot user guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/).

## 84.2 Token Relay

A Token Relay is where an OAuth2 consumer acts as a Client and forwards the incoming token to outgoing resource requests. The consumer can be a pure Client (like an SSO application) or a Resource Server.

### 84.2.1 Client Token Relay

If your app is a user facing OAuth2 client (i.e. has declared  `@EnableOAuth2Sso`  or  `@EnableOAuth2Client` ) then it has an  `OAuth2ClientContext`  in request scope from Spring Boot. You can create your own  `OAuth2RestTemplate`  from this context and an autowired  `OAuth2ProtectedResourceDetails` , and then the context will always forward the access token downstream, also refreshing the access token automatically if it expires. (These are features of Spring Security and Spring Boot.)

> Spring Boot (1.4.1) does not create an  `OAuth2ProtectedResourceDetails`  automatically if you are using  `client_credentials`  tokens. In that case you need to create your own  `ClientCredentialsResourceDetails`  and configure it with  `@ConfigurationProperties("security.oauth2.client")` .

### 84.2.2 Client Token Relay in Zuul Proxy

If your app also has a [Spring Cloud Zuul](https://cloud.spring.io/spring-cloud.html#netflix-zuul-reverse-proxy) embedded reverse proxy (using  `@EnableZuulProxy` ) then you can ask it to forward OAuth2 access tokens downstream to the services it is proxying. Thus the SSO app above can be enhanced simply like this:

**app.groovy.**  

```java
@Controller
@EnableOAuth2Sso
@EnableZuulProxy
class Application {

}
```

and it will (in addition to logging the user in and grabbing a token) pass the authentication token downstream to the  `/proxy/*`  services. If those services are implemented with  `@EnableResourceServer`  then they will get a valid token in the correct header.

How does it work? The  `@EnableOAuth2Sso`  annotation pulls in  `spring-cloud-starter-security`  (which you could do manually in a traditional app), and that in turn triggers some autoconfiguration for a  `ZuulFilter` , which itself is activated because Zuul is on the classpath (via  `@EnableZuulProxy` ). The [filter](https://github.com/spring-cloud/spring-cloud-security/tree/master/src/main/java/org/springframework/cloud/security/oauth2/proxy/OAuth2TokenRelayFilter.java) just extracts an access token from the currently authenticated user, and puts it in a request header for the downstream requests.

### 84.2.3 Resource Server Token Relay

If your app has  `@EnableResourceServer`  you might want to relay the incoming token downstream to other services. If you use a  `RestTemplate`  to contact the downstream services then this is just a matter of how to create the template with the right context.

If your service uses  `UserInfoTokenServices`  to authenticate incoming tokens (i.e. it is using the  `security.oauth2.user-info-uri`  configuration), then you can simply create an  `OAuth2RestTemplate`  using an autowired  `OAuth2ClientContext`  (it will be populated by the authentication process before it hits the backend code). Equivalently (with Spring Boot 1.4), you could inject a  `UserInfoRestTemplateFactory`  and grab its  `OAuth2RestTemplate`  in your configuration. For example:

**MyConfiguration.java.**  

```java
@Bean
public OAuth2RestTemplate restTemplate(UserInfoRestTemplateFactory factory) {
return factory.getUserInfoRestTemplate();
}
```

This rest template will then have the same  `OAuth2ClientContext`  (request-scoped) that is used by the authentication filter, so you can use it to send requests with the same access token.

If your app is not using  `UserInfoTokenServices`  but is still a client (i.e. it declares  `@EnableOAuth2Client`  or  `@EnableOAuth2Sso` ), then with Spring Security Cloud any  `OAuth2RestOperations`  that the user creates from an  `@Autowired`   `@OAuth2Context`  will also forward tokens. This feature is implemented by default as an MVC handler interceptor, so it only works in Spring MVC. If you are not using MVC you could use a custom filter or AOP interceptor wrapping an  `AccessTokenContextRelay`  to provide the same feature.

Here’s a basic example showing the use of an autowired rest template created elsewhere ("foo.com" is a Resource Server accepting the same tokens as the surrounding app):

**MyController.java.**  

```java
@Autowired
private OAuth2RestOperations restTemplate;

@RequestMapping("/relay")
public String relay() {
ResponseEntity<String> response =
restTemplate.getForEntity("https://foo.com/bar", String.class);
return "Success! (" + response.getBody() + ")";
}
```

If you don’t want to forward tokens (and that is a valid choice, since you might want to act as yourself, rather than the client that sent you the token), then you only need to create your own  `OAuth2Context`  instead of autowiring the default one.

Feign clients will also pick up an interceptor that uses the  `OAuth2ClientContext`  if it is available, so they should also do a token relay anywhere where a  `RestTemplate`  would.


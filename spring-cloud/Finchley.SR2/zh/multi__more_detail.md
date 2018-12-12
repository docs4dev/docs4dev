## 84.更多细节

## 84.1单点登录

> 所有OAuth2 SSO和资源服务器功能都在1.3版本中移至Spring Boot.您可以在[Spring Boot user guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)中找到文档.

## 84.2令牌中继

令牌中继是OAuth2使用者充当客户端并将传入令牌转发到传出资源请求的位置.使用者可以是纯客户端（如SSO应用程序）或资源服务器.

### 84.2.1客户端令牌中继

如果您的应用是面向OAuth2客户端的用户（即已声明 `@EnableOAuth2Sso` 或 `@EnableOAuth2Client` ），那么它在Spring Boot的请求范围内具有 `OAuth2ClientContext` .您可以从此上下文创建自己的 `OAuth2RestTemplate` 并使用自动装配的 `OAuth2ProtectedResourceDetails` ，然后上下文将始终将访问令牌转发到下游，并在访问令牌过期时自动刷新访问令牌. （这些是Spring Security和Spring Boot的功能.）

如果使用 `client_credentials` 令牌，则
> Spring Boot（1.4.1）不会自动创建 `OAuth2ProtectedResourceDetails` .在这种情况下，您需要创建自己的 `ClientCredentialsResourceDetails` 并使用 `@ConfigurationProperties("security.oauth2.client")` 进行配置.

### 84.2.2 Zuul代理中的客户端令牌中继

如果您的应用程序还具有[Spring Cloud Zuul](https://cloud.spring.io/spring-cloud.html#netflix-zuul-reverse-proxy)嵌入式反向代理（使用 `@EnableZuulProxy` ），那么您可以要求它将OAuth2访问令牌下游转发到它所代理的服务.因此，上面的SSO应用程序可以简单地增强，如下所示：

**app.groovy.** 

```java
@Controller
@EnableOAuth2Sso
@EnableZuulProxy
class Application {

}
```

并且它（除了记录用户并抓取令牌之外）将认证令牌传递到 `/proxy/*` 服务的下游.如果这些服务是使用 `@EnableResourceServer` 实现的，那么它们将在正确的标头中获得有效的标记.

它是如何工作的？  `@EnableOAuth2Sso` 注释拉入 `spring-cloud-starter-security` （您可以在传统应用程序中手动执行），而这反过来触发一些 `ZuulFilter` 的自动配置，它本身被激活，因为Zuul在类路径上（通过 `@EnableZuulProxy` ）. [filter](https://github.com/spring-cloud/spring-cloud-security/tree/master/src/main/java/org/springframework/cloud/security/oauth2/proxy/OAuth2TokenRelayFilter.java)只是从当前经过身份验证的用户中提取访问令牌，并将其放入下游请求的请求标头中.

### 84.2.3资源服务器令牌中继

如果您的应用程序具有 `@EnableResourceServer` ，您可能希望将传入令牌中继到其他服务.如果您使用 `RestTemplate` 联系下游服务，那么这只是如何使用正确的上下文创建模板的问题.

如果您的服务使用 `UserInfoTokenServices` 来验证传入的令牌（即它使用 `security.oauth2.user-info-uri` 配置），那么您只需使用自动装配的 `OAuth2ClientContext` 创建 `OAuth2RestTemplate` （它将在它到达后端代码之前由身份验证过程填充）.等效地（使用Spring Boot 1.4），您可以注入 `UserInfoRestTemplateFactory` 并在配置中获取 `OAuth2RestTemplate` .例如：

**MyConfiguration.java.** 

```java
@Bean
public OAuth2RestTemplate restTemplate(UserInfoRestTemplateFactory factory) {
return factory.getUserInfoRestTemplate();
}
```

然后，此休息模板将具有与身份验证筛选器使用的相同的 `OAuth2ClientContext` （请求范围），因此您可以使用它来发送具有相同访问令牌的请求.

如果您的应用程序未使用 `UserInfoTokenServices` 但仍然是客户端（即它声明 `@EnableOAuth2Client` 或 `@EnableOAuth2Sso` ），那么使用Spring Security Cloud，用户从 `@Autowired`   `@OAuth2Context` 创建的任何 `OAuth2RestOperations` 也将转发令牌.默认情况下，此功能实现为MVC处理程序拦截器，因此它仅适用于Spring MVC.如果您不使用MVC，则可以使用自定义过滤器或包装 `AccessTokenContextRelay` 的AOP拦截器来提供相同的功能.

这是一个基本示例，显示了在其他地方创建的自动装配的休息模板的使用（“foo.com”是一个资源服务器接受与周围应用程序相同的标记）：

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

如果您不想转发令牌（这是一个有效的选择，因为您可能想要自己行动，而不是向您发送令牌的客户端），那么您只需创建自己的 `OAuth2Context` 而不是自动装配默认一个.

假设客户端也会选择使用 `OAuth2ClientContext` 的拦截器（如果可用），因此他们也应该在 `RestTemplate` 所在的任何地方进行令牌转发.


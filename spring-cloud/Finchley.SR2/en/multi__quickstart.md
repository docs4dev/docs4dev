## 83. Quickstart

## 83.1 OAuth2 Single Sign On

Here’s a Spring Cloud "Hello World" app with HTTP Basic authentication and a single user account:

**app.groovy.**  

```java
@Grab('spring-boot-starter-security')
@Controller
class Application {

@RequestMapping('/')
String home() {
'Hello World'
}

}
```

You can run it with  `spring run app.groovy`  and watch the logs for the password (username is "user"). So far this is just the default for a Spring Boot app.

Here’s a Spring Cloud app with OAuth2 SSO:

**app.groovy.**  

```java
@Controller
@EnableOAuth2Sso
class Application {

@RequestMapping('/')
String home() {
'Hello World'
}

}
```

Spot the difference? This app will actually behave exactly the same as the previous one, because it doesn’t know it’s OAuth2 credentals yet.

You can register an app in github quite easily, so try that if you want a production app on your own domain. If you are happy to test on localhost:8080, then set up these properties in your application configuration:

**application.yml.**  

```java
security:
oauth2:
client:
clientId: bd1c0a783ccdd1c9b9e4
clientSecret: 1a9030fbca47a5b2c28e92f19050bb77824b5ad1
accessTokenUri: https://github.com/login/oauth/access_token
userAuthorizationUri: https://github.com/login/oauth/authorize
clientAuthenticationScheme: form
resource:
userInfoUri: https://api.github.com/user
preferTokenInfo: false
```

run the app above and it will redirect to github for authorization. If you are already signed into github you won’t even notice that it has authenticated. These credentials will only work if your app is running on port 8080.

To limit the scope that the client asks for when it obtains an access token you can set  `security.oauth2.client.scope`  (comma separated or an array in YAML). By default the scope is empty and it is up to to Authorization Server to decide what the defaults should be, usually depending on the settings in the client registration that it holds.

> The examples above are all Groovy scripts. If you want to write the same code in Java (or Groovy) you need to add Spring Security OAuth2 to the classpath (e.g. see the [sample here](https://github.com/spring-cloud-samples/sso)).

## 83.2 OAuth2 Protected Resource

You want to protect an API resource with an OAuth2 token? Here’s a simple example (paired with the client above):

**app.groovy.**  

```java
@Grab('spring-cloud-starter-security')
@RestController
@EnableResourceServer
class Application {

@RequestMapping('/')
def home() {
[message: 'Hello World']
}

}
```

and

**application.yml.**  

```java
security:
oauth2:
resource:
userInfoUri: https://api.github.com/user
preferTokenInfo: false
```


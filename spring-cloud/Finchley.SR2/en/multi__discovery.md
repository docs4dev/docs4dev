## 86. Discovery

Here’s a Spring Cloud app with Cloud Foundry discovery:

**app.groovy.**  

```java
@Grab('org.springframework.cloud:spring-cloud-cloudfoundry')
@RestController
@EnableDiscoveryClient
class Application {

@Autowired
DiscoveryClient client

@RequestMapping('/')
String home() {
'Hello from ' + client.getLocalServiceInstance()
}

}
```

If you run it without any service bindings:

```java
$ spring jar app.jar app.groovy
$ cf push -p app.jar
```

It will show its app name in the home page.

The  `DiscoveryClient`  can lists all the apps in a space, according to the credentials it is authenticated with, where the space defaults to the one the client is running in (if any). If neither org nor space are configured, they default per the user’s profile in Cloud Foundry.

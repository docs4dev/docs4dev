## 86.发现

这是一个使用Cloud Foundry发现的Spring Cloud应用程序：

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

如果在没有任何服务绑定的情况下运行它：

```java
$ spring jar app.jar app.groovy
$ cf push -p app.jar
```

它将在主页中显示其应用名称.

`DiscoveryClient` 可以根据通过身份验证的凭据列出空间中的所有应用，其中空间默认为客户端运行的空间（如果有）.如果既未配置org也未配置空间，则它们默认为Cloud Foundry中用户的配置文件.

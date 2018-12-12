## 20. Developer Tools

Spring Boot includes an additional set of tools that can make the application development experience a little more pleasant. The  `spring-boot-devtools`  module can be included in any project to provide additional development-time features. To include devtools support, add the module dependency to your build, as shown in the following listings for Maven and Gradle:

**Maven.**  

```xml
<dependencies>
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-devtools</artifactId>
		<optional>true</optional>
	</dependency>
</dependencies>
```

**Gradle.**  

```java
configurations {
	developmentOnly
	runtimeClasspath {
		extendsFrom developmentOnly
	}
}
dependencies {
	developmentOnly("org.springframework.boot:spring-boot-devtools")
}
```

> Developer tools are automatically disabled when running a fully packaged application. If your application is launched from  `java -jar`  or if it is started from a special classloader, then it is considered a “production application”. Flagging the dependency as optional in Maven or using a custom`developmentOnly` configuration in Gradle (as shown above) is a best practice that prevents devtools from being transitively applied to other modules that use your project.

> Repackaged archives do not contain devtools by default. If you want to use a [certain remote devtools feature](using-boot-devtools.html#using-boot-devtools-remote), you need to disable the  `excludeDevtools`  build property to include it. The property is supported with both the Maven and Gradle plugins.

## 20.1 Property Defaults

Several of the libraries supported by Spring Boot use caches to improve performance. For example, [template engines](boot-features-developing-web-applications.html#boot-features-spring-mvc-template-engines) cache compiled templates to avoid repeatedly parsing template files. Also, Spring MVC can add HTTP caching headers to responses when serving static resources.

While caching is very beneficial in production, it can be counter-productive during development, preventing you from seeing the changes you just made in your application. For this reason, spring-boot-devtools disables the caching options by default.

Cache options are usually configured by settings in your  `application.properties`  file. For example, Thymeleaf offers the  `spring.thymeleaf.cache`  property. Rather than needing to set these properties manually, the  `spring-boot-devtools`  module automatically applies sensible development-time configuration.

Because you need more information about web requests while developing Spring MVC and Spring WebFlux applications, developer tools will enable  `DEBUG`  logging for the  `web`  logging group. This will give you information about the incoming request, which handler is processing it, the response outcome, etc. If you wish to log all request details (including potentially sensitive information), you can turn on the  `spring.http.log-request-details`  configuration property.

> If you don’t want property defaults to be applied you can set  `spring.devtools.add-properties`  to  `false`  in your  `application.properties` .

> For a complete list of the properties that are applied by the devtools, see [DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java).

## 20.2 Automatic Restart

Applications that use  `spring-boot-devtools`  automatically restart whenever files on the classpath change. This can be a useful feature when working in an IDE, as it gives a very fast feedback loop for code changes. By default, any entry on the classpath that points to a folder is monitored for changes. Note that certain resources, such as static assets and view templates, [do not need to restart the application](using-boot-devtools.html#using-boot-devtools-restart-exclude).

----
**Triggering a restart** 

As DevTools monitors classpath resources, the only way to trigger a restart is to update the classpath. The way in which you cause the classpath to be updated depends on the IDE that you are using. In Eclipse, saving a modified file causes the classpath to be updated and triggers a restart. In IntelliJ IDEA, building the project ( `Build -> Build Project` ) has the same effect.

----

> As long as forking is enabled, you can also start your application by using the supported build plugins (Maven and Gradle), since DevTools needs an isolated application classloader to operate properly. By default, Gradle and Maven do that when they detect DevTools on the classpath.

> Automatic restart works very well when used with LiveReload. [See the LiveReload section](using-boot-devtools.html#using-boot-devtools-livereload) for details. If you use JRebel, automatic restarts are disabled in favor of dynamic class reloading. Other devtools features (such as LiveReload and property overrides) can still be used.

> DevTools relies on the application context’s shutdown hook to close it during a restart. It does not work correctly if you have disabled the shutdown hook ( `SpringApplication.setRegisterShutdownHook(false)` ).

> When deciding if an entry on the classpath should trigger a restart when it changes, DevTools automatically ignores projects named  `spring-boot` ,  `spring-boot-devtools` ,  `spring-boot-autoconfigure` ,  `spring-boot-actuator` , and  `spring-boot-starter` .

> DevTools needs to customize the  `ResourceLoader`  used by the  `ApplicationContext` . If your application provides one already, it is going to be wrapped. Direct override of the  `getResource`  method on the  `ApplicationContext`  is not supported.

----
**Restart vs Reload** 

The restart technology provided by Spring Boot works by using two classloaders. Classes that do not change (for example, those from third-party jars) are loaded into a base classloader. Classes that you are actively developing are loaded into a restart classloader. When the application is restarted, the restart classloader is thrown away and a new one is created. This approach means that application restarts are typically much faster than “cold starts”, since the base classloader is already available and populated.

If you find that restarts are not quick enough for your applications or you encounter classloading issues, you could consider reloading technologies such as [JRebel](https://zeroturnaround.com/software/jrebel/) from ZeroTurnaround. These work by rewriting classes as they are loaded to make them more amenable to reloading.

----

### 20.2.1 Logging changes in condition evaluation

By default, each time your application restarts, a report showing the condition evaluation delta is logged. The report shows the changes to your application’s auto-configuration as you make changes such as adding or removing beans and setting configuration properties.

To disable the logging of the report, set the following property:

```java
spring.devtools.restart.log-condition-evaluation-delta=false
```

### 20.2.2 Excluding Resources

Certain resources do not necessarily need to trigger a restart when they are changed. For example, Thymeleaf templates can be edited in-place. By default, changing resources in  `/META-INF/maven` ,  `/META-INF/resources` ,  `/resources` ,  `/static` ,  `/public` , or  `/templates`  does not trigger a restart but does trigger a [live reload](using-boot-devtools.html#using-boot-devtools-livereload). If you want to customize these exclusions, you can use the  `spring.devtools.restart.exclude`  property. For example, to exclude only  `/static`  and  `/public`  you would set the following property:

```java
spring.devtools.restart.exclude=static/**,public/**
```

> If you want to keep those defaults and add additional exclusions, use the  `spring.devtools.restart.additional-exclude`  property instead.

### 20.2.3 Watching Additional Paths

You may want your application to be restarted or reloaded when you make changes to files that are not on the classpath. To do so, use the  `spring.devtools.restart.additional-paths`  property to configure additional paths to watch for changes. You can use the  `spring.devtools.restart.exclude`  property [described earlier](using-boot-devtools.html#using-boot-devtools-restart-exclude) to control whether changes beneath the additional paths trigger a full restart or a [live reload](using-boot-devtools.html#using-boot-devtools-livereload).

### 20.2.4 Disabling Restart

If you do not want to use the restart feature, you can disable it by using the  `spring.devtools.restart.enabled`  property. In most cases, you can set this property in your  `application.properties`  (doing so still initializes the restart classloader, but it does not watch for file changes).

If you need to completely disable restart support (for example, because it does not work with a specific library), you need to set the  `spring.devtools.restart.enabled`   `System`  property to  `false`  before calling  `SpringApplication.run(…)` , as shown in the following example:

```java
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```

### 20.2.5 Using a Trigger File

If you work with an IDE that continuously compiles changed files, you might prefer to trigger restarts only at specific times. To do so, you can use a “trigger file”, which is a special file that must be modified when you want to actually trigger a restart check. Changing the file only triggers the check and the restart only occurs if Devtools has detected it has to do something. The trigger file can be updated manually or with an IDE plugin.

To use a trigger file, set the  `spring.devtools.restart.trigger-file`  property to the path of your trigger file.

> You might want to set  `spring.devtools.restart.trigger-file`  as a [global setting](using-boot-devtools.html#using-boot-devtools-globalsettings), so that all your projects behave in the same way.

### 20.2.6 Customizing the Restart Classloader

As described earlier in the [Restart vs Reload](using-boot-devtools.html#using-spring-boot-restart-vs-reload) section, restart functionality is implemented by using two classloaders. For most applications, this approach works well. However, it can sometimes cause classloading issues.

By default, any open project in your IDE is loaded with the “restart” classloader, and any regular  `.jar`  file is loaded with the “base” classloader. If you work on a multi-module project, and not every module is imported into your IDE, you may need to customize things. To do so, you can create a  `META-INF/spring-devtools.properties`  file.

The  `spring-devtools.properties`  file can contain properties prefixed with  `restart.exclude`  and  `restart.include` . The  `include`  elements are items that should be pulled up into the “restart” classloader, and the  `exclude`  elements are items that should be pushed down into the “base” classloader. The value of the property is a regex pattern that is applied to the classpath, as shown in the following example:

```java
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```

> All property keys must be unique. As long as a property starts with  `restart.include.`  or  `restart.exclude.`  it is considered.

> All  `META-INF/spring-devtools.properties`  from the classpath are loaded. You can package files inside your project, or in the libraries that the project consumes.

### 20.2.7 Known Limitations

Restart functionality does not work well with objects that are deserialized by using a standard  `ObjectInputStream` . If you need to deserialize data, you may need to use Spring’s  `ConfigurableObjectInputStream`  in combination with  `Thread.currentThread().getContextClassLoader()` .

Unfortunately, several third-party libraries deserialize without considering the context classloader. If you find such a problem, you need to request a fix with the original authors.

## 20.3 LiveReload

The  `spring-boot-devtools`  module includes an embedded LiveReload server that can be used to trigger a browser refresh when a resource is changed. LiveReload browser extensions are freely available for Chrome, Firefox and Safari from [livereload.com](http://livereload.com/extensions/).

If you do not want to start the LiveReload server when your application runs, you can set the  `spring.devtools.livereload.enabled`  property to  `false` .

> You can only run one LiveReload server at a time. Before starting your application, ensure that no other LiveReload servers are running. If you start multiple applications from your IDE, only the first has LiveReload support.

## 20.4 Global Settings

You can configure global devtools settings by adding a file named  `.spring-boot-devtools.properties`  to your  `$HOME`  folder (note that the filename starts with “.”). Any properties added to this file apply to all Spring Boot applications on your machine that use devtools. For example, to configure restart to always use a [trigger file](using-boot-devtools.html#using-boot-devtools-restart-triggerfile), you would add the following property:

**~/.spring-boot-devtools.properties.**  

```java
spring.devtools.reload.trigger-file=.reloadtrigger
```

## 20.5 Remote Applications

The Spring Boot developer tools are not limited to local development. You can also use several features when running applications remotely. Remote support is opt-in. To enable it, you need to make sure that  `devtools`  is included in the repackaged archive, as shown in the following listing:

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-maven-plugin</artifactId>
			<configuration>
				<excludeDevtools>false</excludeDevtools>
			</configuration>
		</plugin>
	</plugins>
</build>
```

Then you need to set a  `spring.devtools.remote.secret`  property, as shown in the following example:

```java
spring.devtools.remote.secret=mysecret
```

> Enabling  `spring-boot-devtools`  on a remote application is a security risk. You should never enable support on a production deployment.

Remote devtools support is provided in two parts: a server-side endpoint that accepts connections and a client application that you run in your IDE. The server component is automatically enabled when the  `spring.devtools.remote.secret`  property is set. The client component must be launched manually.

### 20.5.1 Running the Remote Client Application

The remote client application is designed to be run from within your IDE. You need to run  `org.springframework.boot.devtools.RemoteSpringApplication`  with the same classpath as the remote project that you connect to. The application’s single required argument is the remote URL to which it connects.

For example, if you are using Eclipse or STS and you have a project named  `my-app`  that you have deployed to Cloud Foundry, you would do the following:

- Select  `Run Configurations…`  from the  `Run`  menu.

- Create a new  `Java Application`  “launch configuration”.

- Browse for the  `my-app`  project.

- Use  `org.springframework.boot.devtools.RemoteSpringApplication`  as the main class.

- Add  `https://myapp.cfapps.io`  to the  `Program arguments`  (or whatever your remote URL is).

A running remote client might resemble the following listing:

```java
.   ____          _                                              __ _ _
/\\ / ___'_ __ _ _(_)_ __  __ _          ___               _      \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` |        | _ \___ _ __  ___| |_ ___ \ \ \ \
\\/  ___)| |_)| | | | | || (_| []::::::[]   / -_) '  \/ _ \  _/ -_) ) ) ) )
'  |____| .__|_| |_|_| |_\__, |        |_|_\___|_|_|_\___/\__\___|/ / / /
=========|_|==============|___/===================================/_/_/_/
:: Spring Boot Remote :: 2.1.0.RELEASE

2015-06-10 18:25:06.632  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Starting RemoteSpringApplication on pwmbp with PID 14938 (/Users/pwebb/projects/spring-boot/code/spring-boot-devtools/target/classes started by pwebb in /Users/pwebb/projects/spring-boot/code/spring-boot-samples/spring-boot-sample-devtools)
2015-06-10 18:25:06.671  INFO 14938 --- [           main] s.c.a.AnnotationConfigApplicationContext : Refreshing org.spring[emailprotected]2a17b7b6: startup date [Wed Jun 10 18:25:06 PDT 2015]; root of context hierarchy
2015-06-10 18:25:07.043  WARN 14938 --- [           main] o.s.b.d.r.c.RemoteClientConfiguration    : The connection to http://localhost:8080 is insecure. You should use a URL starting with 'https://'.
2015-06-10 18:25:07.074  INFO 14938 --- [           main] o.s.b.d.a.OptionalLiveReloadServer       : LiveReload server is running on port 35729
2015-06-10 18:25:07.130  INFO 14938 --- [           main] o.s.b.devtools.RemoteSpringApplication   : Started RemoteSpringApplication in 0.74 seconds (JVM running for 1.105)
```

> Because the remote client is using the same classpath as the real application it can directly read application properties. This is how the  `spring.devtools.remote.secret`  property is read and passed to the server for authentication.

> It is always advisable to use  `https://`  as the connection protocol, so that traffic is encrypted and passwords cannot be intercepted.

> If you need to use a proxy to access the remote application, configure the  `spring.devtools.remote.proxy.host`  and  `spring.devtools.remote.proxy.port`  properties.

### 20.5.2 Remote Update

The remote client monitors your application classpath for changes in the same way as the [local restart](using-boot-devtools.html#using-boot-devtools-restart). Any updated resource is pushed to the remote application and (if required) triggers a restart. This can be helpful if you iterate on a feature that uses a cloud service that you do not have locally. Generally, remote updates and restarts are much quicker than a full rebuild and deploy cycle.

> Files are only monitored when the remote client is running. If you change a file before starting the remote client, it is not pushed to the remote server.


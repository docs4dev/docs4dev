## 20.开发人员工具

Spring Boot包含一组额外的工具，可以使应用程序开发体验更加愉快.  `spring-boot-devtools` 模块可以包含在任何项目中，以提供额外的开发时间功能.要包含devtools支持，请将模块依赖项添加到您的构建中，如以下Maven和Gradle列表所示：

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

运行完全打包的应用程序时，会自动禁用
> Developer工具.如果您的应用程序是从 `java -jar` 启动的，或者是从特殊的类加载器启动的，那么它将被视为“生产环境应用程序”.在Maven中将依赖项标记为可选，或在Gradle中使用自定义`开发的配置（如上所示）是最佳实践，可防止devtools传递应用于使用您的项目的其他模块.

> Repackaged归档默认情况下不包含devtools.如果要使用[certain remote devtools feature](using-boot-devtools.html#using-boot-devtools-remote)，则需要禁用 `excludeDevtools` 构建属性以包含它.该属性由Maven和Gradle插件支持.

## 20.1属性默认值

Spring Boot支持的几个库使用缓存来提高性能.例如，[template engines](boot-features-developing-web-applications.html#boot-features-spring-mvc-template-engines)缓存编译模板以避免重复解析模板文件.此外，Spring MVC可以在提供静态资源时为响应添加HTTP缓存头.

虽然缓存在生产环境中非常有用，但在开发过程中可能会适得其反，从而使您无法看到刚刚在应用程序中进行的更改.因此，spring-boot-devtools默认禁用缓存选项.

缓存选项通常由 `application.properties` 文件中的设置配置.例如，Thymeleaf提供 `spring.thymeleaf.cache` 属性.  `spring-boot-devtools` 模块不需要手动设置这些属性，而是自动应用合理的开发时配置.

因为在开发Spring MVC和Spring WebFlux应用程序时需要有关Web请求的更多信息，所以开发人员工具将为 `web` 日志记录组启用 `DEBUG` 日志记录.这将为您提供有关传入请求，处理程序正在处理它，响应结果等的信息.如果您希望记录所有请求详细信息（包括可能的敏感信息），则可以打开 `spring.http.log-request-details` 配置属性.

> 如果您不想应用属性默认值，可以在 `application.properties` 中将 `spring.devtools.add-properties` 设置为 `false` .

> 有关devtools应用的属性的完整列表，请参阅[DevToolsPropertyDefaultsPostProcessor](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-devtools/src/main/java/org/springframework/boot/devtools/env/DevToolsPropertyDefaultsPostProcessor.java).

## 20.2自动重启

只要类路径上的文件发生更改，使用 `spring-boot-devtools` 的应用程序就会自动重新启动.在IDE中工作时，这可能是一个有用的功能，因为它为代码更改提供了非常快速的反馈循环.默认情况下，将监视类路径上指向文件夹的任何条目的更改.请注意某些资源，例如静态资源和视图模板，[do not need to restart the application](using-boot-devtools.html#using-boot-devtools-restart-exclude).

----
**Triggering a restart** 

由于DevTools监视类路径资源，因此触发重新启动的唯一方法是更新类路径.导致更新类路径的方式取决于您使用的IDE.在Eclipse中，保存修改后的文件会导致更新类路径并触发重新启动.在IntelliJ IDEA中，构建项目（ `Build -> Build Project` ）具有相同的效果.

----

> 只要启用了分叉，您也可以使用支持的构建插件（Maven和Gradle）启动应用程序，因为DevTools需要一个独立的应用程序类加载器才能正常运行.默认情况下，Gradle和Maven在类路径上检测到DevTools时会这样做.

> 与LiveReload一起使用时，自动重启非常有效. [See the LiveReload section](using-boot-devtools.html#using-boot-devtools-livereload)了解详情.如果使用JRebel，则禁用自动重新启动以支持动态类重新加载.其他devtools功能（例如LiveReload和属性覆盖）仍然可以使用.

> DevTools依赖应用程序上下文的关闭挂钩在重启期间关闭它.如果已禁用关闭挂钩（ `SpringApplication.setRegisterShutdownHook(false)` ），则无法正常工作.

> 当决定类路径上的条目是否应在更改时触发重新启动时，DevTools会自动忽略名为 `spring-boot` ， `spring-boot-devtools` ， `spring-boot-autoconfigure` ， `spring-boot-actuator` 和 `spring-boot-starter` 的项目.

> DevTools需要自定义 `ApplicationContext` 使用的 `ResourceLoader` .如果您的应用程序已经提供了一个，它将被包装.不支持直接覆盖 `ApplicationContext` 上的 `getResource` 方法.

----
**Restart vs Reload** 

Spring Boot提供的重启技术使用两个类加载器.不更改的类（例如，来自第三方jar的类）将加载到基类加载器中.您正在积极开发的类将加载到重新启动的类加载器中.重新启动应用程序时，重新启动抛弃classloader并创建一个新的.这种方法意味着应用程序重新启动通常比“冷启动”快得多，因为基本类加载器已经可用并已填充.

如果您发现重新启动对于您的应用程序来说不够快，或者遇到类加载问题，您可以考虑从ZeroTurnaround重新加载[JRebel](https://zeroturnaround.com/software/jrebel/)等技术.这些工作通过在加载类时重写类使它们更适合重新加载.

----

### 20.2.1记录条件评估中的更改

默认情况下，每次应用程序重新启动时，都会记录一个显示条件评估增量的报告.该报告显示了在进行更改（例如添加或删除Bean以及设置配置属性）时对应用程序的自动配置所做的更改.

要禁用报告的日志记录，请设置以下属性：

```java
spring.devtools.restart.log-condition-evaluation-delta=false
```

### 20.2.2不包括资源

某些资源在更改时不一定需要触发重启.例如，可以就地编辑Thymeleaf模板.默认情况下，更改 `/META-INF/maven` ， `/META-INF/resources` ， `/resources` ， `/static` ， `/public` 或 `/templates` 中的资源不会触发重新启动但会触发[live reload](using-boot-devtools.html#using-boot-devtools-livereload).如果要自定义这些排除项，可以使用 `spring.devtools.restart.exclude` 属性.例如，要仅排除 `/static` 和 `/public` ，您将设置以下属性：

```java
spring.devtools.restart.exclude=static/**,public/**
```

> 如果要保留这些默认值并添加其他排除项，请改用 `spring.devtools.restart.additional-exclude` 属性.

### 20.2.3正在查看其他路径

当您对不在类路径中的文件进行更改时，您可能希望重新启动或重新加载应用程序.为此，请使用 `spring.devtools.restart.additional-paths` 属性配置其他路径以监视更改.您可以使用 `spring.devtools.restart.exclude` 属性[described earlier](using-boot-devtools.html#using-boot-devtools-restart-exclude)来控制其他路径下的更改是触发完全重启还是[live reload](using-boot-devtools.html#using-boot-devtools-livereload).

### 20.2.4禁用重启

如果您不想使用重启功能，可以使用 `spring.devtools.restart.enabled` 属性将其禁用.在大多数情况下，您可以在 `application.properties` 中设置此属性（这样做仍会初始化重新启动的类加载器，但它不会监视文件更改）.

如果需要完全禁用重新启动支持（例如，因为它不能与特定库一起使用），则需要在调用 `SpringApplication.run(…)` 之前将 `spring.devtools.restart.enabled`   `System` 属性设置为 `false` ，如以下示例所示：

```java
public static void main(String[] args) {
	System.setProperty("spring.devtools.restart.enabled", "false");
	SpringApplication.run(MyApp.class, args);
}
```

### 20.2.5使用触发文件

如果使用不断编译已更改文件的IDE，则可能更喜欢仅在特定时间触发重新启动.为此，您可以使用“触发器文件”，这是一个特殊文件，当您想要实际触发重新启动检查时必须对其进行修改.更改文件只会触发检查，只有在Devtools检测到必须执行某些操作时才会重新启动.触发器文件可以手动更新，也可以使用IDE插件更新.

要使用触发器文件，请将 `spring.devtools.restart.trigger-file` 属性设置为触发器文件的路径.

> 您可能希望将 `spring.devtools.restart.trigger-file` 设置为[global setting](using-boot-devtools.html#using-boot-devtools-globalsettings)，以便所有项目的行为方式相同.

### 20.2.6自定义重启类加载器

如前面[Restart vs Reload](using-boot-devtools.html#using-spring-boot-restart-vs-reload)部分所述，重启功能是通过使用两个类加载器实现的.对于大多数应用程序，此方法运行良好.但是，它有时会导致类加载问题.

默认情况下，IDE中的任何打开项目都使用“restart”类加载器加载，任何常规 `.jar` 文件都加载了“base”类加载器.如果您处理多模块项目，并且并非每个模块都导入到IDE中，则可能需要自定义内容.为此，您可以创建 `META-INF/spring-devtools.properties` 文件.

`spring-devtools.properties` 文件可以包含前缀为 `restart.exclude` 和 `restart.include` 的属性.  `include` 元素是应该被拉入“restart”类加载器的项目，而 `exclude` 元素是应该被下推到“基础”类加载器中的项目.该属性的值是应用于类路径的正则表达式模式，如以下示例所示：

```java
restart.exclude.companycommonlibs=/mycorp-common-[\\w-]+\.jar
restart.include.projectcommon=/mycorp-myproj-[\\w-]+\.jar
```

> 所有属性键必须是唯一的.只要属性以 `restart.include.` 或 `restart.exclude.` 开头，就会被考虑.

> 从类路径中加载所有 `META-INF/spring-devtools.properties` .您可以将文件打包到项目中，也可以打包在项目使用的库中.

### 20.2.7已知限制

对于使用标准 `ObjectInputStream` 反序列化的对象，重新启动功能不起作用.如果需要反序列化数据，可能需要将Spring的 `ConfigurableObjectInputStream` 与 `Thread.currentThread().getContextClassLoader()` 结合使用.

不幸的是，几个第三方库反序列化而没有考虑上下文类加载器.如果您发现此类问题，则需要向原始作者请求修复.

## 20.3 LiveReload

`spring-boot-devtools` 模块包含一个嵌入式LiveReload服务器，可用于在更改资源时触发浏览器刷新. LiveReload浏览器扩展程序可从[livereload.com](http://livereload.com/extensions/)免费用于Chrome，Firefox和Safari.

如果您不希望在应用程序运行时启动LiveReload服务器，则可以设置 `spring.devtools.livereload.enabled` property到 `false` .

> 您一次只能运行一个LiveReload服务器.在启动应用程序之前，请确保没有其他LiveReload服务器正在运行.如果从IDE启动多个应用程序，则只有第一个具有LiveReload支持.

## 20.4全局设置

您可以通过将名为 `.spring-boot-devtools.properties` 的文件添加到 `$HOME` 文件夹来配置全局devtools设置（请注意，文件名以“.”开头）.添加到此文件的任何属性都适用于计算机上使用devtools的所有Spring Boot应用程序.例如，要将restart配置为始终使用[trigger file](using-boot-devtools.html#using-boot-devtools-restart-triggerfile)，您将添加以下属性：

**~/.spring-boot-devtools.properties.** 

```java
spring.devtools.reload.trigger-file=.reloadtrigger
```

## 20.5远程应用程序

Spring Boot开发人员工具不仅限于本地开发.远程运行应用程序时，您还可以使用多个功能.远程支持是选择加入.要启用它，您需要确保 `devtools` 包含在重新打包的存档中，如下面的清单所示：

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

然后，您需要设置 `spring.devtools.remote.secret` 属性，如以下示例所示：

```java
spring.devtools.remote.secret=mysecret
```

> 在远程应用程序上启用 `spring-boot-devtools` 是一种安全风险.您永远不应该在生产环境部署上启用支持.

远程devtools支持由两部分组成：一个接受连接的服务器端endpoints和一个在IDE中运行的客户端应用程序.设置 `spring.devtools.remote.secret` 属性时，将自动启用服务器组件.必须手动启动客户端组件.

### 20.5.1运行远程客户端应用程序

远程客户端应用程序旨在从IDE中运行.您需要使用与连接到的远程项目相同的类路径运行 `org.springframework.boot.devtools.RemoteSpringApplication` .应用程序的单个必需参数是它连接的远程URL.

例如，如果您使用的是Eclipse或STS，并且已经部署到Cloud Foundry的项目名为 `my-app` ，则可以执行以下操作：

- 从 `Run` 菜单中选择 `Run Configurations…` .

- 创建一个新的 `Java Application` “启动配置”.

- Browse  `my-app` 项目.

- 使用 `org.springframework.boot.devtools.RemoteSpringApplication` 作为主类.

- 添加 `https://myapp.cfapps.io` 到 `Program arguments` （或任何远程URL）.

正在运行的远程客户端可能类似于以下列表：

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

> B因为远程客户端使用与真实应用程序相同的类路径，所以它可以直接读取应用程序属性.这是 `spring.devtools.remote.secret` 属性的读取方式并传递给服务器进行身份验证.

> 始终建议使用 `https://` 作为连接协议，以便加密流量并且不能截取密码.

> 如果需要使用代理来访问远程应用程序，请配置 `spring.devtools.remote.proxy.host` 和 `spring.devtools.remote.proxy.port` 属性.

### 20.5.2远程更新

远程客户端以与[local restart](using-boot-devtools.html#using-boot-devtools-restart)相同的方式监视应用程序类路径以进行更改.任何更新的资源都会被推送到远程应用程序，并且（如果需要）会触发重新启动.如果您迭代使用本地没有的Cloud服务的功能，这将非常有用.通常，远程更新和重新启动比完全重建和部署周期快得多.

> Files仅在远程客户端运行时受到监视.如果在启动远程客户端之前更改文件，则不会将其推送到远程服务器.


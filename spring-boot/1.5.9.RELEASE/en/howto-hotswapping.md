## 90. Hot Swapping

Spring Boot supports hot swapping. This section answers questions about how it works.

## 90.1 Reload Static Content

There are several options for hot reloading. The recommended approach is to use [spring-boot-devtools](using-boot-devtools.html), as it provides additional development-time features, such as support for fast application restarts and LiveReload as well as sensible development-time configuration (such as template caching). Devtools works by monitoring the classpath for changes. This means that static resource changes must be "built" for the change to take affect. By default, this happens automatically in Eclipse when you save your changes. In IntelliJ IDEA, the Make Project command triggers the necessary build. Due to the [default restart exclusions](using-boot-devtools.html#using-boot-devtools-restart-exclude), changes to static resources do not trigger a restart of your application. They do, however, trigger a live reload.

Alternatively, running in an IDE (especially with debugging on) is a good way to do development (all modern IDEs allow reloading of static resources and usually also allow hot-swapping of Java class changes).

Finally, the [Maven and Gradle plugins](build-tool-plugins.html) can be configured (see the  `addResources`  property) to support running from the command line with reloading of static files directly from source. You can use that with an external css/js compiler process if you are writing that code with higher-level tools.

## 90.2 Reload Templates without Restarting the Container

Most of the templating technologies supported by Spring Boot include a configuration option to disable caching (described later in this document). If you use the  `spring-boot-devtools`  module, these properties are [automatically configured](using-boot-devtools.html#using-boot-devtools-property-defaults) for you at development time.

### 90.2.1 Thymeleaf Templates

If you use Thymeleaf, set  `spring.thymeleaf.cache`  to  `false` . See [ThymeleafAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/thymeleaf/ThymeleafAutoConfiguration.java) for other Thymeleaf customization options.

### 90.2.2 FreeMarker Templates

If you use FreeMarker, set  `spring.freemarker.cache`  to  `false` . See [FreeMarkerAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/freemarker/FreeMarkerAutoConfiguration.java) for other FreeMarker customization options.

### 90.2.3 Groovy Templates

If you use Groovy templates, set  `spring.groovy.template.cache`  to  `false` . See [GroovyTemplateAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/groovy/template/GroovyTemplateAutoConfiguration.java) for other Groovy customization options.

## 90.3 Fast Application Restarts

The  `spring-boot-devtools`  module includes support for automatic application restarts. While not as fast as technologies such as [JRebel](http://zeroturnaround.com/software/jrebel/) it is usually significantly faster than a “cold start”. You should probably give it a try before investigating some of the more complex reload options discussed later in this document.

For more details, see the [Chapter 20, Developer Tools](using-boot-devtools.html) section.

## 90.4 Reload Java Classes without Restarting the Container

Many modern IDEs (Eclipse, IDEA, and others) support hot swapping of bytecode. Consequently, if you make a change that does not affect class or method signatures, it should reload cleanly with no side effects.


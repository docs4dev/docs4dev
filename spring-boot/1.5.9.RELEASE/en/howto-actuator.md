## 88. Actuator

Spring Boot includes the Spring Boot Actuator. This section answers questions that often arise from its use.

## 88.1 Change the HTTP Port or Address of the Actuator Endpoints

In a standalone application, the Actuator HTTP port defaults to the same as the main HTTP port. To make the application listen on a different port, set the external property:  `management.server.port` . To listen on a completely different network address (such as when you have an internal network for management and an external one for user applications), you can also set  `management.server.address`  to a valid IP address to which the server is able to bind.

For more detail, see the [ManagementServerProperties](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-actuator-autoconfigure/src/main/java/org/springframework/boot/actuate/autoconfigure/web/server/ManagementServerProperties.java) source code and “[Section 54.2, “Customizing the Management Server Port”](production-ready-monitoring.html#production-ready-customizing-management-server-port)” in the “Production-ready features” section.

## 88.2 Customize the ‘whitelabel’ Error Page

Spring Boot installs a ‘whitelabel’ error page that you see in a browser client if you encounter a server error (machine clients consuming JSON and other media types should see a sensible response with the right error code).

> Set  `server.error.whitelabel.enabled=false`  to switch the default error page off. Doing so restores the default of the servlet container that you are using. Note that Spring Boot still tries to resolve the error view, so you should probably add your own error page rather than disabling it completely.

Overriding the error page with your own depends on the templating technology that you use. For example, if you use Thymeleaf, you can add an  `error.html`  template. If you use FreeMarker, you can add an  `error.ftl`  template. In general, you need a  `View`  that resolves with a name of  `error`  or a  `@Controller`  that handles the  `/error`  path. Unless you replaced some of the default configuration, you should find a  `BeanNameViewResolver`  in your  `ApplicationContext` , so a  `@Bean`  named  `error`  would be a simple way of doing that. See [ErrorMvcAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/web/servlet/error/ErrorMvcAutoConfiguration.java) for more options.

See also the section on “[Error Handling](boot-features-developing-web-applications.html#boot-features-error-handling)” for details of how to register handlers in the servlet container.

## 88.3 Sanitize sensible values

Information returned by the  `env`  and  `configprops`  endpoints can be somewhat sensitive so keys matching a certain pattern are sanitized by default (i.e. their values are replaced by  `******` ).

Spring Boot uses sensible defaults for such keys: for instance, any key ending with the word "password", "secret", "key" or "token" is sanitized. It is also possible to use a regular expression instead, such as  `*credentials.*`  to sanitize any key that holds the word  `credentials`  as part of the key.

The patterns to use can be customized using the  `management.endpoint.env.keys-to-sanitize`  and  `management.endpoint.configprops.keys-to-sanitize`  respectively.


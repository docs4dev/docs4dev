## 61. Cloud Foundry Support

Spring Boot’s actuator module includes additional support that is activated when you deploy to a compatible Cloud Foundry instance. The  `/cloudfoundryapplication`  path provides an alternative secured route to all  `@Endpoint`  beans.

The extended support lets Cloud Foundry management UIs (such as the web application that you can use to view deployed applications) be augmented with Spring Boot actuator information. For example, an application status page may include full health information instead of the typical “running” or “stopped” status.

> The  `/cloudfoundryapplication`  path is not directly accessible to regular users. In order to use the endpoint, a valid UAA token must be passed with the request.

## 61.1 Disabling Extended Cloud Foundry Actuator Support

If you want to fully disable the  `/cloudfoundryapplication`  endpoints, you can add the following setting to your  `application.properties`  file:

**application.properties.**  

```java
management.cloudfoundry.enabled=false
```

## 61.2 Cloud Foundry Self-signed Certificates

By default, the security verification for  `/cloudfoundryapplication`  endpoints makes SSL calls to various Cloud Foundry services. If your Cloud Foundry UAA or Cloud Controller services use self-signed certificates, you need to set the following property:

**application.properties.**  

```java
management.cloudfoundry.skip-ssl-validation=true
```

## 61.3 Custom context path

If the server’s context-path has been configured to anything other than  `/` , the Cloud Foundry endpoints will not be available at the root of the application. For example, if  `server.servlet.context-path=/app` , Cloud Foundry endpoints will be available at  `/app/cloudfoundryapplication/*` .

If you expect the Cloud Foundry endpoints to always be available at  `/cloudfoundryapplication/*` , regardless of the server’s context-path, you will need to explicitly configure that in your application. The configuration will differ depending on the web server in use. For Tomcat, the following configuration can be added:

```java
@Bean
public TomcatServletWebServerFactory servletWebServerFactory() {
	return new TomcatServletWebServerFactory() {

		@Override
		protected void prepareContext(Host host,
				ServletContextInitializer[] initializers) {
			super.prepareContext(host, initializers);
			StandardContext child = new StandardContext();
			child.addLifecycleListener(new Tomcat.FixContextListener());
			child.setPath("/cloudfoundryapplication");
			ServletContainerInitializer initializer = getServletContextInitializer(
					getContextPath());
			child.addServletContainerInitializer(initializer, Collections.emptySet());
			child.setCrossContext(true);
			host.addChild(child);
		}

	};
}

private ServletContainerInitializer getServletContextInitializer(String contextPath) {
	return (c, context) -> {
		Servlet servlet = new GenericServlet() {

			@Override
			public void service(ServletRequest req, ServletResponse res)
					throws ServletException, IOException {
				ServletContext context = req.getServletContext()
						.getContext(contextPath);
				context.getRequestDispatcher("/cloudfoundryapplication").forward(req,
						res);
			}

		};
		context.addServlet("cloudfoundry", servlet).addMapping("/*");
	};
}
```


## 61. Cloud Foundry支持

Spring Boot的Actuator模块包括在部署到兼容的Cloud Foundry实例时激活的其他支持.  `/cloudfoundryapplication` 路径为所有 `@Endpoint`  bean提供了另一种安全路由.

扩展支持允许使用Spring BootActuator信息扩充Cloud Foundry管理UI（例如可用于查看已部署应用程序的Web应用程序）.例如，应用程序状态页面可以包括完整的Health信息，而不是典型的“运行”或“停止”状态.

> 常规用户无法直接访问 `/cloudfoundryapplication` 路径.为了使用endpoints，有效的UAA令牌必须与请求一起传递.

## 61.1禁用Extended Cloud Foundry Actuator支持

如果要完全禁用 `/cloudfoundryapplication` endpoints，可以将以下设置添加到 `application.properties` 文件中：

**application.properties.** 

```java
management.cloudfoundry.enabled=false
```

## 61.2 Cloud Foundry自签名证书

默认情况下， `/cloudfoundryapplication` endpoints的安全验证会对各种Cloud Foundry服务进行SSL调用.如果您的Cloud Foundry UAA或Cloud Controller服务使用自签名证书，则需要设置以下属性：

**application.properties.** 

```java
management.cloudfoundry.skip-ssl-validation=true
```

## 61.3自定义上下文路径

如果服务器的上下文路径已配置为 `/` 以外的任何其他内容，则Cloud Foundryendpoints将不会在应用程序的根目录中可用.例如，如果 `server.servlet.context-path=/app` ，Cloud Foundryendpoints将在 `/app/cloudfoundryapplication/*` 处可用.

如果您希望Cloud Foundryendpoints始终在 `/cloudfoundryapplication/*` 处可用，则无论服务器的上下文路径如何，您都需要在应用程序中明确配置它.配置将根据使用的Web服务器而有所不同.对于Tomcat，可以添加以下配置：

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


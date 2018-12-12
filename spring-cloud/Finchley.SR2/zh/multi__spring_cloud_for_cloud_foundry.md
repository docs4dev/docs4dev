# Part XIII. Cloud Foundry的Spring Cloud

Spring Cloud for Cloudfoundry可以在[Cloud Foundry](https://github.com/cloudfoundry)（平台即服务）中轻松运行[Spring Cloud](https://github.com/spring-cloud)应用程序. Cloud Foundry有一个"service"的概念，它是一个appd的中间件，主要是为它提供一个包含凭据的环境变量（例如用于服务的位置和用户名）.

`spring-cloud-cloudfoundry-commons` 模块配置基于Reactor的Cloud Foundry Java客户端v 3.0，可以单独使用.

`spring-cloud-cloudfoundry-web` 项目为Cloud Foundry中的Webapps的某些增强功能提供基本支持：自动绑定到单点登录服务，并可选择为发现启用粘性路由.

`spring-cloud-cloudfoundry-discovery` 项目提供了Spring Cloud Commons  `DiscoveryClient` 的实现，因此您可以 `@EnableDiscoveryClient` 并将您的凭据提供为 `spring.cloud.cloudfoundry.discovery.[username,password]` （如果您没有连接到[Pivotal Web Services](https://run.pivotal.io)也是 `*.url` ）然后您可以直接使用 `DiscoveryClient` 或通过 `LoadBalancerClient` .

第一次使用它时，发现客户端可能会很慢，因为它必须从Cloud Foundry获取访问令牌.


# Part XIII. Spring Cloud for Cloud Foundry

Spring Cloud for Cloudfoundry makes it easy to run [Spring Cloud](https://github.com/spring-cloud) apps in [Cloud Foundry](https://github.com/cloudfoundry) (the Platform as a Service). Cloud Foundry has the notion of a "service", which is middlware that you "bind" to an app, essentially providing it with an environment variable containing credentials (e.g. the location and username to use for the service).

The  `spring-cloud-cloudfoundry-commons`  module configures the Reactor-based Cloud Foundry Java client, v 3.0, and can be used standalone.

The  `spring-cloud-cloudfoundry-web`  project provides basic support for some enhanced features of webapps in Cloud Foundry: binding automatically to single-sign-on services and optionally enabling sticky routing for discovery.

The  `spring-cloud-cloudfoundry-discovery`  project provides an implementation of Spring Cloud Commons  `DiscoveryClient`  so you can  `@EnableDiscoveryClient`  and provide your credentials as  `spring.cloud.cloudfoundry.discovery.[username,password]`  (also  `*.url`  if you are not connecting to [Pivotal Web Services](https://run.pivotal.io)) and then you can use the  `DiscoveryClient`  directly or via a  `LoadBalancerClient` .

The first time you use it the discovery client might be slow owing to the fact that it has to get an access token from Cloud Foundry.


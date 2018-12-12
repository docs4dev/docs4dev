## 36. Samples

For Spring Cloud Stream samples, see the [spring-cloud-stream-samples](https://github.com/spring-cloud/spring-cloud-stream-samples) repository on GitHub.

## 36.1 Deploying Stream Applications on CloudFoundry

On CloudFoundry, services are usually exposed through a special environment variable called [VCAP_SERVICES](https://docs.cloudfoundry.org/devguide/deploy-apps/environment-variable.html#VCAP-SERVICES).

When configuring your binder connections, you can use the values from an environment variable as explained on the [dataflow Cloud Foundry Server](https://docs.spring.io/spring-cloud-dataflow-server-cloudfoundry/docs/current-SNAPSHOT/reference/htmlsingle/#getting-started-ups) docs.


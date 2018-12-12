## 21. HTTP Clients

Spring Cloud Netflix automatically creates the HTTP client used by Ribbon, Feign, and Zuul for you. However, you can also provide your own HTTP clients customized as you need them to be. To do so, you can create a bean of type  `ClosableHttpClient`  if you are using the Apache Http Cient or  `OkHttpClient`  if you are using OK HTTP.

> When you create your own HTTP client, you are also responsible for implementing the correct connection management strategies for these clients. Doing so improperly can result in resource management issues.


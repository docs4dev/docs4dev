## 87. Single Sign On

> All of the OAuth2 SSO and resource server features moved to Spring Boot in version 1.3. You can find documentation in the [Spring Boot user guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/).

This project provides automatic binding from CloudFoundry service credentials to the Spring Boot features. If you have a CloudFoundry service called "sso", for instance, with credentials containing "client_id", "client_secret" and "auth_domain", it will bind automatically to the Spring OAuth2 client that you enable with  `@EnableOAuth2Sso`  (from Spring Boot). The name of the service can be parameterized using  `spring.oauth2.sso.serviceId` .

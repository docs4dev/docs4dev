## 79. Installation

To install, make sure you have [Spring Boot CLI](https://github.com/spring-projects/spring-boot) (1.5.2 or better):

```java
$ spring version
Spring CLI v1.5.4.RELEASE
```

E.g. for SDKMan users

```java
$ sdk install springboot 1.5.4.RELEASE
$ sdk use springboot 1.5.4.RELEASE
```

and install the Spring Cloud plugin

```java
$ mvn install
$ spring install org.springframework.cloud:spring-cloud-cli:1.4.0.BUILD-SNAPSHOT
```

|images/important.png|Important|
|----|----|
| **Prerequisites:**  to use the encryption and decryption features you need the full-strength JCE installed in your JVM (it’s not there by default). You can download the "Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files" from Oracle, and follow instructions for installation (essentially replace the 2 policy files in the JRE lib/security directory with the ones that you downloaded). |


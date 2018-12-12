## 79.安装

要安装，请确保您有[Spring Boot CLI](https://github.com/spring-projects/spring-boot)（1.5.2或更高）：

```java
$ spring version
Spring CLI v1.5.4.RELEASE
```

例如.对于SDKMan用户

```java
$ sdk install springboot 1.5.4.RELEASE
$ sdk use springboot 1.5.4.RELEASE
```

并安装Spring Cloud插件

```java
$ mvn install
$ spring install org.springframework.cloud:spring-cloud-cli:1.4.0.BUILD-SNAPSHOT
```

|图片/ important.png |重要|
| ---- | ---- |
|  **Prerequisites:** 要使用加密和解密功能，您需要在JVM中安装全功能JCE（默认情况下不存在）.您可以从Oracle下载"Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files"，并按照安装说明进行操作（实际上将JRE lib / security目录中的2个策略文件替换为您下载的那些）. |


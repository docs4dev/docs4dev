## 12.服务发现：Eureka Server

本节介绍如何设置Eureka服务器.

## 12.1如何包含Eureka服务器

要在项目中包含Eureka Server，请使用组ID为 `org.springframework.cloud` 且工件ID为 `spring-cloud-starter-netflix-eureka-server` 的starter.有关使用当前Spring Cloud Release Train设置构建系统的详细信息，请参阅[Spring Cloud Project page](https://projects.spring.io/spring-cloud/).

## 12.2如何运行Eureka服务器

以下示例显示了最小的Eureka服务器：

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

public static void main(String[] args) {
new SpringApplicationBuilder(Application.class).web(true).run(args);
}

}
```

服务器有一个主页，其中包含用于 `/eureka/*` 下正常Eureka功能的UI和HTTP APIendpoints.

以下链接有一些Eureka背景阅读：[flux capacitor](https://github.com/cfregly/fluxcapacitor/wiki/NetflixOSS-FAQ#eureka-service-discovery-load-balancer)和[google group discussion](https://groups.google.com/forum/?fromgroups#!topic/eureka_netflix/g3p2r7gHnN0).

> Dyr到Gradle的依赖项解析规则和缺少父bom功能，具体取决于 `spring-cloud-starter-netflix-eureka-server` 可能导致应用程序启动失败.要解决此问题，请添加Spring Boot Gradle插件并导入Spring cloud starter parent bom，如下所示：

## 12.3高可用性，区域和区域

Eureka服务器没有后端存储，但注册表中的服务实例都必须发送心跳以使其注册保持最新（因此可以在内存中完成）.客户端还具有Eureka注册的内存缓存（因此，对于服务的每个请求，他们不必访问注册表）.

默认情况下，每个Eureka服务器也是Eureka客户端，并且需要（至少一个）服务URL来定位对等方.如果您不提供该服务，该服务将运行并正常运行，但它会使您的日志充满了无法向对等方注册的大量噪音.

另请参阅客户端的[below for details of Ribbon support](multi_spring-cloud-ribbon.html)区域和区域.

## 12.4独立模式

两个缓存（客户端和服务器）和心跳的组合使得独立的Eureka服务器可以非常适应故障，只要有某种监视器或弹性运行时（例如Cloud Foundry）使其保持活动状态.在独立模式下，您可能更愿意关闭客户端行为，以便它不会继续尝试并且无法访问其对等方.以下示例显示了如何切换客户端行为：

**application.yml (Standalone Eureka Server).** 

```java
server:
port: 8761

eureka:
instance:
hostname: localhost
client:
registerWithEureka: false
fetchRegistry: false
serviceUrl:
defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

请注意， `serviceUrl` 指向与本地实例相同的主机.

## 12.5同伴意识

通过运行多个实例并要求它们相互注册，可以使Eureka更具弹性和可用性.实际上，这是默认行为，因此要使其工作所需要做的就是向对等方添加有效的 `serviceUrl` ，如以下示例所示：

**application.yml (Two Peer Aware Eureka Servers).** 

```java
---
spring:
profiles: peer1
eureka:
instance:
hostname: peer1
client:
serviceUrl:
defaultZone: http://peer2/eureka/

---
spring:
profiles: peer2
eureka:
instance:
hostname: peer2
client:
serviceUrl:
defaultZone: http://peer1/eureka/
```

在前面的示例中，我们有一个YAML文件，可以通过在不同的Spring配置文件中运行它来在两个主机（ `peer1` 和 `peer2` ）上运行相同的服务器.您可以通过操作 `/etc/hosts` 来解析主机名，使用此配置来测试单个主机上的对等感知（在生产环境中执行此操作没有太大Value）.实际上，如果您在知道自己的主机名的计算机上运行，则不需要 `eureka.instance.hostname` （默认情况下，使用 `java.net.InetAddress` 查找它）.

您可以向系统添加多个对等体，并且只要它们通过至少一个边缘彼此连接，它们就会在它们之间同步注册.如果对等体在物理上是分开的（在数据中心内或在多个数据中心之间），那么系统原则上可以存在“裂脑”类型的故障.您可以向系统添加多个对等体，只要它们彼此直接连接，它们就会在各自之间同步注册.

**application.yml (Three Peer Aware Eureka Servers).** 

```java
eureka:
client:
serviceUrl:
defaultZone: http://peer1/eureka/,http://peer2/eureka/,http://peer3/eureka/

---
spring:
profiles: peer1
eureka:
instance:
hostname: peer1

---
spring:
profiles: peer2
eureka:
instance:
hostname: peer2

---
spring:
profiles: peer3
eureka:
instance:
hostname: peer3
```

## 12.6何时首选IP地址

在某些情况下，Eureka最好通告服务的IP地址而不是主机名.将 `eureka.instance.preferIpAddress` 设置为 `true` ，并且当应用程序向eureka注册时，它使用其IP地址而不是其主机名.

> 如果Java无法确定主机名，则将IP地址发送给Eureka.只有设置主机名的明确方法是设置 `eureka.instance.hostname` 属性.您可以使用环境变量在运行时设置主机名 - 例如， `eureka.instance.hostname=${HOST_NAME}` .

## 12.7保护Eureka服务器

只需将Spring Security添加到服务器的类路径中，即可通过 `spring-boot-starter-security` 保护您的Eureka服务器.默认情况下，当Spring Security位于类路径上时，它将要求每次向应用程序发送请求时都会发送一个有效的CSRF令牌. Eureka客户端通常不会拥有有效的跨站点请求伪造（CSRF）令牌，您需要为 `/eureka/**` endpoints禁用此要求.例如：

```java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

@Override
protected void configure(HttpSecurity http) throws Exception {
http.csrf().ignoringAntMatchers("/eureka/**");
super.configure(http);
}
}
```

有关CSRF的更多信息，请参阅[Spring Security documentation](https://docs.spring.io/spring-security/site/docs/current/reference/htmlsingle/#csrf).

可以在Spring Cloud Samples [repo](https://github.com/spring-cloud-samples/eureka/tree/Eureka-With-Security)中找到演示Eureka Server.


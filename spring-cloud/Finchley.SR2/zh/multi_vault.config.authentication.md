## 103.验证方法

不同组织对安全性和身份验证有不同的要求. Vault通过提供多种身份验证方法来反映这种需求Spring Cloud Vault支持令牌和AppId身份验证.

## 103.1令牌认证

令牌是Vault中验证的核心方法.令牌认证需要使用[Bootstrap Application Context](https://github.com/spring-cloud/spring-cloud-commons/blob/master/docs/src/main/asciidoc/spring-cloud-commons.adoc#the-bootstrap-application-context)提供静态令牌.

> Token身份验证是默认的身份验证方法.如果披露了令牌，则非预期的一方可以访问Vault并可以访问目标客户端的机密.

**Example 103.1. bootstrap.yml** 

```java
spring.cloud.vault:
authentication: TOKEN
token: 00000000-0000-0000-0000-000000000000
```

-  `authentication` 将此值设置为 `TOKEN` 选择令牌验证方法

-  `token` 设置要使用的静态令牌

另见：[Vault Documentation: Tokens](https://www.vaultproject.io/docs/concepts/tokens.html)

## 103.2 AppId身份验证

Vault支持[AppId](https://www.vaultproject.io/docs/auth/app-id.html)身份验证，该身份验证由两个难以猜测的令牌组成. AppId默认为静态配置的 `spring.application.name` .第二个标记是UserId，它是由应用程序确定的部分，通常与运行时环境相关. IP地址，Mac地址或Docker容器名称就是很好的例子. Spring Cloud Vault Config支持IP地址，Mac地址和静态UserId（例如，通过系统属性提供）. IP和Mac地址表示为十六进制编码的SHA256哈希.

基于IP地址的UserId使用本地主机的IP地址.

**Example 103.2. bootstrap.yml using SHA256 IP-Address UserId’s** 

```java
spring.cloud.vault:
authentication: APPID
app-id:
user-id: IP_ADDRESS
```

-  `authentication` 将此值设置为 `APPID` 选择AppId身份验证方法

-  `app-id-path` 设置要使用的AppId挂载的路径

-  `user-id` 设置UserId方法.可能的值为 `IP_ADDRESS` ， `MAC_ADDRESS` 或实现自定义 `AppIdUserIdMechanism` 的类名

从命令行生成IP地址UserId的相应命令是：

```java
$ echo -n 192.168.99.1 | sha256sum
```

> 包括 `echo` 的换行符导致不同的散列值，因此请确保包含 `-n` 标志.

基于Mac地址的UserId从本地主机绑定设备获取其网络设备.配置还允许指定 `network-interface` 提示以选择正确的设备.  `network-interface` 的值是可选的，可以是接口名称或接口索引（从0开始）.

**Example 103.3. bootstrap.yml using SHA256 Mac-Address UserId’s** 

```java
spring.cloud.vault:
authentication: APPID
app-id:
user-id: MAC_ADDRESS
network-interface: eth0
```

-  `network-interface` 设置网络接口以获取物理地址

从命令行生成IP地址UserId的相应命令是：

```java
$ echo -n 0AFEDE1234AC | sha256sum
```

>  Mac地址指定为大写且没有冒号.包括 `echo` 的换行符会导致不同的散列值，因此请确保包含 `-n` 标志.

### 103.2.1自定义UserId

UserId生成是一种开放机制.您可以将 `spring.cloud.vault.app-id.user-id` 设置为任何字符串，配置的值将用作静态UserId.

更高级的方法允许您将 `spring.cloud.vault.app-id.user-id` 设置为类名.此类必须位于类路径上，并且必须实现 `org.springframework.cloud.vault.AppIdUserIdMechanism` 接口和 `createUserId` 方法. Spring Cloud Vault将获取UserId每次使用AppId进行身份验证以获取令牌时调用 `createUserId` .

**Example 103.4. bootstrap.yml** 

```java
spring.cloud.vault:
authentication: APPID
app-id:
user-id: com.examlple.MyUserIdMechanism
```

**Example 103.5. MyUserIdMechanism.java** 

```java
public class MyUserIdMechanism implements AppIdUserIdMechanism {

@Override
public String createUserId() {
String userId = ...
return userId;
}
}
```

另见：[Vault Documentation: Using the App ID auth backend](https://www.vaultproject.io/docs/auth/app-id.html)

## 103.3 AppRole身份验证

[AppRole](https://www.vaultproject.io/docs/auth/app-id.html)用于计算机身份验证，例如已弃用（自Vault 0.6.1）[Section 103.2, “AppId authentication”](multi_vault.config.authentication.html#vault.config.authentication.appid). AppRole身份验证由两个难以猜测（秘密）令牌组成：RoleId和SecretId.

Spring Vault支持各种AppRole场景（推/拉模式和包装）.

RoleId和可选的SecretId必须由配置提供，Spring Vault不会查找这些或创建自定义SecretId.

**Example 103.6. bootstrap.yml with AppRole authentication properties** 

```java
spring.cloud.vault:
authentication: APPROLE
app-role:
role-id: bde2076b-cccb-3cf0-d57e-bca7b1e83a52
```

所需的配置详细信息支持以下方案：

**Table 103.1. Configuration** 

|  **Method**  |  **RoleId**  |  **SecretId**  |  **RoleName**  |  **Token**  |
| ---- | ---- | ---- | ---- | ---- |
|提供RoleId / SecretId |提供|提供| | |
|提供没有SecretId的RoleId |提供| | | |
|提供RoleId，Pull SecretId |提供|提供|提供|提供|
|拉动RoleId，提供SecretId | |提供|提供|提供|
|全拉模式| | |提供|提供|
|包裹| | | |提供|
| Wrapped RoleId，提供了SecretId |提供| | |提供|
|提供RoleId，包装SecretId | |提供| |提供|

**Table 103.2. Pull/Push/Wrapped Matrix** 

|  **RoleId**  |  **SecretId**  |  **Supported**  |
| ---- | ---- | ---- |
|提供|提供|✅|
|提供|拉|✅|
|提供|包裹|✅|
|提供|缺席|✅|
|拉|提供|✅|
|拉|拉|✅|
|拉|包裹|❌|
|拉|缺席|❌|
|包裹|提供|✅|
|包裹|拉|❌|
|包裹|包裹|✅|
|包裹|缺席|❌|

> 您可以通过在引导程序上下文中提供已配置的 `AppRoleAuthentication`  bean来使用推/拉/包裹模式的所有组合. Spring Cloud Vault无法从配置属性派生所有可能的AppRole组合.

|图片/ important.png |重要|
| ---- | ---- |
| AppRole身份验证仅限于使用反应式基础架构的简单拉模式.尚不支持全拉模式.将Spring Cloud Vault与Spring WebFlux堆栈配合使用可启用Vault的反应式自动配置，可通过设置 `spring.cloud.vault.reactive.enabled=false` 来禁用. |

**Example 103.7. bootstrap.yml with all AppRole authentication properties** 

```java
spring.cloud.vault:
authentication: APPROLE
app-role:
role-id: bde2076b-cccb-3cf0-d57e-bca7b1e83a52
secret-id: 1696536f-1976-73b1-b241-0b4213908d39
role: my-role
app-role-path: approle
```

-  `role-id` 设置RoleId.

-  `secret-id` 设置了SecretId.如果配置AppRole而不需要SecretId，则可以省略SecretId（请参阅 `bind_secret_id` ）.

-  `role` ：为拉模式设置AppRole名称.

-  `app-role-path` 设置要使用的审批认证安装的路径.

另见：[Vault Documentation: Using the AppRole auth backend](https://www.vaultproject.io/docs/auth/approle.html)

## 103.4 AWS-EC2身份验证

[aws-ec2](https://www.vaultproject.io/docs/auth/aws-ec2.html) auth后端为AWS EC2实例提供安全的引入机制，允许自动检索Vault令牌.与大多数Vault身份验证后端不同，此后端不需要首先部署或配置安全敏感凭据（令牌，用户名/密码，客户端证书等）.相反，它将AWS视为可信第三方，并使用唯一表示每个EC2实例的加密签名动态元数据信息.

**Example 103.8. bootstrap.yml using AWS-EC2 Authentication** 

```java
spring.cloud.vault:
authentication: AWS_EC2
```

AWS-EC2身份验证默认情况下启用nonce遵循首次使用信任使用（TOFU）原则.任何能够访问PKCS＃7身份元数据的非预期方都可以对Vault进行身份验证.

在首次登录期间，Spring Cloud Vault会生成一个存储在auth后端的nonce，而不是实例Id.重新认证需要发送相同的随机数.任何其他方都没有nonce，可以在Vault中发出警报以进行进一步调查.

随机数将保留在内存中，并在应用程序重新启动时丢失.您可以使用 `spring.cloud.vault.aws-ec2.nonce` 配置静态随机数.

AWS-EC2身份验证角色是可选的，默认为AMI.您可以通过设置 `spring.cloud.vault.aws-ec2.role` 属性来配置身份验证角色.

**Example 103.9. bootstrap.yml with configured role** 

```java
spring.cloud.vault:
authentication: AWS_EC2
aws-ec2:
role: application-server
```

**Example 103.10. bootstrap.yml with all AWS EC2 authentication properties** 

```java
spring.cloud.vault:
authentication: AWS_EC2
aws-ec2:
role: application-server
aws-ec2-path: aws-ec2
identity-document: http://...
nonce: my-static-nonce
```

-  `authentication` 将此值设置为 `AWS_EC2` 选择AWS EC2身份验证方法

-  `role` 设置尝试登录的角色的名称.

-  `aws-ec2-path` 设置要使用的AWS EC2装载的路径

-  `identity-document` 设置PKCS＃7 AWS EC2身份文档的URL

-  `nonce` 用于AWS-EC2身份验证.空nonce默认为nonce生成

另见：[Vault Documentation: Using the aws auth backend](https://www.vaultproject.io/docs/auth/aws.html)

## 103.5 AWS-IAM身份验证

[aws](https://www.vaultproject.io/docs/auth/aws-ec2.html)后端为AWS IAM角色提供安全的身份验证机制，允许基于正在运行的应用程序的当前IAM角色对Vault进行自动身份验证.与大多数Vault身份验证后端不同，此后端不需要首先部署或配置安全敏感凭据（令牌，用户名/密码，客户端证书等）.相反，它将AWS视为可信第三方，并使用调用方签署的4条信息及其IAM凭据来验证调用方确实正在使用该IAM角色.

将自动计算运行应用程序的当前IAM角色.如果您在AWS ECS上运行应用程序，则应用程序将使用分配给正在运行的容器的ECS任务的IAM角色.如果您在EC2实例上运行应用程序，那么就是IAM使用的角色将是分配给EC2实例的角色.

使用AWS-IAM身份验证时，您必须在Vault中创建角色并将其分配给您的IAM角色.空 `role` 默认为当前IAM角色的友好名称.

**Example 103.11. bootstrap.yml with required AWS-IAM Authentication properties** 

```java
spring.cloud.vault:
authentication: AWS_IAM
```

**Example 103.12. bootstrap.yml with all AWS-IAM Authentication properties** 

```java
spring.cloud.vault:
authentication: AWS_IAM
aws-iam:
role: my-dev-role
aws-path: aws
server-id: some.server.name
```

-  `role` 设置尝试登录的角色的名称.这应该与您的IAM角色绑定.如果未提供，则将使用当前IAM用户的友好名称作为库角色.

-  `aws-path` 设置要使用的AWS挂载的路径

-  `server-id` 设置用于 `X-Vault-AWS-IAM-Server-ID` 标头的值，以防止某些类型的重播攻击.

AWS-IAM需要AWS Java SDK依赖关系（ `com.amazonaws:aws-java-sdk-core` ），因为身份验证实现使用AWS SDK类型进行凭据和请求签名.

另见：[Vault Documentation: Using the aws auth backend](https://www.vaultproject.io/docs/auth/aws.html)

## 103.6 TLS证书身份验证

`cert`  auth后端允许使用由CA签名或自签名的SSL / TLS客户端证书进行身份验证.

要启用 `cert` 身份验证，您需要：

使用SSL，请参阅第109章，Vault客户端SSL配置配置包含客户端证书和私钥的Java密钥库将spring.cloud.vault.authentication设置为CERT

**Example 103.13. bootstrap.yml** 

```java
spring.cloud.vault:
authentication: CERT
ssl:
key-store: classpath:keystore.jks
key-store-password: changeit
cert-auth-path: cert
```

另见：[Vault Documentation: Using the Cert auth backend](https://www.vaultproject.io/docs/auth/cert.html)

## 103.7 Cubbyhole认证

Cubbyhole身份验证使用Vault原语来提供安全的身份验证工作流. Cubbyhole身份验证使用令牌作为主要登录方法.一个短暂的令牌用于从Vault的Cubbyhole秘密后端获取第二个登录VaultToken.登录令牌通常寿命较长，用于与Vault进行交互.登录令牌将从存储在 `/cubbyhole/response` 的包装响应中检索.

**Creating a wrapped token** 

> Response包装创建令牌需要Vault 0.6.0或更高版本.

**Example 103.14. Creating and storing tokens** 

```java
$ vault token-create -wrap-ttl="10m"
Key                            Value
---                            -----
wrapping_token:                397ccb93-ff6c-b17b-9389-380b01ca2645
wrapping_token_ttl:            0h10m0s
wrapping_token_creation_time:  2016-09-18 20:29:48.652957077 +0200 CEST
wrapped_accessor:              46b6aebb-187f-932a-26d7-4f3d86a68319
```

**Example 103.15. bootstrap.yml** 

```java
spring.cloud.vault:
authentication: CUBBYHOLE
token: 397ccb93-ff6c-b17b-9389-380b01ca2645
```

也可以看看：

- [Vault Documentation: Tokens](https://www.vaultproject.io/docs/concepts/tokens.html)

- [Vault Documentation: Cubbyhole Secret Backend](https://www.vaultproject.io/docs/secrets/cubbyhole/index.html)

- [Vault Documentation: Response Wrapping](https://www.vaultproject.io/docs/concepts/response-wrapping.html)

## 103.8 Kubernetes认证

Kubernetes身份验证机制（自Vault 0.8.3起）允许使用Kubernetes服务帐户令牌对Vault进行身份验证.身份验证基于角色，角色绑定到服务帐户名称和命名空间.

包含pod的服务帐户的JWT令牌的文件将自动挂载在 `/var/run/secrets/kubernetes.io/serviceaccount/token` .

**Example 103.16. bootstrap.yml with all Kubernetes authentication properties** 

```java
spring.cloud.vault:
authentication: KUBERNETES
kubernetes:
role: my-dev-role
service-account-token-file: /var/run/secrets/kubernetes.io/serviceaccount/token
```

-  `role` 设置角色.

-  `service-account-token-file` 设置包含Kubernetes服务帐户令牌的文件的位置.默认为 `/var/run/secrets/kubernetes.io/serviceaccount/token` .

也可以看看：

- [Vault Documentation: Kubernetes](https://www.vaultproject.io/docs/auth/kubernetes.html)

- [Kubernetes Documentation: Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)


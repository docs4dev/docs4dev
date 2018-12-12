## 103. Authentication methods

Different organizations have different requirements for security and authentication. Vault reflects that need by shipping multiple authentication methods. Spring Cloud Vault supports token and AppId authentication.

## 103.1 Token authentication

Tokens are the core method for authentication within Vault. Token authentication requires a static token to be provided using the [Bootstrap Application Context](https://github.com/spring-cloud/spring-cloud-commons/blob/master/docs/src/main/asciidoc/spring-cloud-commons.adoc#the-bootstrap-application-context).

> Token authentication is the default authentication method. If a token is disclosed an unintended party gains access to Vault and can access secrets for the intended client.

**Example 103.1. bootstrap.yml** 

```java
spring.cloud.vault:
authentication: TOKEN
token: 00000000-0000-0000-0000-000000000000
```

-  `authentication`  setting this value to  `TOKEN`  selects the Token authentication method

-  `token`  sets the static token to use

See also: [Vault Documentation: Tokens](https://www.vaultproject.io/docs/concepts/tokens.html)

## 103.2 AppId authentication

Vault supports [AppId](https://www.vaultproject.io/docs/auth/app-id.html) authentication that consists of two hard to guess tokens. The AppId defaults to  `spring.application.name`  that is statically configured. The second token is the UserId which is a part determined by the application, usually related to the runtime environment. IP address, Mac address or a Docker container name are good examples. Spring Cloud Vault Config supports IP address, Mac address and static UserId’s (e.g. supplied via System properties). The IP and Mac address are represented as Hex-encoded SHA256 hash.

IP address-based UserId’s use the local host’s IP address.

**Example 103.2. bootstrap.yml using SHA256 IP-Address UserId’s** 

```java
spring.cloud.vault:
authentication: APPID
app-id:
user-id: IP_ADDRESS
```

-  `authentication`  setting this value to  `APPID`  selects the AppId authentication method

-  `app-id-path`  sets the path of the AppId mount to use

-  `user-id`  sets the UserId method. Possible values are  `IP_ADDRESS` ,  `MAC_ADDRESS`  or a class name implementing a custom  `AppIdUserIdMechanism` 

The corresponding command to generate the IP address UserId from a command line is:

```java
$ echo -n 192.168.99.1 | sha256sum
```

> Including the line break of  `echo`  leads to a different hash value so make sure to include the  `-n`  flag.

Mac address-based UserId’s obtain their network device from the localhost-bound device. The configuration also allows specifying a  `network-interface`  hint to pick the right device. The value of  `network-interface`  is optional and can be either an interface name or interface index (0-based).

**Example 103.3. bootstrap.yml using SHA256 Mac-Address UserId’s** 

```java
spring.cloud.vault:
authentication: APPID
app-id:
user-id: MAC_ADDRESS
network-interface: eth0
```

-  `network-interface`  sets network interface to obtain the physical address

The corresponding command to generate the IP address UserId from a command line is:

```java
$ echo -n 0AFEDE1234AC | sha256sum
```

> The Mac address is specified uppercase and without colons. Including the line break of  `echo`  leads to a different hash value so make sure to include the  `-n`  flag.

### 103.2.1 Custom UserId

The UserId generation is an open mechanism. You can set  `spring.cloud.vault.app-id.user-id`  to any string and the configured value will be used as static UserId.

A more advanced approach lets you set  `spring.cloud.vault.app-id.user-id`  to a classname. This class must be on your classpath and must implement the  `org.springframework.cloud.vault.AppIdUserIdMechanism`  interface and the  `createUserId`  method. Spring Cloud Vault will obtain the UserId by calling  `createUserId`  each time it authenticates using AppId to obtain a token.

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

See also: [Vault Documentation: Using the App ID auth backend](https://www.vaultproject.io/docs/auth/app-id.html)

## 103.3 AppRole authentication

[AppRole](https://www.vaultproject.io/docs/auth/app-id.html) is intended for machine authentication, like the deprecated (since Vault 0.6.1) [Section 103.2, “AppId authentication”](multi_vault.config.authentication.html#vault.config.authentication.appid). AppRole authentication consists of two hard to guess (secret) tokens: RoleId and SecretId.

Spring Vault supports various AppRole scenarios (push/pull mode and wrapped).

RoleId and optionally SecretId must be provided by configuration, Spring Vault will not look up these or create a custom SecretId.

**Example 103.6. bootstrap.yml with AppRole authentication properties** 

```java
spring.cloud.vault:
authentication: APPROLE
app-role:
role-id: bde2076b-cccb-3cf0-d57e-bca7b1e83a52
```

The following scenarios are supported along the required configuration details:

**Table 103.1. Configuration** 

| **Method**  | **RoleId**  | **SecretId**  | **RoleName**  | **Token**  |
|----|----|----|----|----|
|Provided RoleId/SecretId |Provided |Provided | | |
|Provided RoleId without SecretId |Provided | | | |
|Provided RoleId, Pull SecretId |Provided |Provided |Provided |Provided |
|Pull RoleId, provided SecretId | |Provided |Provided |Provided |
|Full Pull Mode | | |Provided |Provided |
|Wrapped | | | |Provided |
|Wrapped RoleId, provided SecretId |Provided | | |Provided |
|Provided RoleId, wrapped SecretId | |Provided | |Provided |

**Table 103.2. Pull/Push/Wrapped Matrix** 

| **RoleId**  | **SecretId**  | **Supported**  |
|----|----|----|
|Provided |Provided |✅ |
|Provided |Pull |✅ |
|Provided |Wrapped |✅ |
|Provided |Absent |✅ |
|Pull |Provided |✅ |
|Pull |Pull |✅ |
|Pull |Wrapped |❌ |
|Pull |Absent |❌ |
|Wrapped |Provided |✅ |
|Wrapped |Pull |❌ |
|Wrapped |Wrapped |✅ |
|Wrapped |Absent |❌ |

> You can use still all combinations of push/pull/wrapped modes by providing a configured  `AppRoleAuthentication`  bean within the bootstrap context. Spring Cloud Vault cannot derive all possible AppRole combinations from the configuration properties.

|images/important.png|Important|
|----|----|
|AppRole authentication is limited to simple pull mode using reactive infrastructure. Full pull mode is not yet supported. Using Spring Cloud Vault with the Spring WebFlux stack enables Vault’s reactive auto-configuration which can be disabled by setting  `spring.cloud.vault.reactive.enabled=false` . |

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

-  `role-id`  sets the RoleId.

-  `secret-id`  sets the SecretId. SecretId can be omitted if AppRole is configured without requiring SecretId (See  `bind_secret_id` ).

-  `role` : sets the AppRole name for pull mode.

-  `app-role-path`  sets the path of the approle authentication mount to use.

See also: [Vault Documentation: Using the AppRole auth backend](https://www.vaultproject.io/docs/auth/approle.html)

## 103.4 AWS-EC2 authentication

The [aws-ec2](https://www.vaultproject.io/docs/auth/aws-ec2.html) auth backend provides a secure introduction mechanism for AWS EC2 instances, allowing automated retrieval of a Vault token. Unlike most Vault authentication backends, this backend does not require first-deploying, or provisioning security-sensitive credentials (tokens, username/password, client certificates, etc.). Instead, it treats AWS as a Trusted Third Party and uses the cryptographically signed dynamic metadata information that uniquely represents each EC2 instance.

**Example 103.8. bootstrap.yml using AWS-EC2 Authentication** 

```java
spring.cloud.vault:
authentication: AWS_EC2
```

AWS-EC2 authentication enables nonce by default to follow the Trust On First Use (TOFU) principle. Any unintended party that gains access to the PKCS#7 identity metadata can authenticate against Vault.

During the first login, Spring Cloud Vault generates a nonce that is stored in the auth backend aside the instance Id. Re-authentication requires the same nonce to be sent. Any other party does not have the nonce and can raise an alert in Vault for further investigation.

The nonce is kept in memory and is lost during application restart. You can configure a static nonce with  `spring.cloud.vault.aws-ec2.nonce` .

AWS-EC2 authentication roles are optional and default to the AMI. You can configure the authentication role by setting the  `spring.cloud.vault.aws-ec2.role`  property.

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

-  `authentication`  setting this value to  `AWS_EC2`  selects the AWS EC2 authentication method

-  `role`  sets the name of the role against which the login is being attempted.

-  `aws-ec2-path`  sets the path of the AWS EC2 mount to use

-  `identity-document`  sets URL of the PKCS#7 AWS EC2 identity document

-  `nonce`  used for AWS-EC2 authentication. An empty nonce defaults to nonce generation

See also: [Vault Documentation: Using the aws auth backend](https://www.vaultproject.io/docs/auth/aws.html)

## 103.5 AWS-IAM authentication

The [aws](https://www.vaultproject.io/docs/auth/aws-ec2.html) backend provides a secure authentication mechanism for AWS IAM roles, allowing the automatic authentication with vault based on the current IAM role of the running application. Unlike most Vault authentication backends, this backend does not require first-deploying, or provisioning security-sensitive credentials (tokens, username/password, client certificates, etc.). Instead, it treats AWS as a Trusted Third Party and uses the 4 pieces of information signed by the caller with their IAM credentials to verify that the caller is indeed using that IAM role.

The current IAM role the application is running in is automatically calculated. If you are running your application on AWS ECS then the application will use the IAM role assigned to the ECS task of the running container. If you are running your application naked on top of an EC2 instance then the IAM role used will be the one assigned to the EC2 instance.

When using the AWS-IAM authentication you must create a role in Vault and assign it to your IAM role. An empty  `role`  defaults to the friendly name the current IAM role.

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

-  `role`  sets the name of the role against which the login is being attempted. This should be bound to your IAM role. If one is not supplied then the friendly name of the current IAM user will be used as the vault role.

-  `aws-path`  sets the path of the AWS mount to use

-  `server-id`  sets the value to use for the  `X-Vault-AWS-IAM-Server-ID`  header preventing certain types of replay attacks.

AWS-IAM requires the AWS Java SDK dependency ( `com.amazonaws:aws-java-sdk-core` ) as the authentication implementation uses AWS SDK types for credentials and request signing.

See also: [Vault Documentation: Using the aws auth backend](https://www.vaultproject.io/docs/auth/aws.html)

## 103.6 TLS certificate authentication

The  `cert`  auth backend allows authentication using SSL/TLS client certificates that are either signed by a CA or self-signed.

To enable  `cert`  authentication you need to:

Use SSL, see Chapter 109, Vault Client SSL configuration Configure a Java Keystore that contains the client certificate and the private key Set the spring.cloud.vault.authentication to CERT

**Example 103.13. bootstrap.yml** 

```java
spring.cloud.vault:
authentication: CERT
ssl:
key-store: classpath:keystore.jks
key-store-password: changeit
cert-auth-path: cert
```

See also: [Vault Documentation: Using the Cert auth backend](https://www.vaultproject.io/docs/auth/cert.html)

## 103.7 Cubbyhole authentication

Cubbyhole authentication uses Vault primitives to provide a secured authentication workflow. Cubbyhole authentication uses tokens as primary login method. An ephemeral token is used to obtain a second, login VaultToken from Vault’s Cubbyhole secret backend. The login token is usually longer-lived and used to interact with Vault. The login token will be retrieved from a wrapped response stored at  `/cubbyhole/response` .

**Creating a wrapped token** 

> Response Wrapping for token creation requires Vault 0.6.0 or higher.

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

See also:

- [Vault Documentation: Tokens](https://www.vaultproject.io/docs/concepts/tokens.html)

- [Vault Documentation: Cubbyhole Secret Backend](https://www.vaultproject.io/docs/secrets/cubbyhole/index.html)

- [Vault Documentation: Response Wrapping](https://www.vaultproject.io/docs/concepts/response-wrapping.html)

## 103.8 Kubernetes authentication

Kubernetes authentication mechanism (since Vault 0.8.3) allows to authenticate with Vault using a Kubernetes Service Account Token. The authentication is role based and the role is bound to a service account name and a namespace.

A file containing a JWT token for a pod’s service account is automatically mounted at  `/var/run/secrets/kubernetes.io/serviceaccount/token` .

**Example 103.16. bootstrap.yml with all Kubernetes authentication properties** 

```java
spring.cloud.vault:
authentication: KUBERNETES
kubernetes:
role: my-dev-role
service-account-token-file: /var/run/secrets/kubernetes.io/serviceaccount/token
```

-  `role`  sets the Role.

-  `service-account-token-file`  sets the location of the file containing the Kubernetes Service Account Token. Defaults to  `/var/run/secrets/kubernetes.io/serviceaccount/token` .

See also:

- [Vault Documentation: Kubernetes](https://www.vaultproject.io/docs/auth/kubernetes.html)

- [Kubernetes Documentation: Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)


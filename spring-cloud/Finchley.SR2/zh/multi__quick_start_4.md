## 101.快速入门

**Prerequisites** 

要开始使用Vault和本指南，您需要一个类似* NIX的操作系统，它提供：

-  `wget` ， `openssl` 和 `unzip` 

- at至少Java 7和正确配置的 `JAVA_HOME` 环境变量

**Install Vault** 

```java
$ src/test/bash/install_vault.sh
```

**Create SSL certificates for Vault** 

```java
$ src/test/bash/create_certificates.sh
```

>  `create_certificates.sh` 在 `work/ca` 和JKS信任库 `work/keystore.jks` 中创建证书.如果要使用此快速入门指南运行Spring Cloud Vault，则需要将信任库的 `spring.cloud.vault.ssl.trust-store` 属性配置为 `file:work/keystore.jks` .

**Start Vault server** 

```java
$ src/test/bash/local_run_vault.sh
```

Vault使用 `inmem` 存储和 `https` 开始侦听 `0.0.0.0:8200` .保险柜已密封，启动时未初始化.

> 如果要运行测试，请保持Vault未初始化.测试将初始化Vault并创建根令牌 `00000000-0000-0000-0000-000000000000` .

如果您想将Vault用于您的应用程序或试一试，那么您需要先将其初始化.

```java
$ export VAULT_ADDR="https://localhost:8200"
$ export VAULT_SKIP_VERIFY=true # Don't do this for production
$ vault init
```

你应该看到类似的东西：

```java
Key 1: 7149c6a2e16b8833f6eb1e76df03e47f6113a3288b3093faf5033d44f0e70fe701
Key 2: 901c534c7988c18c20435a85213c683bdcf0efcd82e38e2893779f152978c18c02
Key 3: 03ff3948575b1165a20c20ee7c3e6edf04f4cdbe0e82dbff5be49c63f98bc03a03
Key 4: 216ae5cc3ddaf93ceb8e1d15bb9fc3176653f5b738f5f3d1ee00cd7dccbe926e04
Key 5: b2898fc8130929d569c1677ee69dc5f3be57d7c4b494a6062693ce0b1c4d93d805
Initial Root Token: 19aefa97-cccc-bbbb-aaaa-225940e63d76

Vault initialized with 5 keys and a key threshold of 3. Please
securely distribute the above keys. When the Vault is re-sealed,
restarted, or stopped, you must provide at least 3 of these keys
to unseal it again.

Vault does not store the master key. Without at least 3 keys,
your Vault will remain permanently sealed.
```

Vault将初始化并返回一组解封密钥和根令牌.选择3个密钥并启动Vault.将Vault令牌存储在 `VAULT_TOKEN` 环境变量中.

```java
$ vault unseal (Key 1)
$ vault unseal (Key 2)
$ vault unseal (Key 3)
$ export VAULT_TOKEN=(Root token)
# Required to run Spring Cloud Vault tests after manual initialization
$ vault token-create -id="00000000-0000-0000-0000-000000000000" -policy="root"
```

Spring Cloud Vault可访问不同的资源.默认情况下，启用秘密后端，通过JSONendpoints访问秘密配置设置.

HTTP服务具有以下形式的资源：

```java
/secret/{application}/{profile}
/secret/{application}
/secret/{defaultContext}/{profile}
/secret/{defaultContext}
```

"application"在 `SpringApplication` 中被注入 `spring.application.name` （即常规Spring Boot应用程序中通常为"application"），"profile"是一个活动的配置文件（或以逗号分隔的属性列表）.从Vault检索到的属性将不再使用"as-is"属性名称的前缀.

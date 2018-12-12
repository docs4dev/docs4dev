## 101. Quick Start

**Prerequisites** 

To get started with Vault and this guide you need a *NIX-like operating systems that provides:

-  `wget` ,  `openssl`  and  `unzip` 

- at least Java 7 and a properly configured  `JAVA_HOME`  environment variable

**Install Vault** 

```java
$ src/test/bash/install_vault.sh
```

**Create SSL certificates for Vault** 

```java
$ src/test/bash/create_certificates.sh
```

>  `create_certificates.sh`  creates certificates in  `work/ca`  and a JKS truststore  `work/keystore.jks` . If you want to run Spring Cloud Vault using this quickstart guide you need to configure the truststore the  `spring.cloud.vault.ssl.trust-store`  property to  `file:work/keystore.jks` .

**Start Vault server** 

```java
$ src/test/bash/local_run_vault.sh
```

Vault is started listening on  `0.0.0.0:8200`  using the  `inmem`  storage and  `https` . Vault is sealed and not initialized when starting up.

> If you want to run tests, leave Vault uninitialized. The tests will initialize Vault and create a root token  `00000000-0000-0000-0000-000000000000` .

If you want to use Vault for your application or give it a try then you need to initialize it first.

```java
$ export VAULT_ADDR="https://localhost:8200"
$ export VAULT_SKIP_VERIFY=true # Don't do this for production
$ vault init
```

You should see something like:

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

Vault will initialize and return a set of unsealing keys and the root token. Pick 3 keys and unseal Vault. Store the Vault token in the  `VAULT_TOKEN`  environment variable.

```java
$ vault unseal (Key 1)
$ vault unseal (Key 2)
$ vault unseal (Key 3)
$ export VAULT_TOKEN=(Root token)
# Required to run Spring Cloud Vault tests after manual initialization
$ vault token-create -id="00000000-0000-0000-0000-000000000000" -policy="root"
```

Spring Cloud Vault accesses different resources. By default, the secret backend is enabled which accesses secret config settings via JSON endpoints.

The HTTP service has resources in the form:

```java
/secret/{application}/{profile}
/secret/{application}
/secret/{defaultContext}/{profile}
/secret/{defaultContext}
```

where the "application" is injected as the  `spring.application.name`  in the  `SpringApplication`  (i.e. what is normally "application" in a regular Spring Boot app), "profile" is an active profile (or comma-separated list of properties). Properties retrieved from Vault will be used "as-is" without further prefixing of the property names.

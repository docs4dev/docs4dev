## 5. Spring Cloud Config Server

Spring Cloud Config Server provides an HTTP resource-based API for external configuration (name-value pairs or equivalent YAML content). The server is embeddable in a Spring Boot application, by using the  `@EnableConfigServer`  annotation. Consequently, the following application is a config server:

**ConfigServer.java.**  

```java
@SpringBootApplication
@EnableConfigServer
public class ConfigServer {
public static void main(String[] args) {
SpringApplication.run(ConfigServer.class, args);
}
}
```

Like all Spring Boot applications, it runs on port 8080 by default, but you can switch it to the more conventional port 8888 in various ways. The easiest, which also sets a default configuration repository, is by launching it with  `spring.config.name=configserver`  (there is a  `configserver.yml`  in the Config Server jar). Another is to use your own  `application.properties` , as shown in the following example:

**application.properties.**  

```java
server.port: 8888
spring.cloud.config.server.git.uri: file://${user.home}/config-repo
```

where  `${user.home}/config-repo`  is a git repository containing YAML and properties files.

> On Windows, you need an extra "/" in the file URL if it is absolute with a drive prefix (for example, `file:///${user.home}/config-repo` ).

> The following listing shows a recipe for creating the git repository in the preceding example:

> Using the local filesystem for your git repository is intended for testing only. You should use a server to host your configuration repositories in production.

> The initial clone of your configuration repository can be quick and efficient if you keep only text files in it. If you store binary files, especially large ones, you may experience delays on the first request for configuration or encounter out of memory errors in the server.

## 5.1 Environment Repository

Where should you store the configuration data for the Config Server? The strategy that governs this behaviour is the  `EnvironmentRepository` , serving  `Environment`  objects. This  `Environment`  is a shallow copy of the domain from the Spring  `Environment`  (including  `propertySources`  as the main feature). The  `Environment`  resources are parametrized by three variables:

-  `{application}` , which maps to  `spring.application.name`  on the client side.

-  `{profile}` , which maps to  `spring.profiles.active`  on the client (comma-separated list).

-  `{label}` , which is a server side feature labelling a "versioned" set of config files.

Repository implementations generally behave like a Spring Boot application, loading configuration files from a  `spring.config.name`  equal to the  `{application}`  parameter, and  `spring.profiles.active`  equal to the  `{profiles}`  parameter. Precedence rules for profiles are also the same as in a regular Spring Boot application: Active profiles take precedence over defaults, and, if there are multiple profiles, the last one wins (similar to adding entries to a  `Map` ).

The following sample client application has this bootstrap configuration:

**bootstrap.yml.**  

```java
spring:
application:
name: foo
profiles:
active: dev,mysql
```

(As usual with a Spring Boot application, these properties could also be set by environment variables or command line arguments).

If the repository is file-based, the server creates an  `Environment`  from  `application.yml`  (shared between all clients) and  `foo.yml`  (with  `foo.yml`  taking precedence). If the YAML files have documents inside them that point to Spring profiles, those are applied with higher precedence (in order of the profiles listed). If there are profile-specific YAML (or properties) files, these are also applied with higher precedence than the defaults. Higher precedence translates to a  `PropertySource`  listed earlier in the  `Environment` . (These same rules apply in a standalone Spring Boot application.)

You can set spring.cloud.config.server.accept-empty to false so that Server would return a HTTP 404 status, if the application is not found.By default, this flag is set to true.

### 5.1.1 Git Backend

The default implementation of  `EnvironmentRepository`  uses a Git backend, which is very convenient for managing upgrades and physical environments and for auditing changes. To change the location of the repository, you can set the  `spring.cloud.config.server.git.uri`  configuration property in the Config Server (for example in  `application.yml` ). If you set it with a  `file:`  prefix, it should work from a local repository so that you can get started quickly and easily without a server. However, in that case, the server operates directly on the local repository without cloning it (it does not matter if it is not bare because the Config Server never makes changes to the "remote" repository). To scale the Config Server up and make it highly available, you need to have all instances of the server pointing to the same repository, so only a shared file system would work. Even in that case, it is better to use the  `ssh:`  protocol for a shared filesystem repository, so that the server can clone it and use a local working copy as a cache.

This repository implementation maps the  `{label}`  parameter of the HTTP resource to a git label (commit id, branch name, or tag). If the git branch or tag name contains a slash ( `/` ), then the label in the HTTP URL should instead be specified with the special string  `(_)`  (to avoid ambiguity with other URL paths). For example, if the label is  `foo/bar` , replacing the slash would result in the following label:  `foo(_)bar` . The inclusion of the special string  `(_)`  can also be applied to the  `{application}`  parameter. If you use a command-line client such as curl, be careful with the brackets in the URL — you should escape them from the shell with single quotes ('').

#### Skipping SSL Certificate Validation

The configuration server’s validation of the Git server’s SSL certificate can be disabled by setting the  `git.skipSslValidation`  property to  `true`  (default is  `false` ).

```java
spring:
cloud:
config:
server:
git:
uri: https://example.com/my/repo
skipSslValidation: true
```

#### Setting HTTP Connection Timeout

You can configure the time, in seconds, that the configuration server will wait to acquire an HTTP connection. Use the  `git.timeout`  property.

```java
spring:
cloud:
config:
server:
git:
uri: https://example.com/my/repo
timeout: 4
```

#### Placeholders in Git URI

Spring Cloud Config Server supports a git repository URL with placeholders for the  `{application}`  and  `{profile}`  (and  `{label}`  if you need it, but remember that the label is applied as a git label anyway). So you can support a “one repository per application” policy by using a structure similar to the following:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/myorg/{application}
```

You can also support a “one repository per profile” policy by using a similar pattern but with  `{profile}` .

Additionally, using the special string "(_)" within your  `{application}`  parameters can enable support for multiple organizations, as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/{application}
```

where  `{application}`  is provided at request time in the following format:  `organization(_)application` .

#### Pattern Matching and Multiple Repositories

Spring Cloud Config also includes support for more complex requirements with pattern matching on the application and profile name. The pattern format is a comma-separated list of  `{application}/{profile}`  names with wildcards (note that a pattern beginning with a wildcard may need to be quoted), as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
repos:
simple: https://github.com/simple/config-repo
special:
pattern: special*/dev*,*special*/dev*
uri: https://github.com/special/config-repo
local:
pattern: local*
uri: file:/home/configsvc/config-repo
```

If  `{application}/{profile}`  does not match any of the patterns, it uses the default URI defined under  `spring.cloud.config.server.git.uri` . In the above example, for the “simple” repository, the pattern is  `simple/*`  (it only matches one application named  `simple`  in all profiles). The “local” repository matches all application names beginning with  `local`  in all profiles (the  `/*`  suffix is added automatically to any pattern that does not have a profile matcher).

> The “one-liner” short cut used in the “simple” example can be used only if the only property to be set is the URI. If you need to set anything else (credentials, pattern, and so on) you need to use the full form.

The  `pattern`  property in the repo is actually an array, so you can use a YAML array (or  `[0]` ,  `[1]` , etc. suffixes in properties files) to bind to multiple patterns. You may need to do so if you are going to run apps with multiple profiles, as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
repos:
development:
pattern:
- '*/development'
- '*/staging'
uri: https://github.com/development/config-repo
staging:
pattern:
- '*/qa'
- '*/production'
uri: https://github.com/staging/config-repo
```

> Spring Cloud guesses that a pattern containing a profile that does not end in  `*`  implies that you actually want to match a list of profiles starting with this pattern (so  `*/staging`  is a shortcut for  `["*/staging", "*/staging,*"]` , and so on). This is common where, for instance, you need to run applications in the “development” profile locally but also the “cloud” profile remotely.

Every repository can also optionally store config files in sub-directories, and patterns to search for those directories can be specified as  `searchPaths` . The following example shows a config file at the top level:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
searchPaths: foo,bar*
```

In the preceding example, the server searches for config files in the top level and in the  `foo/`  sub-directory and also any sub-directory whose name begins with  `bar` .

By default, the server clones remote repositories when configuration is first requested. The server can be configured to clone the repositories at startup, as shown in the following top-level example:

```java
spring:
cloud:
config:
server:
git:
uri: https://git/common/config-repo.git
repos:
team-a:
pattern: team-a-*
cloneOnStart: true
uri: http://git/team-a/config-repo.git
team-b:
pattern: team-b-*
cloneOnStart: false
uri: http://git/team-b/config-repo.git
team-c:
pattern: team-c-*
uri: http://git/team-a/config-repo.git
```

In the preceding example, the server clones team-a’s config-repo on startup, before it accepts any requests. All other repositories are not cloned until configuration from the repository is requested.

> Setting a repository to be cloned when the Config Server starts up can help to identify a misconfigured configuration source (such as an invalid repository URI) quickly, while the Config Server is starting up. With  `cloneOnStart`  not enabled for a configuration source, the Config Server may start successfully with a misconfigured or invalid configuration source and not detect an error until an application requests configuration from that configuration source.

#### Authentication

To use HTTP basic authentication on the remote repository, add the  `username`  and  `password`  properties separately (not in the URL), as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
username: trolley
password: strongpassword
```

If you do not use HTTPS and user credentials, SSH should also work out of the box when you store keys in the default directories ( `~/.ssh` ) and the URI points to an SSH location, such as  `[email protected]:configuration/cloud-configuration` . It is important that an entry for the Git server be present in the  `~/.ssh/known_hosts`  file and that it is in  `ssh-rsa`  format. Other formats (such as  `ecdsa-sha2-nistp256` ) are not supported. To avoid surprises, you should ensure that only one entry is present in the  `known_hosts`  file for the Git server and that it matches the URL you provided to the config server. If you use a hostname in the URL, you want to have exactly that (not the IP) in the  `known_hosts`  file. The repository is accessed by using JGit, so any documentation you find on that should be applicable. HTTPS proxy settings can be set in  `~/.git/config`  or (in the same way as for any other JVM process) with system properties ( `-Dhttps.proxyHost`  and  `-Dhttps.proxyPort` ).

> If you do not know where your  `~/.git`  directory is, use  `git config --global`  to manipulate the settings (for example,  `git config --global http.sslVerify false` ).

#### Authentication with AWS CodeCommit

Spring Cloud Config Server also supports [AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html) authentication. AWS CodeCommit uses an authentication helper when using Git from the command line. This helper is not used with the JGit library, so a JGit CredentialProvider for AWS CodeCommit is created if the Git URI matches the AWS CodeCommit pattern. AWS CodeCommit URIs follow this pattern://git-codecommit.${AWS_REGION}.amazonaws.com/${repopath}.

If you provide a username and password with an AWS CodeCommit URI, they must be the [AWS accessKeyId and secretAccessKey](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html) that provide access to the repository. If you do not specify a username and password, the accessKeyId and secretAccessKey are retrieved by using the [AWS Default Credential Provider Chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html).

If your Git URI matches the CodeCommit URI pattern (shown earlier), you must provide valid AWS credentials in the username and password or in one of the locations supported by the default credential provider chain. AWS EC2 instances may use [IAM Roles for EC2 Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html).

> The  `aws-java-sdk-core`  jar is an optional dependency. If the  `aws-java-sdk-core`  jar is not on your classpath, the AWS Code Commit credential provider is not created, regardless of the git server URI.

#### Git SSH configuration using properties

By default, the JGit library used by Spring Cloud Config Server uses SSH configuration files such as  `~/.ssh/known_hosts`  and  `/etc/ssh/ssh_config`  when connecting to Git repositories by using an SSH URI. In cloud environments such as Cloud Foundry, the local filesystem may be ephemeral or not easily accessible. For those cases, SSH configuration can be set by using Java properties. In order to activate property-based SSH configuration, the  `spring.cloud.config.server.git.ignoreLocalSshSettings`  property must be set to  `true` , as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: git@gitserver.com:team/repo1.git
ignoreLocalSshSettings: true
hostKey: someHostKey
hostKeyAlgorithm: ssh-rsa
privateKey: |
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEAx4UbaDzY5xjW6hc9jwN0mX33XpTDVW9WqHp5AKaRbtAC3DqX
IXFMPgw3K45jxRb93f8tv9vL3rD9CUG1Gv4FM+o7ds7FRES5RTjv2RT/JVNJCoqF
ol8+ngLqRZCyBtQN7zYByWMRirPGoDUqdPYrj2yq+ObBBNhg5N+hOwKjjpzdj2Ud
1l7R+wxIqmJo1IYyy16xS8WsjyQuyC0lL456qkd5BDZ0Ag8j2X9H9D5220Ln7s9i
oezTipXipS7p7Jekf3Ywx6abJwOmB0rX79dV4qiNcGgzATnG1PkXxqt76VhcGa0W
DDVHEEYGbSQ6hIGSh0I7BQun0aLRZojfE3gqHQIDAQABAoIBAQCZmGrk8BK6tXCd
fY6yTiKxFzwb38IQP0ojIUWNrq0+9Xt+NsypviLHkXfXXCKKU4zUHeIGVRq5MN9b
BO56/RrcQHHOoJdUWuOV2qMqJvPUtC0CpGkD+valhfD75MxoXU7s3FK7yjxy3rsG
EmfA6tHV8/4a5umo5TqSd2YTm5B19AhRqiuUVI1wTB41DjULUGiMYrnYrhzQlVvj
5MjnKTlYu3V8PoYDfv1GmxPPh6vlpafXEeEYN8VB97e5x3DGHjZ5UrurAmTLTdO8
+AahyoKsIY612TkkQthJlt7FJAwnCGMgY6podzzvzICLFmmTXYiZ/28I4BX/mOSe
pZVnfRixAoGBAO6Uiwt40/PKs53mCEWngslSCsh9oGAaLTf/XdvMns5VmuyyAyKG
ti8Ol5wqBMi4GIUzjbgUvSUt+IowIrG3f5tN85wpjQ1UGVcpTnl5Qo9xaS1PFScQ
xrtWZ9eNj2TsIAMp/svJsyGG3OibxfnuAIpSXNQiJPwRlW3irzpGgVx/AoGBANYW
dnhshUcEHMJi3aXwR12OTDnaLoanVGLwLnkqLSYUZA7ZegpKq90UAuBdcEfgdpyi
PhKpeaeIiAaNnFo8m9aoTKr+7I6/uMTlwrVnfrsVTZv3orxjwQV20YIBCVRKD1uX
VhE0ozPZxwwKSPAFocpyWpGHGreGF1AIYBE9UBtjAoGBAI8bfPgJpyFyMiGBjO6z
FwlJc/xlFqDusrcHL7abW5qq0L4v3R+FrJw3ZYufzLTVcKfdj6GelwJJO+8wBm+R
gTKYJItEhT48duLIfTDyIpHGVm9+I1MGhh5zKuCqIhxIYr9jHloBB7kRm0rPvYY4
VAykcNgyDvtAVODP+4m6JvhjAoGBALbtTqErKN47V0+JJpapLnF0KxGrqeGIjIRV
cYA6V4WYGr7NeIfesecfOC356PyhgPfpcVyEztwlvwTKb3RzIT1TZN8fH4YBr6Ee
KTbTjefRFhVUjQqnucAvfGi29f+9oE3Ei9f7wA+H35ocF6JvTYUsHNMIO/3gZ38N
CPjyCMa9AoGBAMhsITNe3QcbsXAbdUR00dDsIFVROzyFJ2m40i4KCRM35bC/BIBs
q0TY3we+ERB40U8Z2BvU61QuwaunJ2+uGadHo58VSVdggqAo0BSkH58innKKt96J
69pcVH/4rmLbXdcmNYGm6iu+MlPQk4BUZknHSmVHIFdJ0EPupVaQ8RHT
-----END RSA PRIVATE KEY-----
```

The following table describes the SSH configuration properties.

**Table 5.1. SSH Configuration Properties** 

|Property Name|Remarks|
|----|----|
| **ignoreLocalSshSettings**  |If  `true` , use property-based instead of file-based SSH config. Must be set at as  `spring.cloud.config.server.git.ignoreLocalSshSettings` ,  **not**  inside a repository definition. |
| **privateKey**  |Valid SSH private key. Must be set if  `ignoreLocalSshSettings`  is true and Git URI is SSH format. |
| **hostKey**  |Valid SSH host key. Must be set if  `hostKeyAlgorithm`  is also set. |
| **hostKeyAlgorithm**  |One of  `ssh-dss, ssh-rsa, ecdsa-sha2-nistp256, ecdsa-sha2-nistp384, or ecdsa-sha2-nistp521` . Must be set if  `hostKey`  is also set. |
| **strictHostKeyChecking**  | `true`  or  `false` . If false, ignore errors with host key. |
| **knownHostsFile**  |Location of custom  `.known_hosts`  file. |
| **preferredAuthentications**  |Override server authentication method order. This should allow for evading login prompts if server has keyboard-interactive authentication before the  `publickey`  method. |

#### Placeholders in Git Search Paths

Spring Cloud Config Server also supports a search path with placeholders for the  `{application}`  and  `{profile}`  (and  `{label}`  if you need it), as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
searchPaths: '{application}'
```

The preceding listing causes a search of the repository for files in the same name as the directory (as well as the top level). Wildcards are also valid in a search path with placeholders (any matching directory is included in the search).

#### Force pull in Git Repositories

As mentioned earlier, Spring Cloud Config Server makes a clone of the remote git repository in case the local copy gets dirty (for example, folder content changes by an OS process) such that Spring Cloud Config Server cannot update the local copy from remote repository.

To solve this issue, there is a  `force-pull`  property that makes Spring Cloud Config Server force pull from the remote repository if the local copy is dirty, as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
force-pull: true
```

If you have a multiple-repositories configuration, you can configure the  `force-pull`  property per repository, as shown in the following example:

```java
spring:
cloud:
config:
server:
git:
uri: https://git/common/config-repo.git
force-pull: true
repos:
team-a:
pattern: team-a-*
uri: http://git/team-a/config-repo.git
force-pull: true
team-b:
pattern: team-b-*
uri: http://git/team-b/config-repo.git
force-pull: true
team-c:
pattern: team-c-*
uri: http://git/team-a/config-repo.git
```

> The default value for  `force-pull`  property is  `false` .

#### Deleting untracked branches in Git Repositories

As Spring Cloud Config Server has a clone of the remote git repository after check-outing branch to local repo (e.g fetching properties by label) it will keep this branch forever or till the next server restart (which creates new local repo). So there could be a case when remote branch is deleted but local copy of it is still available for fetching. And if Spring Cloud Config Server client service starts with  `--spring.cloud.config.label=deletedRemoteBranch,master`  it will fetch properties from  `deletedRemoteBranch`  local branch, but not from  `master` .

In order to keep local repository branches clean and up to remote -  `deleteUntrackedBranches`  property could be set. It will make Spring Cloud Config Server  **force**  delete untracked branches from local repository. Example:

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
deleteUntrackedBranches: true
```

> The default value for  `deleteUntrackedBranches`  property is  `false` .

#### Git Refresh Rate

You can control how often the config server will fetch updated configuration data from your Git backend by using  `spring.cloud.config.server.git.refreshRate` . The value of this property is specified in seconds. By default the value is 0, meaning the config server will fetch updated configuration from the Git repo every time it is requested.

### 5.1.2 Version Control Backend Filesystem Use

> With VCS-based backends (git, svn), files are checked out or cloned to the local filesystem. By default, they are put in the system temporary directory with a prefix of  `config-repo-` . On linux, for example, it could be  `/tmp/config-repo-<randomid>` . Some operating systems [routinely clean out](https://serverfault.com/questions/377348/when-does-tmp-get-cleared/377349#377349) temporary directories. This can lead to unexpected behavior, such as missing properties. To avoid this problem, change the directory that Config Server uses by setting  `spring.cloud.config.server.git.basedir`  or  `spring.cloud.config.server.svn.basedir`  to a directory that does not reside in the system temp structure.

### 5.1.3 File System Backend

There is also a “native” profile in the Config Server that does not use Git but loads the config files from the local classpath or file system (any static URL you want to point to with  `spring.cloud.config.server.native.searchLocations` ). To use the native profile, launch the Config Server with  `spring.profiles.active=native` .

> Remember to use the  `file:`  prefix for file resources (the default without a prefix is usually the classpath). As with any Spring Boot configuration, you can embed  `${}` -style environment placeholders, but remember that absolute paths in Windows require an extra  `/`  (for example,  `file:///${user.home}/config-repo` ).

> The default value of the  `searchLocations`  is identical to a local Spring Boot application (that is,  `[classpath:/, classpath:/config, file:./, file:./config]` ). This does not expose the  `application.properties`  from the server to all clients, because any property sources present in the server are removed before being sent to the client.

> A filesystem backend is great for getting started quickly and for testing. To use it in production, you need to be sure that the file system is reliable and shared across all instances of the Config Server.

The search locations can contain placeholders for  `{application}` ,  `{profile}` , and  `{label}` . In this way, you can segregate the directories in the path and choose a strategy that makes sense for you (such as subdirectory per application or subdirectory per profile).

If you do not use placeholders in the search locations, this repository also appends the  `{label}`  parameter of the HTTP resource to a suffix on the search path, so properties files are loaded from each search location  **and**  a subdirectory with the same name as the label (the labelled properties take precedence in the Spring Environment). Thus, the default behaviour with no placeholders is the same as adding a search location ending with  `/{label}/` . For example,  `file:/tmp/config`  is the same as  `file:/tmp/config,file:/tmp/config/{label}` . This behavior can be disabled by setting  `spring.cloud.config.server.native.addLabelLocations=false` .

### 5.1.4 Vault Backend

Spring Cloud Config Server also supports [Vault](https://www.vaultproject.io) as a backend.

----
Vault is a tool for securely accessing secrets. A secret is anything that to which you want to tightly control access, such as API keys, passwords, certificates, and other sensitive information. Vault provides a unified interface to any secret while providing tight access control and recording a detailed audit log.

----

For more information on Vault, see the [Vault quick start guide](https://www.vaultproject.io/intro/index.html).

To enable the config server to use a Vault backend, you can run your config server with the  `vault`  profile. For example, in your config server’s  `application.properties` , you can add  `spring.profiles.active=vault` .

By default, the config server assumes that your Vault server runs at  `http://127.0.0.1:8200` . It also assumes that the name of backend is  `secret`  and the key is  `application` . All of these defaults can be configured in your config server’s  `application.properties` . The following table describes configurable Vault properties:

|Name|Default Value|
|----|----|
|host |127.0.0.1 |
|port |8200 |
|scheme |http |
|backend |secret |
|defaultKey |application |
|profileSeparator |, |
|kvVersion |1 |
|skipSslValidation |false |
|timeout |5 |

|images/important.png|Important|
|----|----|
|All of the properties in the preceding table must be prefixed with  `spring.cloud.config.server.vault` . |

All configurable properties can be found in  `org.springframework.cloud.config.server.environment.VaultEnvironmentRepository` .

Vault 0.10.0 introduced a versioned key-value backend (k/v backend version 2) that exposes a different API than earlier versions, it now requires a  `data/`  between the mount path and the actual context path and wraps secrets in a  `data`  object. Setting  `kvVersion=2`  will take this into account.

With your config server running, you can make HTTP requests to the server to retrieve values from the Vault backend. To do so, you need a token for your Vault server.

First, place some data in you Vault, as shown in the following example:

```java
$ vault write secret/application foo=bar baz=bam
$ vault write secret/myapp foo=myappsbar
```

Second, make an HTTP request to your config server to retrieve the values, as shown in the following example:

`$ curl -X "GET" "http://localhost:8888/myapp/default" -H "X-Config-Token: yourtoken"` 

You should see a response similar to the following:

```java
{
"name":"myapp",
"profiles":[
"default"
],
"label":null,
"version":null,
"state":null,
"propertySources":[
{
"name":"vault:myapp",
"source":{
"foo":"myappsbar"
}
},
{
"name":"vault:application",
"source":{
"baz":"bam",
"foo":"bar"
}
}
]
}
```

#### Multiple Properties Sources

When using Vault, you can provide your applications with multiple properties sources. For example, assume you have written data to the following paths in Vault:

```java
secret/myApp,dev
secret/myApp
secret/application,dev
secret/application
```

Properties written to  `secret/application`  are available to [all applications using the Config Server](). An application with the name,  `myApp` , would have any properties written to  `secret/myApp`  and  `secret/application`  available to it. When  `myApp`  has the  `dev`  profile enabled, properties written to all of the above paths would be available to it, with properties in the first path in the list taking priority over the others.

### 5.1.5 Accessing Backends Through a Proxy

The configuration server can access a Git or Vault backend through an HTTP or HTTPS proxy. This behavior is controlled for either Git or Vault by settings under  `proxy.http`  and  `proxy.https` . These settings are per repository, so if you are using a [composite environment repository](multi__spring_cloud_config_server.html#composite-environment-repositories) you must configure proxy settings for each backend in the composite individually. If using a network which requires separate proxy servers for HTTP and HTTPS URLs, you can configure both the HTTP and the HTTPS proxy settings for a single backend.

The following table describes the proxy configuration properties for both HTTP and HTTPS proxies. All of these properties must be prefixed by  `proxy.http`  or  `proxy.https` .

**Table 5.2. Proxy Configuration Properties** 

|Property Name|Remarks|
|----|----|
| **host**  |The host of the proxy. |
| **port**  |The port with which to access the proxy. |
| **nonProxyHosts**  |Any hosts which the configuration server should access outside the proxy. If values are provided for both  `proxy.http.nonProxyHosts`  and  `proxy.https.nonProxyHosts` , the  `proxy.http`  value will be used. |
| **username**  |The username with which to authenticate to the proxy. If values are provided for both  `proxy.http.username`  and  `proxy.https.username` , the  `proxy.http`  value will be used. |
| **password**  |The password with which to authenticate to the proxy. If values are provided for both  `proxy.http.password`  and  `proxy.https.password` , the  `proxy.http`  value will be used. |

The following configuration uses an HTTPS proxy to access a Git repository.

```java
spring:
profiles:
active: git
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
proxy:
https:
host: my-proxy.host.io
password: myproxypassword
port: '3128'
username: myproxyusername
nonProxyHosts: example.com
```

### 5.1.6 Sharing Configuration With All Applications

Sharing configuration between all applications varies according to which approach you take, as described in the following topics:

- [the section called “File Based Repositories”](multi__spring_cloud_config_server.html#spring-cloud-config-server-file-based-repositories)

- [the section called “Vault Server”](multi__spring_cloud_config_server.html#spring-cloud-config-server-vault-server)

#### File Based Repositories

With file-based (git, svn, and native) repositories, resources with file names in  `application*`  ( `application.properties` ,  `application.yml` ,  `application-*.properties` , and so on) are shared between all client applications. You can use resources with these file names to configure global defaults and have them be overridden by application-specific files as necessary.

The #_property_overrides[property overrides] feature can also be used for setting global defaults, with placeholders applications allowed to override them locally.

> With the “native” profile (a local file system backend) , you should use an explicit search location that is not part of the server’s own configuration. Otherwise, the  `application*`  resources in the default search locations get removed because they are part of the server.

#### Vault Server

When using Vault as a backend, you can share configuration with all applications by placing configuration in  `secret/application` . For example, if you run the following Vault command, all applications using the config server will have the properties  `foo`  and  `baz`  available to them:

```java
$ vault write secret/application foo=bar baz=bam
```

### 5.1.7 JDBC Backend

Spring Cloud Config Server supports JDBC (relational database) as a backend for configuration properties. You can enable this feature by adding  `spring-jdbc`  to the classpath and using the  `jdbc`  profile or by adding a bean of type  `JdbcEnvironmentRepository` . If you include the right dependencies on the classpath (see the user guide for more details on that), Spring Boot configures a data source.

The database needs to have a table called  `PROPERTIES`  with columns called  `APPLICATION` ,  `PROFILE` , and  `LABEL`  (with the usual  `Environment`  meaning), plus  `KEY`  and  `VALUE`  for the key and value pairs in  `Properties`  style. All fields are of type String in Java, so you can make them  `VARCHAR`  of whatever length you need. Property values behave in the same way as they would if they came from Spring Boot properties files named  `{application}-{profile}.properties` , including all the encryption and decryption, which will be applied as post-processing steps (that is, not in the repository implementation directly).

### 5.1.8 Composite Environment Repositories

In some scenarios, you may wish to pull configuration data from multiple environment repositories. To do so, you can enable the  `composite`  profile in your configuration server’s application properties or YAML file. If, for example, you want to pull configuration data from a Subversion repository as well as two Git repositories, you can set the following properties for your configuration server:

```java
spring:
profiles:
active: composite
cloud:
config:
server:
composite:
-
type: svn
uri: file:///path/to/svn/repo
-
type: git
uri: file:///path/to/rex/git/repo
-
type: git
uri: file:///path/to/walter/git/repo
```

Using this configuration, precedence is determined by the order in which repositories are listed under the  `composite`  key. In the above example, the Subversion repository is listed first, so a value found in the Subversion repository will override values found for the same property in one of the Git repositories. A value found in the  `rex`  Git repository will be used before a value found for the same property in the  `walter`  Git repository.

If you want to pull configuration data only from repositories that are each of distinct types, you can enable the corresponding profiles, rather than the  `composite`  profile, in your configuration server’s application properties or YAML file. If, for example, you want to pull configuration data from a single Git repository and a single HashiCorp Vault server, you can set the following properties for your configuration server:

```java
spring:
profiles:
active: git, vault
cloud:
config:
server:
git:
uri: file:///path/to/git/repo
order: 2
vault:
host: 127.0.0.1
port: 8200
order: 1
```

Using this configuration, precedence can be determined by an  `order`  property. You can use the  `order`  property to specify the priority order for all your repositories. The lower the numerical value of the  `order`  property, the higher priority it has. The priority order of a repository helps resolve any potential conflicts between repositories that contain values for the same properties.

> If your composite environment includes a Vault server as in the previous example, you must include a Vault token in every request made to the configuration server. See [Vault Backend](multi__spring_cloud_config_server.html#vault-backend).

> Any type of failure when retrieving values from an environment repository results in a failure for the entire composite environment.

> When using a composite environment, it is important that all repositories contain the same labels. If you have an environment similar to those in the preceding examples and you request configuration data with the  `master`  label but the Subversion repository does not contain a branch called  `master` , the entire request fails.

#### Custom Composite Environment Repositories

In addition to using one of the environment repositories from Spring Cloud, you can also provide your own  `EnvironmentRepository`  bean to be included as part of a composite environment. To do so, your bean must implement the  `EnvironmentRepository`  interface. If you want to control the priority of your custom  `EnvironmentRepository`  within the composite environment, you should also implement the  `Ordered`  interface and override the  `getOrdered`  method. If you do not implement the  `Ordered`  interface, your  `EnvironmentRepository`  is given the lowest priority.

### 5.1.9 Property Overrides

The Config Server has an “overrides” feature that lets the operator provide configuration properties to all applications. The overridden properties cannot be accidentally changed by the application with the normal Spring Boot hooks. To declare overrides, add a map of name-value pairs to  `spring.cloud.config.server.overrides` , as shown in the following example:

```java
spring:
cloud:
config:
server:
overrides:
foo: bar
```

The preceding examples causes all applications that are config clients to read  `foo=bar` , independent of their own configuration.

> A configuration system cannot force an application to use configuration data in any particular way. Consequently, overrides are not enforceable. However, they do provide useful default behavior for Spring Cloud Config clients.

> Normally, Spring environment placeholders with  `${}`  can be escaped (and resolved on the client) by using backslash ( `\` ) to escape the  `$`  or the  `{` . For example,  `\${app.foo:bar}`  resolves to  `bar` , unless the app provides its own  `app.foo` .

> In YAML, you do not need to escape the backslash itself. However, in properties files, you do need to escape the backslash, when you configure the overrides on the server.

You can change the priority of all overrides in the client to be more like default values, letting applications supply their own values in environment variables or System properties, by setting the  `spring.cloud.config.overrideNone=true`  flag (the default is false) in the remote repository.

## 5.2 Health Indicator

Config Server comes with a Health Indicator that checks whether the configured  `EnvironmentRepository`  is working. By default, it asks the  `EnvironmentRepository`  for an application named  `app` , the  `default`  profile, and the default label provided by the  `EnvironmentRepository`  implementation.

You can configure the Health Indicator to check more applications along with custom profiles and custom labels, as shown in the following example:

```java
spring:
cloud:
config:
server:
health:
repositories:
myservice:
label: mylabel
myservice-dev:
name: myservice
profiles: development
```

You can disable the Health Indicator by setting  `spring.cloud.config.server.health.enabled=false` .

## 5.3 Security

You can secure your Config Server in any way that makes sense to you (from physical network security to OAuth2 bearer tokens), because Spring Security and Spring Boot offer support for many security arrangements.

To use the default Spring Boot-configured HTTP Basic security, include Spring Security on the classpath (for example, through  `spring-boot-starter-security` ). The default is a username of  `user`  and a randomly generated password. A random password is not useful in practice, so we recommend you configure the password (by setting  `spring.security.user.password` ) and encrypt it (see below for instructions on how to do that).

## 5.4 Encryption and Decryption

|images/important.png|Important|
|----|----|
|To use the encryption and decryption features you need the full-strength JCE installed in your JVM (it is not included by default). You can download the “Java Cryptography Extension (JCE) Unlimited Strength Jurisdiction Policy Files” from Oracle and follow the installation instructions (essentially, you need to replace the two policy files in the JRE lib/security directory with the ones that you downloaded). |

If the remote property sources contain encrypted content (values starting with  `{cipher}` ), they are decrypted before sending to clients over HTTP. The main advantage of this setup is that the property values need not be in plain text when they are “at rest” (for example, in a git repository). If a value cannot be decrypted, it is removed from the property source and an additional property is added with the same key but prefixed with  `invalid`  and a value that means “not applicable” (usually  `<n/a>` ). This is largely to prevent cipher text being used as a password and accidentally leaking.

If you set up a remote config repository for config client applications, it might contain an  `application.yml`  similar to the following:

**application.yml.**  

```java
spring:
datasource:
username: dbuser
password: '{cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ'
```

Encrypted values in a .properties file must not be wrapped in quotes. Otherwise, the value is not decrypted. The following example shows values that would work:

**application.properties.**  

```java
spring.datasource.username: dbuser
spring.datasource.password: {cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ
```

You can safely push this plain text to a shared git repository, and the secret password remains protected.

The server also exposes  `/encrypt`  and  `/decrypt`  endpoints (on the assumption that these are secured and only accessed by authorized agents). If you edit a remote config file, you can use the Config Server to encrypt values by POSTing to the  `/encrypt`  endpoint, as shown in the following example:

```java
$ curl localhost:8888/encrypt -d mysecret
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
```

> If the value you encrypt has characters in it that need to be URL encoded, you should use the  `--data-urlencode`  option to  `curl`  to make sure they are encoded properly.

> Be sure not to include any of the curl command statistics in the encrypted value. Outputting the value to a file can help avoid this problem.

The inverse operation is also available through  `/decrypt`  (provided the server is configured with a symmetric key or a full key pair), as shown in the following example:

```java
$ curl localhost:8888/decrypt -d 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

> If you testing with curl, then use  `--data-urlencode`  (instead of  `-d` ) or set an explicit  `Content-Type: text/plain`  to make sure curl encodes the data correctly when there are special characters ('+' is particularly tricky).

Take the encrypted value and add the  `{cipher}`  prefix before you put it in the YAML or properties file and before you commit and push it to a remote (potentially insecure) store.

The  `/encrypt`  and  `/decrypt`  endpoints also both accept paths in the form of  `/*/{name}/{profiles}` , which can be used to control cryptography on a per-application (name) and per-profile basis when clients call into the main environment resource.

> To control the cryptography in this granular way, you must also provide a  `@Bean`  of type  `TextEncryptorLocator`  that creates a different encryptor per name and profiles. The one that is provided by default does not do so (all encryptions use the same key).

The  `spring`  command line client (with Spring Cloud CLI extensions installed) can also be used to encrypt and decrypt, as shown in the following example:

```java
$ spring encrypt mysecret --key foo
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
$ spring decrypt --key foo 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

To use a key in a file (such as an RSA public key for encryption), prepend the key value with "@" and provide the file path, as shown in the following example:

```java
$ spring encrypt mysecret --key @${HOME}/.ssh/id_rsa.pub
AQAjPgt3eFZQXwt8tsHAVv/QHiY5sI2dRcR+...
```

> The  `--key`  argument is mandatory (despite having a  `--`  prefix).

## 5.5 Key Management

The Config Server can use a symmetric (shared) key or an asymmetric one (RSA key pair). The asymmetric choice is superior in terms of security, but it is often more convenient to use a symmetric key since it is a single property value to configure in the  `bootstrap.properties` .

To configure a symmetric key, you need to set  `encrypt.key`  to a secret String (or use the  `ENCRYPT_KEY`  environment variable to keep it out of plain-text configuration files).

To configure an asymmetric key, you can either set the key as a PEM-encoded text value (in  `encrypt.key` ) or use a keystore (such as the keystore created by the  `keytool`  utility that comes with the JDK). The following table describes the keystore properties:

|Property|Description|
|----|----|
| `encrypt.keyStore.location`  |Contains a  `Resource`  location |
| `encrypt.keyStore.password`  |Holds the password that unlocks the keystore |
| `encrypt.keyStore.alias`  |Identifies which key in the store to use |

The encryption is done with the public key, and a private key is needed for decryption. Thus, in principle, you can configure only the public key in the server if you want to only encrypt (and are prepared to decrypt the values yourself locally with the private key). In practice, you might not want to do decrypt locally, because it spreads the key management process around all the clients, instead of concentrating it in the server. On the other hand, it can be a useful option if your config server is relatively insecure and only a handful of clients need the encrypted properties.

## 5.6 Creating a Key Store for Testing

To create a keystore for testing, you can use a command resembling the following:

```java
$ keytool -genkeypair -alias mytestkey -keyalg RSA \
-dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
-keypass changeme -keystore server.jks -storepass letmein
```

Put the  `server.jks`  file in the classpath (for instance) and then, in your  `bootstrap.yml` , for the Config Server, create the following settings:

```java
encrypt:
keyStore:
location: classpath:/server.jks
password: letmein
alias: mytestkey
secret: changeme
```

## 5.7 Using Multiple Keys and Key Rotation

In addition to the  `{cipher}`  prefix in encrypted property values, the Config Server looks for zero or more  `{name:value}`  prefixes before the start of the (Base64 encoded) cipher text. The keys are passed to a  `TextEncryptorLocator` , which can do whatever logic it needs to locate a  `TextEncryptor`  for the cipher. If you have configured a keystore ( `encrypt.keystore.location` ), the default locator looks for keys with aliases supplied by the  `key`  prefix, with a cipher text like resembling the following:

```java
foo:
bar: `{cipher}{key:testkey}...`
```

The locator looks for a key named "testkey". A secret can also be supplied by using a  `{secret:…}`  value in the prefix. However, if it is not supplied, the default is to use the keystore password (which is what you get when you build a keytore and do not specify a secret). If you do supply a secret, you should also encrypt the secret using a custom  `SecretLocator` .

When the keys are being used only to encrypt a few bytes of configuration data (that is, they are not being used elsewhere), key rotation is hardly ever necessary on cryptographic grounds. However, you might occasionally need to change the keys (for example, in the event of a security breach). In that case, all the clients would need to change their source config files (for example, in git) and use a new  `{key:…}`  prefix in all the ciphers. Note that the clients need to first check that the key alias is available in the Config Server keystore.

> If you want to let the Config Server handle all encryption as well as decryption, the  `{name:value}`  prefixes can also be added as plain text posted to the  `/encrypt`  endpoint, .

## 5.8 Serving Encrypted Properties

Sometimes you want the clients to decrypt the configuration locally, instead of doing it in the server. In that case, if you provide the  `encrypt.*`  configuration to locate a key, you can still have  `/encrypt`  and  `/decrypt`  endpoints, but you need to explicitly switch off the decryption of outgoing properties by placing  `spring.cloud.config.server.encrypt.enabled=false`  in  `bootstrap.[yml|properties]` . If you do not care about the endpoints, it should work if you do not configure either the key or the enabled flag.


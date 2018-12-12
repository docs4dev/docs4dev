## 5. Spring Cloud Config Server

Spring Cloud Config Server为外部配置（名称 - 值对或等效的YAML内容）提供基于HTTP资源的API.通过使用 `@EnableConfigServer` 注释，服务器可嵌入Spring Boot应用程序中.因此，以下应用程序是配置服务器：

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

与所有Spring Boot应用程序一样，它默认在端口8080上运行，但您可以通过各种方式将其切换到更传统的端口8888.最简单的，也是设置默认配置存储库，是通过 `spring.config.name=configserver` 启动它（在Config Server jar中有一个 `configserver.yml` ）.另一个是使用您自己的 `application.properties` ，如以下示例所示：

**application.properties.** 

```java
server.port: 8888
spring.cloud.config.server.git.uri: file://${user.home}/config-repo
```

其中 `${user.home}/config-repo` 是包含YAML和属性文件的git存储库.

> 在Windows上，如果文件URL是绝对的，则在文件URL中需要额外的"/"（例如， `file:///${user.home}/config-repo` ）.

> 以下清单显示了在前面的示例中创建git存储库的方法：

> 使用git存储库的本地文件系统仅用于测试.您应该使用服务器在生产环境中托管配置存储库.

> 如果只保留文本文件，则配置库的初始克隆可以快速有效.如果你存储二进制文件，尤其是大型文件，您可能会遇到第一次配置请求或服务器中遇到内存不足错误的延迟.

## 5.1环境信息库

您应该在哪里存储配置服务器的配置数据？管理此行为的策略是 `EnvironmentRepository` ，提供 `Environment` 对象.这个 `Environment` 是来自Spring  `Environment` 的域的浅表副本（包括 `propertySources` 作为主要特征）.  `Environment` 资源由三个变量参数化：

-  `{application}` ，在客户端映射到 `spring.application.name` .

-  `{profile}` ，映射到客户端上的 `spring.profiles.active` （以逗号分隔的列表）.

-  `{label}` ，这是标记"versioned"配置文件集的服务器端功能.

存储库实现通常表现得像Spring Boot应用程序，从 `spring.config.name` 加载配置文件等于 `{application}` 参数， `spring.profiles.active` 等于 `{profiles}` 参数.配置文件的优先规则也与常规Spring Boot应用程序相同：活动配置文件优先于默认配置文件，如果有多个配置文件，则最后一个配置文件获胜（类似于向 `Map` 添加条目）.

以下示例客户端应用程序具有此引导程序配置：

**bootstrap.yml.** 

```java
spring:
application:
name: foo
profiles:
active: dev,mysql
```

（像往常一样，Spring Boot应用程序也可以通过环境变量或命令行参数设置这些属性）.

如果存储库是基于文件的，则服务器从 `application.yml` （在所有客户端之间共享）和 `foo.yml` （优先使用 `foo.yml` ）创建 `Environment` .如果YAML文件中包含指向Spring配置文件的文档，则应用优先级较高的文档（按列出的配置文件的顺序）.如果存在特定于配置文件的YAML（或属性）文件，则这些文件的优先级也高于默认值.较高的优先级转换为 `Environment` 中前面列出的 `PropertySource` . （这些相同的规则适用于独立的Spring Boot应用程序.）

您可以将spring.cloud.config.server.accept-empty设置为false，以便在找不到应用程序时，Server将返回HTTP 404状态.默认情况下，此标志设置为true.

### 5.1.1 Git Backend

`EnvironmentRepository` 的默认实现使用Git后端，这对于管理升级和物理环境以及审计更改非常方便.要更改存储库的位置，可以在Config Server中设置 `spring.cloud.config.server.git.uri` 配置属性（例如，在 `application.yml` 中）.如果使用 `file:` 前缀设置它，它应该从本地存储库工作，这样您就可以在没有服务器的情况下快速轻松地开始使用.但是，在这种情况下，服务器直接在本地存储库上运行而不进行克隆（如果它不是裸的则无关紧要，因为Config Server永远不会对"remote"存储库进行更改）.要向上扩展Config Server并使其具有高可用性，您需要让服务器的所有实例指向同一个存储库，因此只有共享文件系统才能工作.即使在这种情况下，最好将 `ssh:` 协议用于共享文件系统存储库，以便服务器可以克隆它并使用本地工作副本作为缓存.

此存储库实现将HTTP资源的 `{label}` 参数映射到git标签（提交标识，分支名称或标记）.如果git分支或标记名称包含斜杠（ `/` ），则应使用特殊字符串 `(_)` 指定HTTP URL中的标签（以避免与其他URL路径不一致）.例如，如果标签是 `foo/bar` ，则替换斜杠将导致以下标签： `foo(_)bar` .包含特殊字符串 `(_)` 也可以应用于 `{application}` 参数.如果您使用命令行客户端（如curl），请小心URL中的括号 - 您应该使用单引号（''）将它们从shell中转义.

#### Skipping SSL证书验证

通过将 `git.skipSslValidation` 属性设置为 `true` （默认为 `false` ），可以禁用配置服务器对Git服务器的SSL证书的验证.

```java
spring:
cloud:
config:
server:
git:
uri: https://example.com/my/repo
skipSslValidation: true
```

#### Setting HTTP连接超时

您可以配置配置服务器等待获取HTTP连接的时间（以秒为单位）.使用 `git.timeout` 属性.

```java
spring:
cloud:
config:
server:
git:
uri: https://example.com/my/repo
timeout: 4
```

Git URI中的
#### Placeholders

Spring Cloud Config Server支持带有占位符的git存储库URL，用于 `{application}` 和 `{profile}` （如果需要，可以使用 `{label}` ，但请记住，无论如何都将标签应用为git标签）.因此，您可以使用类似于以下的结构来支持“每个应用程序一个存储库”策略：

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/myorg/{application}
```

您还可以使用类似的模式支持“每个配置文件一个存储库”策略，但使用 `{profile}` .

此外，在 `{application}` 参数中使用特殊字符串"(_)"可以启用对多个组织的支持，如以下示例所示：

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/{application}
```

其中 `{application}` 在请求时以下列格式提供： `organization(_)application` .

#### Pattern匹配和多个存储库

Spring Cloud Config还包括对应用程序和配置文件名称上的模式匹配更复杂的要求.模式格式是带有通配符的 `{application}/{profile}` 名称的逗号分隔列表（请注意，可能需要引用以通配符开头的模式），如以下示例所示：

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

如果 `{application}/{profile}` 与任何模式都不匹配，则使用 `spring.cloud.config.server.git.uri` 下定义的默认URI.在上面的示例中，对于“简单”存储库，模式为 `simple/*` （它仅匹配所有配置文件中名为 `simple` 的一个应用程序）. “本地”存储库匹配所有配置文件中以 `local` 开头的所有应用程序名称（ `/*` 后缀会自动添加到任何没有配置文件匹配器的模式）.

> 只有当要设置的唯一属性是URI时，才能使用“简单”示例中使用的“单行”快捷方式.如果您需要设置其他任何内容（凭据，模式等），则需要使用完整表单.

repo中的 `pattern` 属性实际上是一个数组，因此您可以使用YAML数组（或属性文件中的 `[0]` ， `[1]` 等后缀）绑定到多个模式.如果要运行具有多个配置文件的应用程序，则可能需要执行此操作，如以下示例所示：

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

> Spring Cloud猜测包含未在 `*` 结尾的配置文件的模式意味着您实际上想要匹配以此模式开头的配置文件列表（因此 `*/staging` 是 `["*/staging", "*/staging,*"]` 的快捷方式，依此类推）.这很常见，例如，您需要在本地“开发”配置文件中运行应用程序，而远程运行“Cloud”配置文件.

每个存储库还可以选择将配置文件存储在子目录中，搜索这些目录的模式可以指定为 `searchPaths` .以下示例显示顶级配置文件：

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
searchPaths: foo,bar*
```

在前面的示例中，服务器搜索顶级和 `foo/` 子目录中的配置文件以及名称以 `bar` 开头的任何子目录.

默认情况下，服务器在首次请求配置时克隆远程存储库.可以将服务器配置为在启动时克隆存储库，如以下顶级示例所示：

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

在前面的示例中，服务器在接受任何请求之前在启动时克隆team-a的config-repo.在请求来自存储库的配置之前，不会克隆所有其他存储库.

_0015在Config Server启动时设置要克隆的存储库有助于在Config Server启动时快速识别配置错误的配置源（例如无效的存储库URI）.如果配置源未启用 `cloneOnStart` ，则配置服务器可能会在配置错误或配置无效的情况下成功启动，并且在应用程序从该配置源请求配置之前不会检测到错误.

#### Authentication

要在远程存储库上使用HTTP基本身份验证，请单独添加 `username` 和 `password` 属性（不在URL中），如以下示例所示：

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

如果您不使用HTTPS和用户凭据，则在将密钥存储在默认目录（ `~/.ssh` ）中并且URI指向SSH位置（例如 `[email protected]:configuration/cloud-configuration` ）时，SSH也应该开箱即用.重要的是，Git服务器的条目存在于 `~/.ssh/known_hosts` 文件中，并且它是 `ssh-rsa` 格式.不支持其他格式（例如 `ecdsa-sha2-nistp256` ）.为避免出现意外，您应确保Git服务器的 `known_hosts` 文件中只有一个条目，并且它与您提供给配置服务器的URL相匹配.如果在URL中使用主机名，则希望在 `known_hosts` 文件中具有该主机名（而不是IP）.使用JGit访问存储库，因此您在其上找到的任何文档都应该适用. HTTPS代理设置可以在 `~/.git/config` 中设置或（与任何其他JVM进程相同）具有系统属性（ `-Dhttps.proxyHost` 和 `-Dhttps.proxyPort` ）.

> 如果您不知道 `~/.git` 目录的位置，请使用 `git config --global` 来操作设置（例如， `git config --global http.sslVerify false` ）.

#### 使用AWS CodeCommit进行身份验证

Spring Cloud Config Server还支持[AWS CodeCommit](https://docs.aws.amazon.com/codecommit/latest/userguide/welcome.html)身份验证.从命令行使用Git时，AWS CodeCommit使用身份验证帮助程序.此助手不与JGit库一起使用，因此如果Git URI与AWS CodeCommit模式匹配，则会创建AWS CodeCommit的JGit CredentialProvider. AWS CodeCommit URI遵循以下模式：//git-codecommit.$ {AWS_REGION} .amazonaws.com / $ {repopath}.

如果您提供带有AWS CodeCommit URI的用户名和密码，则它们必须是[AWS accessKeyId and secretAccessKey](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSGettingStartedGuide/AWSCredentials.html)，以提供对存储库的访问权限.如果未指定用户名和密码，则使用[AWS Default Credential Provider Chain](https://docs.aws.amazon.com/sdk-for-java/v1/developer-guide/credentials.html)检索accessKeyId和secretAccessKey.

如果您的Git URI与CodeCommit URI模式匹配（如前所示），则必须在用户名和密码或默认凭据提供程序链支持的某个位置提供有效的AWS凭据. AWS EC2实例可以使用[IAM Roles for EC2 Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html).

> The  `aws-java-sdk-core`  jar是一个可选的依赖项.如果 `aws-java-sdk-core`  jar不在您的类路径中，则无论git服务器URI如何，都不会创建AWS Code Commit凭证提供程序.

#### 使用属性进行SSH配置

默认情况下，Spring Cloud Config Server使用的JGit库在使用SSH URI连接到Git存储库时使用SSH配置文件，如 `~/.ssh/known_hosts` 和 `/etc/ssh/ssh_config` .在Cloud Foundry等Cloud环境中，本地文件系统可能是短暂的或不易访问的.对于这些情况，可以使用Java属性设置SSH配置.要激活基于属性的SSH配置，必须将 `spring.cloud.config.server.git.ignoreLocalSshSettings` 属性设置为 `true` ，如以下示例所示：

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

下表介绍了SSH配置属性.

**Table 5.1. SSH Configuration Properties** 

|properties名称|备注|
| ---- | ---- |
|  **ignoreLocalSshSettings**  |如果是 `true` ，请使用基于属性而不是基于文件的SSH配置.必须在存储库定义中设置为 `spring.cloud.config.server.git.ignoreLocalSshSettings` ， **not** . |
|  **privateKey**  |有效的SSH私钥.如果 `ignoreLocalSshSettings` 为true且Git URI为SSH格式，则必须设置. |
|  **hostKey**  |有效的SSH主机密钥.如果还设置了 `hostKeyAlgorithm` ，则必须设置. |
|  **hostKeyAlgorithm**  |  `ssh-dss, ssh-rsa, ecdsa-sha2-nistp256, ecdsa-sha2-nistp384, or ecdsa-sha2-nistp521` 之一.如果还设置了 `hostKey` ，则必须设置. |
|  **strictHostKeyChecking**  |  `true` 或 `false` .如果为false，则忽略主机密钥错误. |
|  **knownHostsFile**  |自定义 `.known_hosts` 文件的位置. |
|  **preferredAuthentications**  |覆盖服务器身份验证方法顺序.如果服务器在 `publickey` 方法之前具有键盘交互式身份验证，则应允许避免登录提示. |

Git搜索路径中的
#### Placeholders

Spring Cloud Config Server还支持带有占位符的搜索路径 `{application}` 和 `{profile}` （如果需要，还有 `{label}` ），如以下示例所示：

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
searchPaths: '{application}'
```

上面的清单导致在存储库中搜索与目录（以及顶层）同名的文件.通配符在带占位符的搜索路径中也有效（搜索中包含任何匹配的目录）.

#### Force拉入Git存储库

如前所述，Spring Cloud Config Server会在本地副本变脏（例如，OS进程更改文件夹内容）时复制远程git存储库，以使Spring Cloud Config Server无法从远程存储库更新本地副本.

要解决此问题，有一个 `force-pull` 属性，如果本地副本是脏的，则会从远程存储库强制提取Spring Cloud Config Server，如以下示例所示：

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
force-pull: true
```

如果您有多个存储库配置，则可以为每个存储库配置 `force-pull` 属性，如以下示例所示：

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

>   `force-pull` 属性的默认值为 `false` .

#### 删除Git存储库中未跟踪的分支

由于Spring Cloud Config Server在签出分支到本地存储库后具有远程git存储库的克隆（例如，通过标签获取属性），它将永久保留此分支或直到下一个服务器重新启动（这将创建新的本地存储库）.因此可能存在删除远程分支但仍然可以获取其本地副本的情况.如果Spring Cloud Config Server客户端服务以 `--spring.cloud.config.label=deletedRemoteBranch,master` 开头，它将从 `deletedRemoteBranch` 本地分支获取属性，但不从 `master` 获取.

为了保持本地存储库分支的清洁和远程 - 可以设置 `deleteUntrackedBranches` 属性.它将使Spring Cloud Config Server  **force** 从本地存储库中删除未跟踪的分支.例：

```java
spring:
cloud:
config:
server:
git:
uri: https://github.com/spring-cloud-samples/config-repo
deleteUntrackedBranches: true
```

>   `deleteUntrackedBranches` 属性的默认值为 `false` .

#### Git刷新率

您可以使用 `spring.cloud.config.server.git.refreshRate` 控制配置服务器从Git后端获取更新配置数据的频率.此属性的值以秒为单位指定.默认情况下，该值为0，这意味着配置服务器将在每次请求时从Git存储库获取更新的配置.

### 5.1.2版本控制后端文件系统使用

> 使用基于VCS的后端（git，svn），文件被签出或克隆到本地文件系统.默认情况下，它们放在系统临时目录中，前缀为 `config-repo-` .例如，在linux上，它可能是 `/tmp/config-repo-<randomid>` .一些操作系统[routinely clean out](https://serverfault.com/questions/377348/when-does-tmp-get-cleared/377349#377349)临时目录.这可能会导致意外行为，例如缺少属性.要避免此问题，请通过将 `spring.cloud.config.server.git.basedir` 或 `spring.cloud.config.server.svn.basedir` 设置为不驻留在系统临时结构中的目录来更改Config Server使用的目录.

### 5.1.3文件系统后端

Config Server中还有一个“本机”配置文件，它不使用Git，但从本地类路径或文件系统加载配置文件（使用 `spring.cloud.config.server.native.searchLocations` 指向的任何静态URL）.要使用本机配置文件，请使用 `spring.profiles.active=native` 启动配置服务器.

> 请记住使用 `file:` 前缀作为文件资源（没有前缀的默认值通常是类路径）.与任何Spring Boot配置一样，您可以嵌入 `${}` 样式的环境占位符，但请记住，Windows中的绝对路径需要额外的 `/` （例如， `file:///${user.home}/config-repo` ）.

> 默认值 `searchLocations` 与本地Spring Boot应用程序（即 `[classpath:/, classpath:/config, file:./, file:./config]` ）相同.这不会将 `application.properties` 从服务器暴露给所有客户端，因为服务器中存在的任何属性源在被发送到客户端之前都会被删除.

> A文件系统后端非常适合快速入门和测试.要在生产环境中使用它，您需要确保文件系统可靠并在Config Server的所有实例之间共享.

搜索位置可以包含 `{application}` ， `{profile}` 和 `{label}` 的占位符.通过这种方式，您可以隔离路径中的目录并选择对您有意义的策略（例如每个应用程序的子目录或每个配置文件的子目录）.

如果您不在搜索位置使用占位符，则此存储库还会将HTTP资源的 `{label}` 参数附加到搜索路径上的后缀，因此属性文件将从每个搜索位置 **and** 加载一个与该标签同名的子目录（标记的属性在Spring环境中优先.因此，没有占位符的默认行为与添加以 `/{label}/` 结尾的搜索位置相同.例如， `file:/tmp/config` 与 `file:/tmp/config,file:/tmp/config/{label}` 相同.可以通过设置 `spring.cloud.config.server.native.addLabelLocations=false` 来禁用此行为.

### 5.1.4 Vault后端

Spring Cloud Config Server还支持[Vault](https://www.vaultproject.io)作为后端.

----
Vault是一种安全访问机密的工具.秘密就是您要严格控制访问的任何内容，例如API密钥，密码，证书和其他敏感信息. Vault为任何机密提供统一的界面，同时提供严格的访问控制并记录详细的审计日志.

----

有关Vault的更多信息，请参阅[Vault quick start guide](https://www.vaultproject.io/intro/index.html).

要使配置服务器能够使用Vault后端，您可以使用 `vault` 配置文件运行配置服务器.例如，在配置服务器的 `application.properties` 中，您可以添加 `spring.profiles.active=vault` .

默认情况下，配置服务器假定您的Vault服务器在 `http://127.0.0.1:8200` 运行.它还假定后端的名称是 `secret` ，密钥是 `application` .所有这些默认值都可以在配置服务器的 `application.properties` 中配置.下表介绍了可配置的Vault属性：

|名称|默认值|
| ---- | ---- |
|主机| 127.0.0.1 |
|港口| 8200 |
|计划| http |
|后端|秘密|
| defaultKey | application |
| profileSeparator |，|
| kvVersion | 1 |
| skipSslValidation | false |
|超时| 5 |

|图片/ important.png |重要|
| ---- | ---- |
|上表中的所有属性都必须以 `spring.cloud.config.server.vault` 为前缀. |

所有可配置的属性都可以在 `org.springframework.cloud.config.server.environment.VaultEnvironmentRepository` 中找到.

Vault 0.10.0引入了一个版本化的键值后端（k / v后端版本2），它暴露了与早期版本不同的API，它现在需要在挂载路径和实际上下文路径之间使用 `data/` 并在 `data` 对象中包装秘密.设置 `kvVersion=2` 将考虑到这一点.

在配置服务器运行时，您可以向服务器发出HTTP请求以从Vault后端检索值.为此，您需要Vault服务器的令牌.

首先，在Vault中放置一些数据，如以下示例所示：

```java
$ vault write secret/application foo=bar baz=bam
$ vault write secret/myapp foo=myappsbar
```

其次，向配置服务器发出HTTP请求以检索值，如以下示例所示：

`$ curl -X "GET" "http://localhost:8888/myapp/default" -H "X-Config-Token: yourtoken"` 

您应该看到类似于以下内容的响应：

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

#### Multiple属性来源

使用Vault时，您可以为应用程序提供多个属性源.例如，假设您已将数据写入Vault中的以下路径：

```java
secret/myApp,dev
secret/myApp
secret/application,dev
secret/application
```

写入 `secret/application` 的属性可用于[all applications using the Config Server]().名为 `myApp` 的应用程序将具有写入 `secret/myApp` 和 `secret/application` 的任何属性.当 `myApp` 启用了 `dev` 配置文件时，写入所有上述路径的属性将可用，其中列表中第一个路径中的属性优先于其他路径.

### 5.1.5通过代理访问后端

配置服务器可以通过HTTP或HTTPS代理访问Git或Vault后端.通过 `proxy.http` 和 `proxy.https` 下的设置可以控制Git或Vault的此行为.这些设置是每个存储库，因此如果您使用[composite environment repository](multi__spring_cloud_config_server.html#composite-environment-repositories)，则必须单独为复合中的每个后端配置代理设置.如果使用需要单独的代理服务器用于HTTP和HTTPS URL的网络，则可以为单个后端配置HTTP和HTTPS代理设置.

下表描述了HTTP和HTTPS代理的代理配置属性.所有这些属性必须以 `proxy.http` 或 `proxy.https` 作为前缀.

**Table 5.2. Proxy Configuration Properties** 

|properties名称|备注|
| ---- | ---- |
|  **host**  |代理的主机. |
|  **port**  |用于访问代理的端口. |
|  **nonProxyHosts**  |配置服务器应在代理外部访问的任何主机.如果为 `proxy.http.nonProxyHosts` 和 `proxy.https.nonProxyHosts` 都提供了值，则将使用 `proxy.http` 值. |
|  **username**  |用于向代理进行身份验证的用户名.如果为 `proxy.http.username` 和.提供了值 `proxy.https.username` ，将使用 `proxy.http` 值. |
|  **password**  |用于向代理进行身份验证的密码.如果为 `proxy.http.password` 和 `proxy.https.password` 都提供了值，则将使用 `proxy.http` 值. |

以下配置使用HTTPS代理访问Git存储库.

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

### 5.1.6与所有应用程序共享配置

所有应用程序之间的共享配置因您采用的方法而异，如以下主题中所述：

- [the section called “File Based Repositories”](multi__spring_cloud_config_server.html#spring-cloud-config-server-file-based-repositories)

- [the section called “Vault Server”](multi__spring_cloud_config_server.html#spring-cloud-config-server-vault-server)

#### File Based Repositories

使用基于文件（git，svn和native）的存储库，所有客户端应用程序之间共享文件名在 `application*` （ `application.properties` ， `application.yml` ， `application-*.properties` 等）中的资源.您可以使用具有这些文件名的资源来配置全局默认值，并根据需要由特定于应用程序的文件覆盖它们.

#_property_overrides [property overrides]功能也可用于设置全局默认值，允许占位符应用程序在本地覆盖它们.

> 使用“本机”配置文件（本地文件系统后端），您应该使用不属于服务器自身配置的显式搜索位置.否则，默认搜索位置中的 `application*` 资源将被删除，因为它们是服务器的一部分.

#### Vault服务器

使用Vault作为后端时，您可以通过在 `secret/application` 中放置配置来与所有应用程序共享配置.例如，如果运行以下Vault命令，则使用配置服务器的所有应用程序将具有可用的属性 `foo` 和 `baz` ：

```java
$ vault write secret/application foo=bar baz=bam
```

### 5.1.7 JDBC后端

Spring Cloud Config Server支持JDBC（关系数据库）作为配置属性的后端.您可以通过将 `spring-jdbc` 添加到类路径并使用 `jdbc` 配置文件或添加 `JdbcEnvironmentRepository` 类型的bean来启用此功能.如果在类路径中包含正确的依赖项（有关详细信息，请参阅用户指南），Spring Boot会配置数据源.

数据库需要有一个名为 `PROPERTIES` 的表，其中包含名为 `APPLICATION` ， `PROFILE` 和 `LABEL` （通常为 `Environment` 含义）的列，以及 `Properties` 样式中键和值对的 `KEY` 和 `VALUE` .所有字段都是Java中的String类型，因此您可以将它们设置为 `VARCHAR` ，无论您需要多长.属性值的行为方式与它们来自名为 `{application}-{profile}.properties` 的Spring Boot属性文件（包括所有加密和解密）的行为相同，后者将作为后处理步骤（即不直接在存储库实现中）应用.

### 5.1.8复合环境知识库

在某些情况下，您可能希望从多个环境存储库中提取配置数据.为此，您可以在配置服务器的应用程序属性或YAML文件中启用 `composite` 配置文件.例如，如果要从Subversion存储库以及两个Git存储库中提取配置数据，则可以为配置服务器设置以下属性：

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

使用此配置，优先级由 `composite` 键下列出存储库的顺序决定.在上面的示例中，首先列出Subversion存储库，因此在Subversion存储库中找到的值将覆盖在其中一个Git存储库中为相同属性找到的值.在为 `walter`  Git存储库中的相同属性找到的值之前，将使用 `rex`  Git存储库中找到的值.

如果只想从每个不同类型的存储库中提取配置数据，则可以在配置服务器的应用程序属性或YAML文件中启用相应的配置文件，而不是 `composite` 配置文件.例如，如果要从单个Git存储库和单个HashiCorp Vault服务器提取配置数据，可以为配置服务器设置以下属性：

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

使用此配置，优先级可以由 `order` 属性确定.您可以使用 `order` 属性指定所有存储库的优先级顺序.  `order` 属性的数值越低，它具有的优先级越高.存储库的优先级顺序有助于解决包含相同属性值的存储库之间的任何潜在冲突.

> 如果您的复合环境包含Vault服务器，如上例所示，则必须在对配置服务器发出的每个请求中包含Vault令牌.见[Vault Backend](multi__spring_cloud_config_server.html#vault-backend).

> 从环境存储库检索值时出现任何类型的故障都会导致整个复合环境失败.

> 使用复合环境时，所有存储库都必须包含相同的标签.如果您具有与前面示例中的环境类似的环境，并且您使用 `master` 标签请求配置数据，但Subversion存储库不包含名为 `master` 的分支，则整个请求将失败.

#### Custom复合环境存储库

在除了使用Spring Cloud中的一个环境存储库之外，您还可以提供自己的 `EnvironmentRepository`  bean作为复合环境的一部分.为此，您的bean必须实现 `EnvironmentRepository` 接口.如果要在复合环境中控制自定义 `EnvironmentRepository` 的优先级，还应实现 `Ordered` 接口并覆盖 `getOrdered` 方法.如果未实现 `Ordered` 接口，则 `EnvironmentRepository` 的优先级最低.

### 5.1.9property覆盖

Config Server具有“覆盖”功能，允许操作员为所有应用程序提供配置属性.使用普通Spring Boot挂钩的应用程序不会意外更改被覆盖的属性.要声明覆盖，请将名称 - 值对的映射添加到 `spring.cloud.config.server.overrides` ，如以下示例所示：

```java
spring:
cloud:
config:
server:
overrides:
foo: bar
```

上述示例使所有作为配置客户端的应用程序读取 `foo=bar` ，与其自身配置无关.

> A配置系统无法强制应用程序以任何特定方式使用配置数据.因此，覆盖是不可执行的.但是，它们确实为Spring Cloud Config客户端提供了有用的默认行为.

> Normally，带有 `${}` 的Spring环境占位符可以使用反斜杠（ `\` ）转义（并在客户端上解析）以转义 `$` 或 `{` .例如， `\${app.foo:bar}` 解析为 `bar` ，除非应用程序提供自己的 `app.foo` .

> 在YAML中，您无需转义反斜杠本身.但是，在属性文件中，在服务器上配置替代时，需要转义反斜杠.

您可以通过在远程存储库中设置 `spring.cloud.config.overrideNone=true` 标志（默认值为false），将客户端中所有覆盖的优先级更改为更像默认值，让应用程序在环境变量或系统属性中提供自己的值.

## 5.2Health指标

Config Server附带一个运行状况指示器，用于检查配置的 `EnvironmentRepository` 是否正常工作.默认情况下，它会向 `EnvironmentRepository` 请求名为 `app` 的应用程序， `default` 配置文件以及 `EnvironmentRepository` 实现提供的默认标签.

您可以配置运行状况指示器以检查更多应用程序以及自定义配置文件和自定义标签，如以下示例所示：

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

您可以通过设置 `spring.cloud.config.server.health.enabled=false` 来禁用Health指示器.

## 5.3安全

您可以以对您有意义的任何方式保护您的Config Server（从物理网络安全到OAuth2承载令牌），因为Spring Security和Spring Boot为许多安全安排提供支持.

要使用默认的Spring Boot配置的HTTP Basic安全性，请在类路径中包含Spring Security（例如，通过 `spring-boot-starter-security` ）.默认值为 `user` 的用户名和随机生成的密码.随机密码在实践中没有用，因此我们建议您配置密码（通过设置 `spring.security.user.password` ）并对其进行加密（有关如何执行此操作的说明，请参阅下文）.

## 5.4加密和解密

|图片/ important.png |重要|
| ---- | ---- |
|要使用加密和解密功能，您需要在JVM中安装全功能JCE（默认情况下不包括它）.您可以从Oracle下载“Java Cryptography Extension（JCE）Unlimited Strength Jurisdiction Policy Files”并按照安装说明进行操作（实质上，您需要将JRE lib / security目录中的两个策略文件替换为您下载的那些）. |

如果远程属性源包含加密内容（以 `{cipher}` 开头的值），则在通过HTTP发送到客户端之前对它们进行解密.此设置的主要优点是，属性值在“静止”时不必是纯文本格式（例如，在git存储库中）.如果无法解密某个值，则会从属性源中删除该值，并使用相同的键添加其他属性，但前缀为 `invalid` ，且值为“不适用”（通常为 `<n/a>` ）.这主要是为了防止密文被用作密码并意外泄露.

如果为配置客户端应用程序设置远程配置存储库，则它可能包含类似于以下内容的 `application.yml` ：

**application.yml.** 

```java
spring:
datasource:
username: dbuser
password: '{cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ'
```

.properties文件中的加密值不得用引号括起来.否则，该值不会被解密.以下示例显示了可行的值：

**application.properties.** 

```java
spring.datasource.username: dbuser
spring.datasource.password: {cipher}FKSAJDFGYOS8F7GLHAKERGFHLSAJ
```

您可以安全地将此纯文本推送到共享的git存储库，并且密码保密.

服务器还公开 `/encrypt` 和 `/decrypt` endpoints（假设这些endpoints是安全的并且只能由授权代理访问）.如果编辑远程配置文件，则可以使用配置服务器通过POST到 `/encrypt` endpoints来加密值，如以下示例所示：

```java
$ curl localhost:8888/encrypt -d mysecret
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
```

> 如果您加密的值中包含需要进行URL编码的字符，则应使用 `--data-urlencode` 选项 `curl` 来制作确保它们编码正确.

> Be确保不在加密值中包含任何curl命令统计信息.将值输出到文件可以帮助避免此问题.

通过 `/decrypt` 也可以进行逆操作（前提是服务器配置了对称密钥或完整密钥对），如以下示例所示：

```java
$ curl localhost:8888/decrypt -d 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

> 如果使用curl进行测试，则使用 `--data-urlencode` （而不是 `-d` ）或设置显式 `Content-Type: text/plain` 以确保在有特殊字符时（''特别棘手）curl正确编码数据.

获取加密值并添加 `{cipher}` 前缀，然后再将其放入YAML或属性文件中，然后再提交并将其推送到远程（可能不安全）存储.

`/encrypt` 和 `/decrypt` endpoints也都接受 `/*/{name}/{profiles}` 形式的路径，当客户端调用主环境资源时，它可用于控制每个应用程序（名称）和每个配置文件的加密.

> 以这种精细的方式控制加密，您还必须提供 `@Bean` 类型 `TextEncryptorLocator` ，为每个名称和配置文件创建不同的加密器.默认情况下提供的那个不会这样做（所有加密都使用相同的密钥）.

`spring` 命令行客户端（安装了Spring Cloud CLI扩展）也可用于加密和解密，如以下示例所示：

```java
$ spring encrypt mysecret --key foo
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
$ spring decrypt --key foo 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

要在文件中使用密钥（例如用于加密的RSA公钥），请在密钥值前加上“@”并提供文件路径，如以下示例所示：

```java
$ spring encrypt mysecret --key @${HOME}/.ssh/id_rsa.pub
AQAjPgt3eFZQXwt8tsHAVv/QHiY5sI2dRcR+...
```

>   `--key` 参数是必需的（尽管有 `--` 前缀）.

## 5.5密钥管理

Config Server可以使用对称（共享）密钥或非对称密钥（RSA密钥对）.非对称选择在安全性方面更优越，但使用对称密钥通常更方便，因为它是在 `bootstrap.properties` 中配置的单个属性值.

要配置对称密钥，需要将 `encrypt.key` 设置为秘密字符串（或使用 `ENCRYPT_KEY` 环境变量使其不受纯文本配置文件的影响）.

要配置非对称密钥，您可以将密钥设置为PEM编码的文本值（在 `encrypt.key` 中）或使用密钥库（例如由JDK附带的 `keytool` 实用程序创建的密钥库）.下表描述了密钥库属性：

|地产|简介|
| ---- | ---- |
|  `encrypt.keyStore.location`  |包含 `Resource` 位置|
|  `encrypt.keyStore.password`  |保存解锁密钥库的密码
|  `encrypt.keyStore.alias`  |标识要使用的商店中的哪个键

加密是使用公钥完成的，并且需要私钥进行解密.因此，原则上，如果只想加密（并准备使用私钥本地解密值），则只能在服务器中配置公钥.实际上，您可能不希望在本地进行解密，因为它会围绕所有客户端传播密钥管理过程，而不是将其集中在服务器中.另一方面，如果您的配置服务器相对不安全且只有少数客户端需要加密属性，那么它可能是一个有用的选项.

## 5.6创建用于测试的密钥库

要创建用于测试的密钥库，可以使用类似于以下内容的命令：

```java
$ keytool -genkeypair -alias mytestkey -keyalg RSA \
-dname "CN=Web Server,OU=Unit,O=Organization,L=City,S=State,C=US" \
-keypass changeme -keystore server.jks -storepass letmein
```

将 `server.jks` 文件放在类路径中（例如），然后在 `bootstrap.yml` 中，为Config Server创建以下设置：

```java
encrypt:
keyStore:
location: classpath:/server.jks
password: letmein
alias: mytestkey
secret: changeme
```

## 5.7使用多个键和键旋转

除了加密属性值中的 `{cipher}` 前缀之外，Config Server还会在（Base64编码）密码文本开始之前查找零个或多个 `{name:value}` 前缀.密钥传递给 `TextEncryptorLocator` ，它可以执行为密码定位 `TextEncryptor` 所需的任何逻辑.如果已配置密钥库（ `encrypt.keystore.location` ），则默认定位器将查找具有 `key` 前缀提供的别名的密钥，其密码文本类似于以下内容：

```java
foo:
bar: `{cipher}{key:testkey}...`
```

定位器查找名为"testkey"的密钥.也可以通过在前缀中使用 `{secret:…}` 值来提供密钥.但是，如果未提供，则默认使用密钥库密码（这是您在构建密钥组时获得的密码，并且未指定密钥）.如果您确实提供了机密，则还应使用自定义 `SecretLocator` 加密机密.

当密钥仅用于加密几个字节的配置数据时（也就是说，它们没有在其他地方使用），在加密方面几乎不需要密钥轮换.但是，您可能偶尔需要更改密钥（例如，在发生安全漏洞的情况下）.在这种情况下，所有客户端都需要更改其源配置文件（例如，在git中）并在所有密码中使用新的 `{key:…}` 前缀.请注意，客户端需要首先检查Config Server密钥库中的密钥别名是否可用.

> 如果您想让Config Server处理所有加密和解密， `{name:value}` 前缀也可以作为发布到 `/encrypt` endpoints的纯文本添加，.

## 5.8提供加密属性

有时您希望客户端在本地解密配置，而不是在服务器中执行此操作.在这种情况下，如果您提供 `encrypt.*` 配置来查找密钥，您仍然可以拥有 `/encrypt` 和 `/decrypt` endpoints，但是您需要通过在 `bootstrap.[yml|properties]` 中放置 `spring.cloud.config.server.encrypt.enabled=false` 来明确关闭传出属性的解密.如果您不关心endpoints，则在未配置密钥或启用标志的情况下应该可以正常工作.


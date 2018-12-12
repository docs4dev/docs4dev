## 64.安装Spring Boot应用程序

除了使用 `java -jar` 运行Spring Boot应用程序之外，还可以为Unix系统创建完全可执行的应用程序.完全可执行的jar可以像任何其他可执行二进制文件一样执行，也可以[registered with init.d or systemd](deployment-install.html#deployment-service).这使得在常见的生产环境环境中安装和管理Spring Boot应用程序变得非常容易.

> 完全可执行的jar通过在文件的前面嵌入额外的脚本来工作.目前，某些工具不接受此格式，因此您可能无法始终使用此技术.例如， `jar -xf` 可能无法提取已完全可执行的jar或war.建议您只有在打算直接执行jar或war时才能使jar或war完全可执行，而不是使用 `java -jar` 运行它或将其部署到servlet容器.

要使用Maven创建“完全可执行”jar，请使用以下插件配置：

```xml
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<configuration>
		<executable>true</executable>
	</configuration>
</plugin>
```

以下示例显示了等效的Gradle配置：

```java
bootJar {
	launchScript()
}
```

然后，您可以通过键入 `./my-application.jar` （其中 `my-application` 是工件的名称）来运行您的应用程序.包含jar的目录用作应用程序的工作目录.

## 64.1支持的操作系统

默认脚本支持大多数Linux发行版，并在CentOS和Ubuntu上进行测试.其他平台，如OS X和FreeBSD，需要使用自定义 `embeddedLaunchScript` .

## 64.2 Unix / Linux服务

通过使用 `init.d` 或 `systemd` ，可以轻松地将Spring Boot应用程序作为Unix / Linux服务启动.

### 64.2.1作为init.d服务安装（系统V）

如果您将Spring Boot的Maven或Gradle插件配置为生成[fully executable jar](deployment-install.html)，并且您不使用自定义 `embeddedLaunchScript` ，则您的应用程序可用作 `init.d` 服务.为此，将jar符号链接到 `init.d` 以支持标准 `start` ， `stop` ， `restart` 和 `status` 命令.

该脚本支持以下功能：

- 以拥有jar文件的用户身份启动服务

- 使用 `/var/run/<appname>/<appname>.pid` 跟踪应用程序的PID

- Writes控制台日志到 `/var/log/<appname>.log` 

假设您在 `/var/myapp` 中安装了Spring Boot应用程序，要将Spring Boot应用程序安装为 `init.d` 服务，请创建一个符号链接，如下所示：

```java
$ sudo ln -s /var/myapp/myapp.jar /etc/init.d/myapp
```

安装后，您可以按常规方式启动和停止服务.例如，在基于Debian的系统上，您可以使用以下命令启动它：

```java
$ service myapp start
```

> 如果您的应用程序无法启动，请检查写入 `/var/log/<appname>.log` 的日志文件是否有错误.

您还可以使用标准操作系统工具标记应用程序以自动启动.例如，在Debian上，您可以使用以下命令：

```xml
$ update-rc.d myapp defaults <priority>
```

#### 确保init.d服务

> 以下是一套有关如何保护作为init.d服务运行的Spring Boot应用程序的指南.它并不是为了强化应用程序及其运行环境而应该做的所有事情的详尽列表.

当以root身份执行时，就像root用于启动init.d服务的情况一样，默认可执行脚本以拥有jar文件的用户身份运行应用程序.您永远不应该将Spring Boot应用程序作为 `root` 运行，因此您的应用程序的jar文件永远不应该由root拥有.而是创建一个特定用户来运行您的应用程序并使用 `chown` 使其成为jar文件的所有者，如以下示例所示：

```java
$ chown bootapp:bootapp your-app.jar
```

在这种情况下，默认可执行脚本运行应用程序作为 `bootapp` 用户.

> 为了减少应用程序用户帐户被盗用的可能性，您应该考虑阻止它使用登录shell.例如，您可以将帐户的shell设置为 `/usr/sbin/nologin` .

您还应该采取措施来防止修改应用程序的jar文件.首先，配置其权限，使其无法写入，只能由其所有者读取或执行，如以下示例所示：

```java
$ chmod 500 your-app.jar
```

其次，如果您的应用程序或运行它的帐户受到损害，您还应该采取措施限制损害.如果攻击者确实获得了访问权限，他们可以使jar文件可写并更改其内容.防止这种情况的一种方法是使用 `chattr` 使其不可变，如以下示例所示：

```java
$ sudo chattr +i your-app.jar
```

这将阻止任何用户（包括root）修改jar.

如果root用于控制应用程序的服务，并且您[use a .conf file](deployment-install.html#deployment-script-customization-conf-file)自定义其启动，则root用户将读取并评估 `.conf` 文件.它应该得到相应的保护.使用 `chmod` 以便文件只能由所有者读取并使用 `chown` 使root成为所有者，如以下示例所示：

```java
$ chmod 400 your-app.conf
$ sudo chown root:root your-app.conf
```

### 64.2.2作为systemd服务安装

`systemd` 是System V init系统的后继者，现在正被许多现代Linux发行版使用.虽然您可以继续将 `init.d` 脚本与 `systemd` 一起使用，但也可以使用 `systemd` 'service'脚本启动Spring Boot应用程序.

假设您在 `/var/myapp` 中安装了Spring Boot应用程序，要将Spring Boot应用程序安装为 `systemd` 服务，请创建一个名为 `myapp.service` 的脚本并将其放在 `/etc/systemd/system` 目录中.以下脚本提供了一个示例：

```java
[Unit]
Description=myapp
After=syslog.target

[Service]
User=myapp
ExecStart=/var/myapp/myapp.jar
SuccessExitStatus=143

[Install]
WantedBy=multi-user.target
```

|图片/ important.png |重要|
| ---- | ---- |
|请记住更改应用程序的 `Description` ， `User` 和 `ExecStart` 字段. |

>   `ExecStart` 字段未声明脚本操作命令，这意味着默认使用 `run` 命令.

请注意，与作为 `init.d` 服务运行时不同，运行应用程序的用户，PID文件和控制台日志文件由 `systemd` 本身管理，因此必须使用“service”脚本中的相应字段进行配置.有关详细信息，请参阅[service unit configuration man page](https://www.freedesktop.org/software/systemd/man/systemd.service.html).

要将应用程序标记为在系统引导时自动启动，请使用以下命令：

```java
$ systemctl enable myapp.service
```

有关详细信息，请参阅 `man systemctl` .

### 64.2.3自定义启动脚本

可以通过多种方式自定义Maven或Gradle插件编写的默认嵌入式启动脚本.对于大多数人来说，使用默认脚本和一些自定义通常就足够了.如果您发现无法自定义所需内容，请使用 `embeddedLaunchScript` 选项完全编写自己的文件.

#### 在写入时自定义启动脚本

在将脚本写入jar文件时自定义启动脚本的元素通常是有意义的.例如，init.d脚本可以提供“描述”.由于您事先了解了描述（并且不需要更改），因此您可以在生成jar时提供它.

要自定义书面元素，请使用Spring Boot Maven或Gradle插件的 `embeddedLaunchScriptProperties` 选项.

默认脚本支持以下属性替换：

|名称|描述| Gradle默认| Maven默认|
| ---- | ---- | ---- | ---- |
|  `mode`  |脚本模式. |  `auto`  |  `auto`  |
|  `initInfoProvides`  |“INIT INFO”的 `Provides` 部分|  `${task.baseName}`  |  `${project.artifactId}`  |
|  `initInfoRequiredStart`  |  `Required-Start` “INIT INFO”部分. |  `$remote_fs $syslog $network`  |  `$remote_fs $syslog $network`  |
|  `initInfoRequiredStop`  |  `Required-Stop` “INIT INFO”部分. |  `$remote_fs $syslog $network`  |  `$remote_fs $syslog $network`  |
|  `initInfoDefaultStart`  |  `Default-Start` “INIT INFO”部分. |  `2 3 4 5`  |  `2 3 4 5`  |
|  `initInfoDefaultStop`  |  `Default-Stop` “INIT INFO”部分. |  `0 1 6`  |  `0 1 6`  |
|  `initInfoShortDescription`  |  `Short-Description` “INIT INFO”部分. |单行版 `${project.description}` （回落到 `${task.baseName}` ）|  `${project.name}`  |
|  `initInfoDescription`  |  `Description` “INIT INFO”部分. |  `${project.description}` （回落到 `${task.baseName}` ）|  `${project.description}` （回落到 `${project.name}` ）|
|  `initInfoChkconfig`  |  `chkconfig` “INIT INFO”部分|  `2345 99 01`  |  `2345 99 01`  |
|  `confFolder`  |  `CONF_FOLDER`  |包含jar的文件夹的默认值|包含jar的文件夹|
|  `inlinedConfScript`  |引用应在默认启动脚本中内联的文件脚本.这可用于在加载任何外部配置文件之前设置环境变量，例如 `JAVA_OPTS`  | |
|  `logFolder`  |  `LOG_FOLDER` 的默认值.仅对 `init.d` 服务有效| |
|  `logFilename`  |  `LOG_FILENAME` 的默认值.仅对 `init.d` 服务有效| |
|  `pidFolder`  |  `PID_FOLDER` 的默认值.仅对 `init.d` 服务有效| |
|  `pidFilename`  |  `PID_FOLDER` 中PID文件名称的默认值.仅对 `init.d` 服务有效| |
|  `useStartStopDaemon`  |  `start-stop-daemon` 命令是否可用，应该用于控制进程|  `true`  |  `true`  |
|  `stopWaitTime`  |  `STOP_WAIT_TIME` 的默认值（以秒为单位）.仅对 `init.d` 服务有效| 60 | 60 |

#### 在运行时自定义脚本

对于在编写jar后需要自定义的脚本项，可以使用环境变量或[config file](deployment-install.html#deployment-script-customization-conf-file).

以下环境默认脚本支持属性：

|变量|说明|
| ---- | ---- |
|  `MODE`  |操作的“模式”.默认值取决于jar的构建方式，但通常是 `auto` （意味着它通过检查它是否是名为 `init.d` 的目录中的符号链接来尝试猜测它是否是init脚本）.您可以显式将其设置为 `service` ，以便 `stop|start|status|restart` 命令可以工作，或者如果要在前台运行脚本，则可以设置为 `run` . |
|  `USE_START_STOP_DAEMON`  |  `start-stop-daemon` 命令是否可用，应该用于控制进程.默认为 `true` . |
|  `PID_FOLDER`  | pid文件夹的根名称（默认为 `/var/run` ）. |
|  `LOG_FOLDER`  |放置日志文件的文件夹的名称（默认为 `/var/log` ）. |
|  `CONF_FOLDER`  |从中读取.conf文件的文件夹的名称（默认情况下与jar文件相同）. |
|  `LOG_FILENAME`  |  `LOG_FOLDER` 中的日志文件名（默认为 `<appname>.log` ）. |
|  `APP_NAME`  |应用程序的名称.如果jar从符号链接运行，则脚本会猜测应用程序名称.如果它不是符号链接或您想要显式设置应用程序名称，这可能很有用. |
|  `RUN_ARGS`  |传递给程序的参数（Spring Boot应用程序）. |
|  `JAVA_HOME`  |默认情况下使用 `PATH` 发现 `java` 可执行文件的位置，但如果 `$JAVA_HOME/bin/java` 处有可执行文件，则可以显式设置它. |
|  `JAVA_OPTS`  |启动时传递给JVM的选项. |
|  `JARFILE`  | jar文件的显式位置，以防脚本用于启动实际未嵌入的jar. |
|  `DEBUG`  |如果不为空，则在shell进程上设置 `-x` 标志，以便于在脚本中查看逻辑. |
|  `STOP_WAIT_TIME`  |在强制关闭之前停止应用程序时等待的时间（默认为 `60` ）. |

>   `PID_FOLDER` ， `LOG_FOLDER` 和 `LOG_FILENAME` 变量仅对 `init.d` 服务有效.对于 `systemd` ，使用“服务”脚本进行等效的自定义.有关详细信息，请参阅[service unit configuration man page](https://www.freedesktop.org/software/systemd/man/systemd.service.html).

除 `JARFILE` 和 `APP_NAME` 外，可以使用 `.conf` 文件配置上一节中列出的设置.该文件应该在jar文件的旁边，并且具有相同的名称，但后缀为 `.conf` 而不是 `.jar` .例如，名为 `/var/myapp/myapp.jar` 的jar使用名为 `/var/myapp/myapp.conf` 的配置文件，如以下示例所示：

**myapp.conf.** 

```java
JAVA_OPTS=-Xmx1024M
LOG_FOLDER=/custom/log/folder
```

> 如果您不喜欢在jar文件旁边有配置文件，可以设置 `CONF_FOLDER` 环境变量来自定义配置文件的位置.

要了解有关正确保护此文件的信息，请参阅[the guidelines for securing an init.d service](deployment-install.html#deployment-initd-service-securing).

## 64.3 Microsoft Windows服务

可以使用[winsw](https://github.com/kohsuke/winsw)将Spring Boot应用程序作为Windows服务启动.

A（[separately maintained sample](https://github.com/snicoll-scratches/spring-boot-daemon)）逐步介绍了如何为Spring Boot应用程序创建Windows服务.


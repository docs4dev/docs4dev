## 64. Installing Spring Boot Applications

In addition to running Spring Boot applications by using  `java -jar` , it is also possible to make fully executable applications for Unix systems. A fully executable jar can be executed like any other executable binary or it can be [registered with init.d or systemd](deployment-install.html#deployment-service). This makes it very easy to install and manage Spring Boot applications in common production environments.

> Fully executable jars work by embedding an extra script at the front of the file. Currently, some tools do not accept this format, so you may not always be able to use this technique. For example,  `jar -xf`  may silently fail to extract a jar or war that has been made fully executable. It is recommended that you make your jar or war fully executable only if you intend to execute it directly, rather than running it with  `java -jar`  or deploying it to a servlet container.

To create a ‘fully executable’ jar with Maven, use the following plugin configuration:

```xml
<plugin>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-maven-plugin</artifactId>
	<configuration>
		<executable>true</executable>
	</configuration>
</plugin>
```

The following example shows the equivalent Gradle configuration:

```java
bootJar {
	launchScript()
}
```

You can then run your application by typing  `./my-application.jar`  (where  `my-application`  is the name of your artifact). The directory containing the jar is used as your application’s working directory.

## 64.1 Supported Operating Systems

The default script supports most Linux distributions and is tested on CentOS and Ubuntu. Other platforms, such as OS X and FreeBSD, require the use of a custom  `embeddedLaunchScript` .

## 64.2 Unix/Linux Services

Spring Boot application can be easily started as Unix/Linux services by using either  `init.d`  or  `systemd` .

### 64.2.1 Installation as an init.d Service (System V)

If you configured Spring Boot’s Maven or Gradle plugin to generate a [fully executable jar](deployment-install.html), and you do not use a custom  `embeddedLaunchScript` , your application can be used as an  `init.d`  service. To do so, symlink the jar to  `init.d`  to support the standard  `start` ,  `stop` ,  `restart` , and  `status`  commands.

The script supports the following features:

- Starts the services as the user that owns the jar file

- Tracks the application’s PID by using  `/var/run/<appname>/<appname>.pid` 

- Writes console logs to  `/var/log/<appname>.log` 

Assuming that you have a Spring Boot application installed in  `/var/myapp` , to install a Spring Boot application as an  `init.d`  service, create a symlink, as follows:

```java
$ sudo ln -s /var/myapp/myapp.jar /etc/init.d/myapp
```

Once installed, you can start and stop the service in the usual way. For example, on a Debian-based system, you could start it with the following command:

```java
$ service myapp start
```

> If your application fails to start, check the log file written to  `/var/log/<appname>.log`  for errors.

You can also flag the application to start automatically by using your standard operating system tools. For example, on Debian, you could use the following command:

```xml
$ update-rc.d myapp defaults <priority>
```

#### Securing an init.d Service

> The following is a set of guidelines on how to secure a Spring Boot application that runs as an init.d service. It is not intended to be an exhaustive list of everything that should be done to harden an application and the environment in which it runs.

When executed as root, as is the case when root is being used to start an init.d service, the default executable script runs the application as the user who owns the jar file. You should never run a Spring Boot application as  `root` , so your application’s jar file should never be owned by root. Instead, create a specific user to run your application and use  `chown`  to make it the owner of the jar file, as shown in the following example:

```java
$ chown bootapp:bootapp your-app.jar
```

In this case, the default executable script runs the application as the  `bootapp`  user.

> To reduce the chances of the application’s user account being compromised, you should consider preventing it from using a login shell. For example, you can set the account’s shell to  `/usr/sbin/nologin` .

You should also take steps to prevent the modification of your application’s jar file. Firstly, configure its permissions so that it cannot be written and can only be read or executed by its owner, as shown in the following example:

```java
$ chmod 500 your-app.jar
```

Second, you should also take steps to limit the damage if your application or the account that’s running it is compromised. If an attacker does gain access, they could make the jar file writable and change its contents. One way to protect against this is to make it immutable by using  `chattr` , as shown in the following example:

```java
$ sudo chattr +i your-app.jar
```

This will prevent any user, including root, from modifying the jar.

If root is used to control the application’s service and you [use a .conf file](deployment-install.html#deployment-script-customization-conf-file) to customize its startup, the  `.conf`  file is read and evaluated by the root user. It should be secured accordingly. Use  `chmod`  so that the file can only be read by the owner and use  `chown`  to make root the owner, as shown in the following example:

```java
$ chmod 400 your-app.conf
$ sudo chown root:root your-app.conf
```

### 64.2.2 Installation as a systemd Service

`systemd`  is the successor of the System V init system and is now being used by many modern Linux distributions. Although you can continue to use  `init.d`  scripts with  `systemd` , it is also possible to launch Spring Boot applications by using  `systemd`  ‘service’ scripts.

Assuming that you have a Spring Boot application installed in  `/var/myapp` , to install a Spring Boot application as a  `systemd`  service, create a script named  `myapp.service`  and place it in  `/etc/systemd/system`  directory. The following script offers an example:

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

|images/important.png|Important|
|----|----|
|Remember to change the  `Description` ,  `User` , and  `ExecStart`  fields for your application. |

> The  `ExecStart`  field does not declare the script action command, which means that the  `run`  command is used by default.

Note that, unlike when running as an  `init.d`  service, the user that runs the application, the PID file, and the console log file are managed by  `systemd`  itself and therefore must be configured by using appropriate fields in the ‘service’ script. Consult the [service unit configuration man page](https://www.freedesktop.org/software/systemd/man/systemd.service.html) for more details.

To flag the application to start automatically on system boot, use the following command:

```java
$ systemctl enable myapp.service
```

Refer to  `man systemctl`  for more details.

### 64.2.3 Customizing the Startup Script

The default embedded startup script written by the Maven or Gradle plugin can be customized in a number of ways. For most people, using the default script along with a few customizations is usually enough. If you find you cannot customize something that you need to, use the  `embeddedLaunchScript`  option to write your own file entirely.

#### Customizing the Start Script when It Is Written

It often makes sense to customize elements of the start script as it is written into the jar file. For example, init.d scripts can provide a “description”. Since you know the description up front (and it need not change), you may as well provide it when the jar is generated.

To customize written elements, use the  `embeddedLaunchScriptProperties`  option of the Spring Boot Maven or Gradle plugins.

The following property substitutions are supported with the default script:

|Name|Description|Gradle default|Maven default|
|----|----|----|----|
| `mode`  |The script mode. | `auto`  | `auto`  |
| `initInfoProvides`  |The  `Provides`  section of “INIT INFO” | `${task.baseName}`  | `${project.artifactId}`  |
| `initInfoRequiredStart`  | `Required-Start`  section of “INIT INFO”. | `$remote_fs $syslog $network`  | `$remote_fs $syslog $network`  |
| `initInfoRequiredStop`  | `Required-Stop`  section of “INIT INFO”. | `$remote_fs $syslog $network`  | `$remote_fs $syslog $network`  |
| `initInfoDefaultStart`  | `Default-Start`  section of “INIT INFO”. | `2 3 4 5`  | `2 3 4 5`  |
| `initInfoDefaultStop`  | `Default-Stop`  section of “INIT INFO”. | `0 1 6`  | `0 1 6`  |
| `initInfoShortDescription`  | `Short-Description`  section of “INIT INFO”. |Single-line version of  `${project.description}`  (falling back to  `${task.baseName}` ) | `${project.name}`  |
| `initInfoDescription`  | `Description`  section of “INIT INFO”. | `${project.description}`  (falling back to  `${task.baseName}` ) | `${project.description}`  (falling back to  `${project.name}` ) |
| `initInfoChkconfig`  | `chkconfig`  section of “INIT INFO” | `2345 99 01`  | `2345 99 01`  |
| `confFolder`  |The default value for  `CONF_FOLDER`  |Folder containing the jar |Folder containing the jar |
| `inlinedConfScript`  |Reference to a file script that should be inlined in the default launch script. This can be used to set environmental variables such as  `JAVA_OPTS`  before any external config files are loaded | | |
| `logFolder`  |Default value for  `LOG_FOLDER` . Only valid for an  `init.d`  service | | |
| `logFilename`  |Default value for  `LOG_FILENAME` . Only valid for an  `init.d`  service | | |
| `pidFolder`  |Default value for  `PID_FOLDER` . Only valid for an  `init.d`  service | | |
| `pidFilename`  |Default value for the name of the PID file in  `PID_FOLDER` . Only valid for an  `init.d`  service | | |
| `useStartStopDaemon`  |Whether the  `start-stop-daemon`  command, when it’s available, should be used to control the process | `true`  | `true`  |
| `stopWaitTime`  |Default value for  `STOP_WAIT_TIME`  in seconds. Only valid for an  `init.d`  service |60 |60 |

#### Customizing a Script When It Runs

For items of the script that need to be customized after the jar has been written, you can use environment variables or a [config file](deployment-install.html#deployment-script-customization-conf-file).

The following environment properties are supported with the default script:

|Variable|Description|
|----|----|
| `MODE`  |The “mode” of operation. The default depends on the way the jar was built but is usually  `auto`  (meaning it tries to guess if it is an init script by checking if it is a symlink in a directory called  `init.d` ). You can explicitly set it to  `service`  so that the  `stop|start|status|restart`  commands work or to  `run`  if you want to run the script in the foreground. |
| `USE_START_STOP_DAEMON`  |Whether the  `start-stop-daemon`  command, when it’s available, should be used to control the process. Defaults to  `true` . |
| `PID_FOLDER`  |The root name of the pid folder ( `/var/run`  by default). |
| `LOG_FOLDER`  |The name of the folder in which to put log files ( `/var/log`  by default). |
| `CONF_FOLDER`  |The name of the folder from which to read .conf files (same folder as jar-file by default). |
| `LOG_FILENAME`  |The name of the log file in the  `LOG_FOLDER`  ( `<appname>.log`  by default). |
| `APP_NAME`  |The name of the app. If the jar is run from a symlink, the script guesses the app name. If it is not a symlink or you want to explicitly set the app name, this can be useful. |
| `RUN_ARGS`  |The arguments to pass to the program (the Spring Boot app). |
| `JAVA_HOME`  |The location of the  `java`  executable is discovered by using the  `PATH`  by default, but you can set it explicitly if there is an executable file at  `$JAVA_HOME/bin/java` . |
| `JAVA_OPTS`  |Options that are passed to the JVM when it is launched. |
| `JARFILE`  |The explicit location of the jar file, in case the script is being used to launch a jar that it is not actually embedded. |
| `DEBUG`  |If not empty, sets the  `-x`  flag on the shell process, making it easy to see the logic in the script. |
| `STOP_WAIT_TIME`  |The time in seconds to wait when stopping the application before forcing a shutdown ( `60`  by default). |

> The  `PID_FOLDER` ,  `LOG_FOLDER` , and  `LOG_FILENAME`  variables are only valid for an  `init.d`  service. For  `systemd` , the equivalent customizations are made by using the ‘service’ script. See the [service unit configuration man page](https://www.freedesktop.org/software/systemd/man/systemd.service.html) for more details.

With the exception of  `JARFILE`  and  `APP_NAME` , the settings listed in the preceding section can be configured by using a  `.conf`  file. The file is expected to be next to the jar file and have the same name but suffixed with  `.conf`  rather than  `.jar` . For example, a jar named  `/var/myapp/myapp.jar`  uses the configuration file named  `/var/myapp/myapp.conf` , as shown in the following example:

**myapp.conf.**  

```java
JAVA_OPTS=-Xmx1024M
LOG_FOLDER=/custom/log/folder
```

> If you do not like having the config file next to the jar file, you can set a  `CONF_FOLDER`  environment variable to customize the location of the config file.

To learn about securing this file appropriately, see [the guidelines for securing an init.d service](deployment-install.html#deployment-initd-service-securing).

## 64.3 Microsoft Windows Services

A Spring Boot application can be started as a Windows service by using [winsw](https://github.com/kohsuke/winsw).

A ([separately maintained sample](https://github.com/snicoll-scratches/spring-boot-daemon)) describes step-by-step how you can create a Windows service for your Spring Boot application.


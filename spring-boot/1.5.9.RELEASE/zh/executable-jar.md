## Appendix E.可执行的Jar格式

`spring-boot-loader` 模块允许Spring Boot支持可执行jar和war文件.如果您使用Maven插件或Gradle插件，则会自动生成可执行jar，您通常不需要知道它们如何工作的详细信息.

如果您需要从不同的构建系统创建可执行jar，或者如果您只是对底层技术感到好奇，本节提供了一些背景知识.

## E.1嵌套JAR

Java没有提供任何标准方法来加载嵌套的jar文件（即jar文件本身包含在jar中）.如果您需要分发可以从命令行运行而不解压缩的自包含应用程序，则可能会出现问题.

为了解决这个问题，许多开发人员使用“阴影”jar.一个带阴影的jar将所有类别的所有类别打包成一个“超级jar”.阴影jar的问题在于很难看出哪些库实际上在您的应用程序中.如果在多个jar中使用相同的文件名（但具有不同的内容），也可能会有问题. Spring Boot采用不同的方法，让你直接嵌套jar.

### E.1.1可执行Jar文件结构

Spring Boot Loader兼容的jar文件应按以下方式构建：

```xml
example.jar
|
+-META-INF
|  +-MANIFEST.MF
+-org
|  +-springframework
|     +-boot
|        +-loader
|           +-<spring boot loader classes>
+-BOOT-INF
+-classes
|  +-mycompany
|     +-project
|        +-YourClasses.class
+-lib
+-dependency1.jar
+-dependency2.jar
```

应用程序类应放在嵌套的 `BOOT-INF/classes` 目录中.依赖项应放在嵌套的 `BOOT-INF/lib` 目录中.

### E.1.2可执行的战争文件结构

Spring Boot Loader兼容的war文件应按以下方式构建：

```xml
example.war
|
+-META-INF
|  +-MANIFEST.MF
+-org
|  +-springframework
|     +-boot
|        +-loader
|           +-<spring boot loader classes>
+-WEB-INF
+-classes
|  +-com
|     +-mycompany
|        +-project
|           +-YourClasses.class
+-lib
|  +-dependency1.jar
|  +-dependency2.jar
+-lib-provided
+-servlet-api.jar
+-dependency3.jar
```

依赖项应放在嵌套的 `WEB-INF/lib` 目录中.运行嵌入式时所需的任何依赖项，但在部署到传统Web容器时不需要，应放在 `WEB-INF/lib-provided` 中.

## E.2 Spring Boot的“JarFile”类

用于支持加载嵌套jar的核心类是 `org.springframework.boot.loader.jar.JarFile` .它允许您从标准jar文件或嵌套子jar数据中加载jar内容.首次加载时，每个 `JarEntry` 的位置都映射到外部jar的物理文件偏移量，如以下示例所示：

```java
myapp.jar
+-------------------+-------------------------+
| /BOOT-INF/classes | /BOOT-INF/lib/mylib.jar |
|+-----------------+||+-----------+----------+|
||     A.class      |||  B.class  |  C.class ||
|+-----------------+||+-----------+----------+|
+-------------------+-------------------------+
^                    ^           ^
0063                 3452        3980
```

上面的示例显示如何在位于 `0063` 的 `myapp.jar`   `/BOOT-INF/classes` 中找到 `A.class` .来自嵌套jar的 `B.class` 实际上可以在位置 `3452` 的 `myapp.jar` 中找到，而 `C.class` 位于 `3980` 位置.

有了这些信息，我们可以通过寻找外部jar的适当部分来加载特定的嵌套条目.我们不需要解压缩归档文件，也不需要将所有条目数据读入内存.

### E.2.1与标准Java“JarFile”的兼容性

Spring Boot Loader力求与现有代码和库保持兼容.  `org.springframework.boot.loader.jar.JarFile` 从 `java.util.jar.JarFile` 延伸，应该作为替代品.  `getURL()` 方法返回一个 `URL` ，它打开一个与 `java.net.JarURLConnection` 兼容的连接，可以与Java的 `URLClassLoader` 一起使用.

## E.3启动可执行的JAR

`org.springframework.boot.loader.Launcher` 类是一个特殊的引导类，用作可执行jar的主要入口点.它是jar文件中的实际 `Main-Class` ，它用于设置适当的 `URLClassLoader` 并最终调用 `main()` 方法.

有三个启动器子类（ `JarLauncher` ， `WarLauncher` 和 `PropertiesLauncher` ）.它们的目的是从嵌套的jar文件或目录中的war文件加载资源（ `.class` 文件等）.（而不是在类路径上显式的那些）.在 `JarLauncher` 和 `WarLauncher` 的情况下，嵌套路径是固定的.  `JarLauncher` 在 `BOOT-INF/lib/` 中查找， `WarLauncher` 在 `WEB-INF/lib/` 和 `WEB-INF/lib-provided/` 中查找.如果您需要更多，可以在这些位置添加额外的jar.默认情况下， `PropertiesLauncher` 在应用程序归档中查找 `BOOT-INF/lib/` ，但您可以通过在 `loader.properties` 中设置名为 `LOADER_PATH` 或 `loader.path` 的环境变量来添加其他位置（这是一个以逗号分隔的目录列表，档案或档案馆内的目录）.

### E.3.1启动器清单

您需要指定适当的 `Launcher` 作为 `META-INF/MANIFEST.MF` 的 `Main-Class` 属性.应在 `Start-Class` 属性中指定要启动的实际类（即包含 `main` 方法的类）.

以下示例显示了可执行jar文件的典型 `MANIFEST.MF` ：

```java
Main-Class: org.springframework.boot.loader.JarLauncher
Start-Class: com.mycompany.project.MyApplication
```

对于war文件，它将如下：

```java
Main-Class: org.springframework.boot.loader.WarLauncher
Start-Class: com.mycompany.project.MyApplication
```

> 您无需在清单文件中指定 `Class-Path` 条目.类路径是从嵌套的jar中推导出来的.

### E.3.2分解档案

某些PaaS实现可能会选择在运行之前解压缩档案.例如，Cloud Foundry就是这样运作的.您可以通过启动相应的启动程序来运行解压缩的存档，如下所示：

```java
$ unzip -q myapp.jar
$ java org.springframework.boot.loader.JarLauncher
```

## E.4 PropertiesLauncher功能

`PropertiesLauncher` 具有一些可以使用外部属性启用的特殊功能（系统属性，环境变量，清单条目或 `loader.properties` ）.下表描述了这些属性：

|主要|目的|
| ---- | ---- |
|  `loader.path`  |以逗号分隔的类路径，例如 `lib,${HOME}/app/lib` .较早的条目优先，如 `javac` 命令行上的常规 `-classpath` . |
|  `loader.home`  |用于解析 `loader.path` 中的相对路径.例如，给定 `loader.path=lib` ，则 `${loader.home}/lib` 是类路径位置（以及该目录中的所有jar文件）.此属性还用于查找 `loader.properties` 文件，如下例所示 `/opt/app` 默认为 `${user.dir}` . |
|  `loader.args`  | main方法的默认参数（以空格分隔）. |
|  `loader.main`  |要启动的主类的名称（例如， `com.app.Application` ）. |
|  `loader.config.name`  |属性文件的名称（例如， `launcher` ）默认为 `loader` . |
|  `loader.config.location`  |属性文件的路径（例如， `classpath:loader.properties` ）.它默认为 `loader.properties` . |
|  `loader.system`  |布尔标志，指示应将所有属性添加到系统属性中默认为 `false` . |

指定为环境变量或清单条目时，应使用以下名称：

|键|清单输入|环境变量|
| ---- | ---- | ---- |
|  `loader.path`  |  `Loader-Path`  |  `LOADER_PATH`  |
|  `loader.home`  |  `Loader-Home`  |  `LOADER_HOME`  |
|  `loader.args`  |  `Loader-Args`  |  `LOADER_ARGS`  |
|  `loader.main`  |  `Start-Class`  |  `LOADER_MAIN`  |
|  `loader.config.location`  |  `Loader-Config-Location`  |  `LOADER_CONFIG_LOCATION`  |
|  `loader.system`  |  `Loader-System`  |  `LOADER_SYSTEM`  |

> Build插件在构建胖jar时自动将 `Main-Class` 属性移动到 `Start-Class` .如果使用它，请使用 `Main-Class` 属性指定要启动的类的名称，并省略 `Start-Class` .

以下规则适用于使用 `PropertiesLauncher` ：

在 `loader.home` 中搜索
-  `loader.properties` ，然后在类路径的根目录中搜索，然后在 `classpath:/BOOT-INF/classes` 中搜索.使用具有该名称的文件的第一个位置.
仅当未指定 `loader.config.location` 时，
-  `loader.home` 是附加属性文件的目录位置（覆盖默认值）.

-  `loader.path` 可以包含目录（以递归方式扫描jar和zip文件），存档路径，存档中扫描jar文件的目录（例如， `dependencies.jar!/lib` ），或通配符模式（用于默认的JVM行为）.存档路径可以相对于 `loader.home` 或文件系统中具有 `jar:file:` 前缀的任何位置.

-  `loader.path` （如果为空）默认为 `BOOT-INF/lib` （表示本地目录或嵌套的目录，如果从存档运行）.因此，当没有提供其他配置时， `PropertiesLauncher` 的行为与 `JarLauncher` 相同.

-  `loader.path` 不能用于配置 `loader.properties` 的位置（用于搜索后者的类路径是启动 `PropertiesLauncher` 时的JVM类路径）.

- Placeholder替换是在使用之前从系统和环境变量以及所有值上的属性文件本身完成的.

- 属性的搜索顺序（在多个位置查看的位置）是环境变量，系统属性， `loader.properties` ，展开的存档清单和存档清单.

## E.5可执行的Jar限制

使用Spring Boot Loader打包应用程序时，您需要考虑以下限制：

- Zip条目压缩：必须使用 `ZipEntry.STORED` 方法保存嵌套jar的 `ZipEntry` .这是必需的，以便我们可以直接寻找嵌套jar中的单个内容.嵌套的jar文件本身的内容仍然可以被压缩，外部jar中的任何其他条目也是如此.

- System classLoader：启动的应用程序在加载类时应使用 `Thread.getContextClassLoader()` （默认情况下，大多数库和框架都这样做）.尝试使用 `ClassLoader.getSystemClassLoader()` 加载嵌套的jar类失败.  `java.util.Logging` 始终使用系统类加载器.因此，您应该考虑使用不同的日志记录实现.

## E.6替代单jar解决方案

如果前面的限制意味着您不能使用Spring Boot Loader，请考虑以下备选方案：

- [Maven Shade Plugin](https://maven.apache.org/plugins/maven-shade-plugin/)

- [JarClassLoader](http://www.jdotsoft.com/JarClassLoader.php)

- [OneJar](http://one-jar.sourceforge.net)


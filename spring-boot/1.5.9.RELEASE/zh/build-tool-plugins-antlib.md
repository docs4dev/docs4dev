## 73. Spring Boot AntLib模块

Spring Boot AntLib模块为Apache Ant提供基本的Spring Boot支持.您可以使用该模块创建可执行jar.要使用该模块，您需要在 `build.xml` 中声明一个额外的 `spring-boot` 命名空间，如以下示例所示：

```xml
<project xmlns:ivy="antlib:org.apache.ivy.ant"
	xmlns:spring-boot="antlib:org.springframework.boot.ant"
	name="myapp" default="build">
	...
</project>
```

您需要记住使用 `-lib` 选项启动Ant，如以下示例所示：

```xml
$ ant -lib <folder containing spring-boot-antlib-2.1.0.RELEASE.jar>
```

> “使用Spring Boot”部分包含一个更完整的[using Apache Ant with spring-boot-antlib](using-boot-build-systems.html#using-boot-ant)示例.

## 73.1 Spring Boot Ant任务

声明 `spring-boot-antlib` 名称空间后，可以使用以下附加任务：

- [Section 73.1.1, “spring-boot:exejar”](build-tool-plugins-antlib.html#spring-boot-ant-exejar)

- [Section 73.2, “spring-boot:findmainclass”](build-tool-plugins-antlib.html#spring-boot-ant-findmainclass)

### 73.1.1 spring-boot：exejar

您可以使用 `exejar` 任务来创建Spring Boot可执行jar.任务支持以下属性：

|属性|说明|必需|
| ---- | ---- | ---- |
|  `destfile`  |要创建的目标jar文件|是|
|  `classes`  | Java类文件的根目录|是|
|  `start-class`  |要运行的主应用程序类|否（默认是找到的第一个声明 `main` 方法的类）

以下嵌套元素可与任务一起使用：

|元素|说明|
| ---- | ---- |
|  `resources`  |描述一组[Resources](https://ant.apache.org/manual/Types/resources.html)的一个或多个[Resource Collections](https://ant.apache.org/manual/Types/resources.html#collection)应该添加到创建的jar文件的内容中. |
|  `lib`  |应该添加到构成应用程序的运行时依赖性类路径的jar库集中的一个或多个[Resource Collections](https://ant.apache.org/manual/Types/resources.html#collection). |

### 73.1.2示例

本节显示了Ant任务的两个示例.

**Specify start-class.** 

```xml
<spring-boot:exejar destfile="target/my-application.jar"
		classes="target/classes" start-class="com.example.MyApplication">
	<resources>
		<fileset dir="src/main/resources" />
	</resources>
	<lib>
		<fileset dir="lib" />
	</lib>
</spring-boot:exejar>
```

**Detect start-class.** 

```xml
<exejar destfile="target/my-application.jar" classes="target/classes">
	<lib>
		<fileset dir="lib" />
	</lib>
</exejar>
```

## 73.2 spring-boot：findmainclass

`findmainclass` 任务在 `exejar` 内部用于查找声明 `main` 的类.如有必要，您还可以直接在构建中使用此任务.支持以下属性：

|属性|说明|必需|
| ---- | ---- | ---- |
|  `classesroot`  | Java类文件的根目录|是（除非指定了 `mainclass` ）|
|  `mainclass`  |可用于短路 `main` 类搜索|否|
|  `property`  |应使用结果|否设置的Ant属性（如果未指定，将记录结果）|

### 73.2.1示例

本节包含使用 `findmainclass` 的三个示例.

**Find and log.** 

```xml
<findmainclass classesroot="target/classes" />
```

**Find and set.** 

```xml
<findmainclass classesroot="target/classes" property="main-class" />
```

**Override and set.** 

```xml
<findmainclass mainclass="com.example.MainClass" property="main-class" />
```


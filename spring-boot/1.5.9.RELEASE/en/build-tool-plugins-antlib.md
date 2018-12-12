## 73. Spring Boot AntLib Module

The Spring Boot AntLib module provides basic Spring Boot support for Apache Ant. You can use the module to create executable jars. To use the module, you need to declare an additional  `spring-boot`  namespace in your  `build.xml` , as shown in the following example:

```xml
<project xmlns:ivy="antlib:org.apache.ivy.ant"
	xmlns:spring-boot="antlib:org.springframework.boot.ant"
	name="myapp" default="build">
	...
</project>
```

You need to remember to start Ant using the  `-lib`  option, as shown in the following example:

```xml
$ ant -lib <folder containing spring-boot-antlib-2.1.0.RELEASE.jar>
```

> The “Using Spring Boot” section includes a more complete example of [using Apache Ant with spring-boot-antlib](using-boot-build-systems.html#using-boot-ant).

## 73.1 Spring Boot Ant Tasks

Once the  `spring-boot-antlib`  namespace has been declared, the following additional tasks are available:

- [Section 73.1.1, “spring-boot:exejar”](build-tool-plugins-antlib.html#spring-boot-ant-exejar)

- [Section 73.2, “spring-boot:findmainclass”](build-tool-plugins-antlib.html#spring-boot-ant-findmainclass)

### 73.1.1 spring-boot:exejar

You can use the  `exejar`  task to create a Spring Boot executable jar. The following attributes are supported by the task:

|Attribute|Description|Required|
|----|----|----|
| `destfile`  |The destination jar file to create |Yes |
| `classes`  |The root directory of Java class files |Yes |
| `start-class`  |The main application class to run |No (the default is the first class found that declares a  `main`  method) |

The following nested elements can be used with the task:

|Element|Description|
|----|----|
| `resources`  |One or more [Resource Collections](https://ant.apache.org/manual/Types/resources.html#collection) describing a set of [Resources](https://ant.apache.org/manual/Types/resources.html) that should be added to the content of the created jar file. |
| `lib`  |One or more [Resource Collections](https://ant.apache.org/manual/Types/resources.html#collection) that should be added to the set of jar libraries that make up the runtime dependency classpath of the application. |

### 73.1.2 Examples

This section shows two examples of Ant tasks.

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

## 73.2 spring-boot:findmainclass

The  `findmainclass`  task is used internally by  `exejar`  to locate a class declaring a  `main` . If necessary, you can also use this task directly in your build. The following attributes are supported:

|Attribute|Description|Required|
|----|----|----|
| `classesroot`  |The root directory of Java class files |Yes (unless  `mainclass`  is specified) |
| `mainclass`  |Can be used to short-circuit the  `main`  class search |No |
| `property`  |The Ant property that should be set with the result |No (result will be logged if unspecified) |

### 73.2.1 Examples

This section contains three examples of using  `findmainclass` .

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


## 124.入门

从命令行构建（并“安装”示例）：

```java
$ ./mvnw clean install
```

（如果你喜欢YOLO添加 `-DskipTests` .）

运行其中一个样本，例如

```java
$ java -jar spring-cloud-function-samples/function-sample/target/*.jar
```

这会运行应用程序并通过HTTP公开其功能，因此您可以将字符串转换为大写，如下所示：

```java
$ curl -H "Content-Type: text/plain" localhost:8080/uppercase -d Hello
HELLO
```

您可以通过用新行分隔多个字符串（ `Flux<String>` ）来转换它们

```java
$ curl -H "Content-Type: text/plain" localhost:8080/uppercase -d 'Hello
> World'
HELLOWORLD
```

（您可以在终端中使用 `QJ` 在文字字符串中插入新行.）

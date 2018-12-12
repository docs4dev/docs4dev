## 131.无服务器平台适配器

除了能够作为独立进程运行之外，Spring Cloud Function应用程序还可以适用于运行现有的无服务器平台之一.在项目中有适用于[AWS Lambda](https://github.com/spring-cloud/spring-cloud-function/tree/master/spring-cloud-function-adapters/spring-cloud-function-adapter-aws)，[Azure](https://github.com/spring-cloud/spring-cloud-function/tree/master/spring-cloud-function-adapters/spring-cloud-function-adapter-azure)和[Apache OpenWhisk](https://github.com/spring-cloud/spring-cloud-function/tree/master/spring-cloud-function-adapters/spring-cloud-function-adapter-openwhisk)的适配器. [Oracle Fn platform](https://github.com/fnproject/fn)有自己的Spring Cloud Function适配器. [Riff](https://projectriff.io)支持Java函数，其本身的[Java Function Invoker](https://github.com/projectriff/java-function-invoker)行为是Spring Cloud Function jar的适配器.

## 131.1 AWS Lambda

[AWS](https://aws.amazon.com/)适配器采用Spring Cloud Function应用程序并将其转换为可在AWS Lambda中运行的表单.

### 131.1.1简介

适配器有几个可以使用的通用请求处理程序.最通用的是 `SpringBootStreamHandler` ，它使用Spring Boot提供的Jackson  `ObjectMapper` 来序列化和反序列化函数中的对象.您还可以扩展 `SpringBootRequestHandler` ，并提供输入和输出类型作为类型参数（使AWS能够检查类并自行执行JSON转换）.

如果您的应用有多个 `@Bean` 类型为 `Function` 等，然后您可以通过配置 `function.name` 来选择要使用的那个（例如，在AWS中为 `FUNCTION_NAME` 环境变量）.这些函数是从Spring Cloud  `FunctionCatalog` 中提取的（首先搜索 `Function` 然后搜索 `Consumer` ，最后搜索 `Supplier` ）.

### 131.1.2关于JAR布局的注释

您不需要在运行时在Lambda中使用Spring Cloud Function Web或Stream适配器，因此您可能需要在创建发送给AWS的JAR之前将其排除. Lambda应用程序必须使用阴影，但Spring Boot独立应用程序不需要，因此您可以使用2个单独的jar运行相同的应用程序（根据示例）.示例应用程序创建了2个jar文件，一个用于在Lambda中部署的 `aws` 分类器，以及一个在运行时包含 `spring-cloud-function-web` 的可执行（瘦）jar. Spring Cloud Function将尝试使用 `Start-Class` 属性从JAR文件清单中为您找到"main class"（如果您使用初始父级，将由Spring Boot工具为您添加）.如果清单中没有 `Start-Class` ，则可以在将功能部署到AWS时使用环境变量 `MAIN_CLASS` .

### 131.1.3上传

在 `spring-cloud-function-samples/function-sample-aws` 下构建示例并将 `-aws`  jar文件上载到Lambda.处理程序可以是 `example.Handler` 或 `org.springframework.cloud.function.adapter.aws.SpringBootStreamHandler` （类的FQN，不是方法引用，尽管Lambda确实接受方法引用）.

```java
./mvnw -U clean package
```

使用AWS命令行工具，它看起来像这样：

```java
aws lambda create-function --function-name Uppercase --role arn:aws:iam::[USERID]:role/service-role/[ROLE] --zip-file fileb://function-sample-aws/target/function-sample-aws-1.0.0.RELEASE-aws.jar --handler org.springframework.cloud.function.adapter.aws.SpringBootStreamHandler --description "Spring Cloud Function Adapter Example" --runtime java8 --region us-east-1 --timeout 30 --memory-size 1024 --publish
```

AWS示例中函数的输入类型是Foo，其中包含一个名为“value”的属性.所以你需要这个来测试它：

```java
{
"value": "test"
}
```

### 131.1.4 Platfom特定功能

#### HTTP和API网关

AWS具有一些特定于平台的数据类型，包括消息的批处理，这比单独处理每个数据更有效.要使用这些类型，您可以编写一个依赖于这些类型的函数.或者您可以依靠Spring从AWS类型中提取数据并将其转换为Spring  `Message` .为此，您告诉AWS该函数属于特定的通用处理程序类型（取决于AWS服务）并提供 `Function<Message<S>,Message<T>>` 类型的bean，其中 `S` 和 `T` 是您的业务数据类型.如果有多个 `Function` 类型的bean，您可能还需要将Spring Boot属性 `function.name` 配置为目标bean的名称（例如，使用 `FUNCTION_NAME` 作为环境变量）.

下面列出了受支持的AWS服务和通用处理程序类型：

|服务| AWS类型|通用处理程序| |
| ---- | ---- | ---- | ---- |
| API网关|  `APIGatewayProxyRequestEvent` ， `APIGatewayProxyResponseEvent`  |  `org.springframework.cloud.function.adapter.aws.SpringBootApiGatewayRequestHandler`  | |
| Kinesis | KinesisEvent | org.springframework.cloud.function.adapter.aws.SpringBootKinesisEventHandler | |

例如，要在API网关后部署，请在AWS命令行中使用 `--handler org.springframework.cloud.function.adapter.aws.SpringBootApiGatewayRequestHandler` （通过UI）并定义 `@Bean` 类型 `Function<Message<Foo>,Message<Bar>>` ，其中 `Foo` 和 `Bar` 是POJO类型（数据将由AWS使用Jackson编组和解组） .

## 131.2 Azure功能

[Azure](https://azure.microsoft.com)适配器引导Spring Cloud Function上下文，并在必要时使用Spring Boot配置将Azure框架中的函数调用引导到用户函数中. Azure Functions具有相当独特但侵入性的编程模型，涉及特定于平台的用户代码中的注释. Spring Cloud Function Azure适配器交换了这些注释的便利性，以实现功能实现的可移植性.您不必使用注释来手动编写一些JSON（至少目前为止），以指导平台调用适配器中的正确方法.

该项目为Azure上的Spring Cloud Function应用程序提供了一个适配器层.您可以使用 `@Bean` 类型的单个 `@Bean` 编写应用程序，如果您正确布置了JAR文件，它将可以在Azure中部署.

适配器具有可以选择使用的通用HTTP请求处理程序.您必须扩展 `AzureSpringBootRequestHandler` ，并提供输入和输出类型作为类型参数（使Azure能够检查类并自行执行JSON转换）.

如果您的应用有多个 `@Bean` 类型 `Function` 等，那么您可以通过配置 `function.name` 来选择要使用的那个.这些函数是从Spring Cloud  `FunctionCatalog` 中提取的.

### 131.2.1关于JAR布局的注释

Azure中的运行时不需要Spring Cloud Function Web，因此在创建部署到Azure的JAR之前，需要将其排除. Azure上的功能应用程序必须加阴影，但Spring Boot独立应用程序不需要，因此您可以使用2个独立的jar运行相同的应用程序（根据此处的示例）.示例应用程序创建着色jar文件，使用 `azure` 分类器在Azure中进行部署.

### 131.2.2 JSON配置

Azure工具需要找到一些JSON配置文件，以告诉它如何部署和集成该函数（例如，哪个Java类用作入口点，以及使用哪个触发器）.可以使用Maven插件为非Spring函数创建这些文件，但是工具在当前形式的适配器中不起作用.有一个例子示例中的 `function.json` 将函数挂钩为HTTPendpoints：

```java
{
"scriptFile" : "../function-sample-azure-1.0.0.RELEASE-azure.jar",
"entryPoint" : "example.FooHandler.execute",
"bindings" : [ {
"type" : "httpTrigger",
"name" : "foo",
"direction" : "in",
"authLevel" : "anonymous",
"methods" : [ "get", "post" ]
}, {
"type" : "http",
"name" : "$return",
"direction" : "out"
} ],
"disabled" : false
}
```

### 131.2.3建造

```java
./mvnw -U clean package
```

### 131.2.4运行样本

您可以在本地运行示例，就像其他Spring Cloud Function示例一样：

和 `curl -H "Content-Type: text/plain" localhost:8080/function -d '{"value": "hello foobar"}'` .

您将需要 `az`  CLI应用程序和一些node.js fu（有关详细信息，请参阅[https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-java-maven](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-java-maven)）.要在Azure运行时上部署该功能：

```java
$ az login
$ mvn azure-functions:deploy
```

在另一个终端试试这个： `curl https://<azure-function-url-from-the-log>/api/uppercase -d '{"value": "hello foobar!"}'` .请确保您使用上述功能的正确URL.或者，您可以在Azure仪表板UI中测试该功能（单击功能名称，转到右侧并单击"Test"，然后单击右下角，"Run"）.

Azure示例中函数的输入类型是Foo，其中包含一个名为“value”的属性.因此，您需要使用以下内容对其进行测试：

```java
{
"value": "foobar"
}
```

## 131.3 Apache Openwhisk

[OpenWhisk](https://openwhisk.apache.org/)适配器采用可执行jar的形式，可以在docker镜像中使用，以部署到Openwhisk.该平台在请求 - 响应模式下工作，侦听特定endpoints上的端口8080，因此适配器是一个简单的Spring MVC应用程序.

### 131.3.1快速入门

实现POF（务必使用 `functions` 包）：

```java
package functions;

import java.util.function.Function;

public class Uppercase implements Function<String, String> {

	public String apply(String input) {
		return input.toUpperCase();
	}
}
```

将其安装到您当地的Maven存储库中：

```java
./mvnw clean install
```

创建一个提供其Maven坐标的 `function.properties` 文件.例如：

```java
dependencies.function: com.example:pof:0.0.1-SNAPSHOT
```

将openwhisk runner JAR复制到工作目录（与属性文件相同的目录）：

```java
cp spring-cloud-function-adapters/spring-cloud-function-adapter-openwhisk/target/spring-cloud-function-adapter-openwhisk-1.0.0.RELEASE.jar runner.jar
```

使用上述属性文件从跑步者JAR的 `--thin.dryrun` 生成m2 repo：

```java
java -jar -Dthin.root=m2 runner.jar --thin.name=function --thin.dryrun
```

使用以下Dockerfile：

```java
FROM openjdk:8-jdk-alpine
VOLUME /tmp
COPY m2 /m2
ADD runner.jar .
ADD function.properties .
ENV JAVA_OPTS=""
ENTRYPOINT [ "java", "-Djava.security.egd=file:/dev/./urandom", "-jar", "runner.jar", "--thin.root=/m2", "--thin.name=function", "--function.name=uppercase"]
EXPOSE 8080
```

请注意，您可以使用Spring Cloud Function应用程序，而不仅仅是带有POF的jar，在这种情况下，您必须更改应用程序在容器中运行的方式，以便它将主类作为源文件获取.例如，您可以更改上面的ENTRYPOINT并添加--spring.main.sources = com.example.SampleApplication.

构建Docker镜像：

```java
docker build -t [username/appname] .
```

推动Docker镜像：

```java
docker push [username/appname]
```

使用OpenWhisk CLI（例如在 `vagrant ssh` 之后）创建操作：

```java
wsk action create example --docker [username/appname]
```

调用动作：

```java
wsk action invoke example --result --param payload foo
{
"result": "FOO"
}
```


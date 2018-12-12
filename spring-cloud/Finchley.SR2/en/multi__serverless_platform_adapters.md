## 131. Serverless Platform Adapters

As well as being able to run as a standalone process, a Spring Cloud Function application can be adapted to run one of the existing serverless platforms. In the project there are adapters for [AWS Lambda](https://github.com/spring-cloud/spring-cloud-function/tree/master/spring-cloud-function-adapters/spring-cloud-function-adapter-aws), [Azure](https://github.com/spring-cloud/spring-cloud-function/tree/master/spring-cloud-function-adapters/spring-cloud-function-adapter-azure), and [Apache OpenWhisk](https://github.com/spring-cloud/spring-cloud-function/tree/master/spring-cloud-function-adapters/spring-cloud-function-adapter-openwhisk). The [Oracle Fn platform](https://github.com/fnproject/fn) has its own Spring Cloud Function adapter. And [Riff](https://projectriff.io) supports Java functions and its [Java Function Invoker](https://github.com/projectriff/java-function-invoker) acts natively is an adapter for Spring Cloud Function jars.

## 131.1 AWS Lambda

The [AWS](https://aws.amazon.com/) adapter takes a Spring Cloud Function app and converts it to a form that can run in AWS Lambda.

### 131.1.1 Introduction

The adapter has a couple of generic request handlers that you can use. The most generic is  `SpringBootStreamHandler` , which uses a Jackson  `ObjectMapper`  provided by Spring Boot to serialize and deserialize the objects in the function. There is also a  `SpringBootRequestHandler`  which you can extend, and provide the input and output types as type parameters (enabling AWS to inspect the class and do the JSON conversions itself).

If your app has more than one  `@Bean`  of type  `Function`  etc. then you can choose the one to use by configuring  `function.name`  (e.g. as  `FUNCTION_NAME`  environment variable in AWS). The functions are extracted from the Spring Cloud  `FunctionCatalog`  (searching first for  `Function`  then  `Consumer`  and finally  `Supplier` ).

### 131.1.2 Notes on JAR Layout

You don’t need the Spring Cloud Function Web or Stream adapter at runtime in Lambda, so you might need to exclude those before you create the JAR you send to AWS. A Lambda application has to be shaded, but a Spring Boot standalone application does not, so you can run the same app using 2 separate jars (as per the sample). The sample app creates 2 jar files, one with an  `aws`  classifier for deploying in Lambda, and one executable (thin) jar that includes  `spring-cloud-function-web`  at runtime. Spring Cloud Function will try and locate a "main class" for you from the JAR file manifest, using the  `Start-Class`  attribute (which will be added for you by the Spring Boot tooling if you use the starter parent). If there is no  `Start-Class`  in your manifest you can use an environment variable  `MAIN_CLASS`  when you deploy the function to AWS.

### 131.1.3 Upload

Build the sample under  `spring-cloud-function-samples/function-sample-aws`  and upload the  `-aws`  jar file to Lambda. The handler can be  `example.Handler`  or  `org.springframework.cloud.function.adapter.aws.SpringBootStreamHandler`  (FQN of the class, not a method reference, although Lambda does accept method references).

```java
./mvnw -U clean package
```

Using the AWS command line tools it looks like this:

```java
aws lambda create-function --function-name Uppercase --role arn:aws:iam::[USERID]:role/service-role/[ROLE] --zip-file fileb://function-sample-aws/target/function-sample-aws-1.0.0.RELEASE-aws.jar --handler org.springframework.cloud.function.adapter.aws.SpringBootStreamHandler --description "Spring Cloud Function Adapter Example" --runtime java8 --region us-east-1 --timeout 30 --memory-size 1024 --publish
```

The input type for the function in the AWS sample is a Foo with a single property called "value". So you would need this to test it:

```java
{
"value": "test"
}
```

### 131.1.4 Platfom Specific Features

#### HTTP and API Gateway

AWS has some platform-specific data types, including batching of messages, which is much more efficient than processing each one individually. To make use of these types you can write a function that depends on those types. Or you can rely on Spring to extract the data from the AWS types and convert it to a Spring  `Message` . To do this you tell AWS that the function is of a specific generic handler type (depending on the AWS service) and provide a bean of type  `Function<Message<S>,Message<T>>` , where  `S`  and  `T`  are your business data types. If there is more than one bean of type  `Function`  you may also need to configure the Spring Boot property  `function.name`  to be the name of the target bean (e.g. use  `FUNCTION_NAME`  as an environment variable).

The supported AWS services and generic handler types are listed below:

|Service|AWS Types|Generic Handler| |
|----|----|----|----|
|API Gateway | `APIGatewayProxyRequestEvent` ,  `APIGatewayProxyResponseEvent`  | `org.springframework.cloud.function.adapter.aws.SpringBootApiGatewayRequestHandler`  | |
|Kinesis |KinesisEvent |org.springframework.cloud.function.adapter.aws.SpringBootKinesisEventHandler | |

For example, to deploy behind an API Gateway, use  `--handler org.springframework.cloud.function.adapter.aws.SpringBootApiGatewayRequestHandler`  in your AWS command line (in via the UI) and define a  `@Bean`  of type  `Function<Message<Foo>,Message<Bar>>`  where  `Foo`  and  `Bar`  are POJO types (the data will be marshalled and unmarshalled by AWS using Jackson).

## 131.2 Azure Functions

The [Azure](https://azure.microsoft.com) adapter bootstraps a Spring Cloud Function context and channels function calls from the Azure framework into the user functions, using Spring Boot configuration where necessary. Azure Functions has quite a unique, but invasive programming model, involving annotations in user code that are specific to the platform. The Spring Cloud Function Azure adapter trades the convenience of these annotations for portability of the function implementations. Instead of using the annotations you have to write some JSON by hand (at least for now) to guide the platform to call the right methods in the adapter.

This project provides an adapter layer for a Spring Cloud Function application onto Azure. You can write an app with a single  `@Bean`  of type  `Function`  and it will be deployable in Azure if you get the JAR file laid out right.

The adapter has a generic HTTP request handler that you can use optionally. There is a  `AzureSpringBootRequestHandler`  which you must extend, and provide the input and output types as type parameters (enabling Azure to inspect the class and do the JSON conversions itself).

If your app has more than one  `@Bean`  of type  `Function`  etc. then you can choose the one to use by configuring  `function.name` . The functions are extracted from the Spring Cloud  `FunctionCatalog` .

### 131.2.1 Notes on JAR Layout

You don’t need the Spring Cloud Function Web at runtime in Azure, so you need to exclude this before you create the JAR you deploy to Azure. A function application on Azure has to be shaded, but a Spring Boot standalone application does not, so you can run the same app using 2 separate jars (as per the sample here). The sample app creates the shaded jar file, with an  `azure`  classifier for deploying in Azure.

### 131.2.2 JSON Configuration

The Azure tooling needs to find some JSON configuration files to tell it how to deploy and integrate the function (e.g. which Java class to use as the entry point, and which triggers to use). Those files can be created with the Maven plugin for a non-Spring function, but the tooling doesn’t work yet with the adapter in its current form. There is an example  `function.json`  in the sample which hooks the function up as an HTTP endpoint:

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

### 131.2.3 Build

```java
./mvnw -U clean package
```

### 131.2.4 Running the sample

You can run the sample locally, just like the other Spring Cloud Function samples:

and  `curl -H "Content-Type: text/plain" localhost:8080/function -d '{"value": "hello foobar"}'` .

You will need the  `az`  CLI app and some node.js fu (see [https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-java-maven](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-first-java-maven) for more detail). To deploy the function on Azure runtime:

```java
$ az login
$ mvn azure-functions:deploy
```

On another terminal try this:  `curl https://<azure-function-url-from-the-log>/api/uppercase -d '{"value": "hello foobar!"}'` . Please ensure that you use the right URL for the function above. Alternatively you can test the function in the Azure Dashboard UI (click on the function name, go to the right hand side and click "Test" and to the bottom right, "Run").

The input type for the function in the Azure sample is a Foo with a single property called "value". So you need this to test it with something like below:

```java
{
"value": "foobar"
}
```

## 131.3 Apache Openwhisk

The [OpenWhisk](https://openwhisk.apache.org/) adapter is in the form of an executable jar that can be used in a a docker image to be deployed to Openwhisk. The platform works in request-response mode, listening on port 8080 on a specific endpoint, so the adapter is a simple Spring MVC application.

### 131.3.1 Quick Start

Implement a POF (be sure to use the  `functions`  package):

```java
package functions;

import java.util.function.Function;

public class Uppercase implements Function<String, String> {

	public String apply(String input) {
		return input.toUpperCase();
	}
}
```

Install it into your local Maven repository:

```java
./mvnw clean install
```

Create a  `function.properties`  file that provides its Maven coordinates. For example:

```java
dependencies.function: com.example:pof:0.0.1-SNAPSHOT
```

Copy the openwhisk runner JAR to the working directory (same directory as the properties file):

```java
cp spring-cloud-function-adapters/spring-cloud-function-adapter-openwhisk/target/spring-cloud-function-adapter-openwhisk-1.0.0.RELEASE.jar runner.jar
```

Generate a m2 repo from the  `--thin.dryrun`  of the runner JAR with the above properties file:

```java
java -jar -Dthin.root=m2 runner.jar --thin.name=function --thin.dryrun
```

Use the following Dockerfile:

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

Note you could use a Spring Cloud Function app, instead of just a jar with a POF in it, in which case you would have to change the way the app runs in the container so that it picks up the main class as a source file. For example, you could change the ENTRYPOINT above and add --spring.main.sources=com.example.SampleApplication.

Build the Docker image:

```java
docker build -t [username/appname] .
```

Push the Docker image:

```java
docker push [username/appname]
```

Use the OpenWhisk CLI (e.g. after  `vagrant ssh` ) to create the action:

```java
wsk action create example --docker [username/appname]
```

Invoke the action:

```java
wsk action invoke example --result --param payload foo
{
"result": "FOO"
}
```


## 124. Getting Started

Build from the command line (and "install" the samples):

```java
$ ./mvnw clean install
```

(If you like to YOLO add  `-DskipTests` .)

Run one of the samples, e.g.

```java
$ java -jar spring-cloud-function-samples/function-sample/target/*.jar
```

This runs the app and exposes its functions over HTTP, so you can convert a string to uppercase, like this:

```java
$ curl -H "Content-Type: text/plain" localhost:8080/uppercase -d Hello
HELLO
```

You can convert multiple strings (a  `Flux<String>` ) by separating them with new lines

```java
$ curl -H "Content-Type: text/plain" localhost:8080/uppercase -d 'Hello
> World'
HELLOWORLD
```

(You can use  `QJ`  in a terminal to insert a new line in a literal string like that.)

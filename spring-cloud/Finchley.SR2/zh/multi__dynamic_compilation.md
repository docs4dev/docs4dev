## 130.动态编译

有一个示例应用程序使用函数编译器从配置属性创建函数.香草"function-sample"也有这个功能.还有一些脚本可以运行以查看运行时发生的编译.要运行这些示例，请转到 `scripts` 目录：

```java
cd scripts
```

此外，在本地启动RabbitMQ服务器（例如，执行 `rabbitmq-server` ）.

启动Function Registry Service：

```java
./function-registry.sh
```

注册功能：

```java
./registerFunction.sh -n uppercase -f "f->f.map(s->s.toString().toUpperCase())"
```

使用该功能运行REST微服务：

```java
./web.sh -f uppercase -p 9000
curl -H "Content-Type: text/plain" -H "Accept: text/plain" localhost:9000/uppercase -d foo
```

注册供应商：

```java
./registerSupplier.sh -n words -f "()->Flux.just(\"foo\",\"bar\")"
```

使用该供应商运行REST微服务：

```java
./web.sh -s words -p 9001
curl -H "Accept: application/json" localhost:9001/words
```

注册消费者：

```java
./registerConsumer.sh -n print -t String -f "System.out::println"
```

使用该Consumer运行REST微服务：

```java
./web.sh -c print -p 9002
curl -X POST -H "Content-Type: text/plain" -d foo localhost:9002/print
```

运行流处理微服务：

首先注册流媒体字供应商：

```java
./registerSupplier.sh -n wordstream -f "()->Flux.interval(Duration.ofMillis(1000)).map(i->\"message-\"+i)"
```

然后启动源（供应商），处理器（功能）和接收（消费者）应用程序（按相反顺序）：

```java
./stream.sh -p 9103 -i uppercaseWords -c print
./stream.sh -p 9102 -i words -f uppercase -o uppercaseWords
./stream.sh -p 9101 -s wordstream -o words
```

输出将显示在接收器应用程序的控制台中（每秒一条消息，转换为大写）：

```java
MESSAGE-0
MESSAGE-1
MESSAGE-2
MESSAGE-3
MESSAGE-4
MESSAGE-5
MESSAGE-6
MESSAGE-7
MESSAGE-8
MESSAGE-9
...
```

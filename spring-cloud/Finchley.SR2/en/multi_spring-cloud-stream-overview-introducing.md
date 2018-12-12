## 25. Introducing Spring Cloud Stream

Spring Cloud Stream is a framework for building message-driven microservice applications. Spring Cloud Stream builds upon Spring Boot to create standalone, production-grade Spring applications and uses Spring Integration to provide connectivity to message brokers. It provides opinionated configuration of middleware from several vendors, introducing the concepts of persistent publish-subscribe semantics, consumer groups, and partitions.

You can add the  `@EnableBinding`  annotation to your application to get immediate connectivity to a message broker, and you can add  `@StreamListener`  to a method to cause it to receive events for stream processing. The following example shows a sink application that receives external messages:

```java
@SpringBootApplication
@EnableBinding(Sink.class)
public class VoteRecordingSinkApplication {

public static void main(String[] args) {
SpringApplication.run(VoteRecordingSinkApplication.class, args);
}

@StreamListener(Sink.INPUT)
public void processVote(Vote vote) {
votingService.recordVote(vote);
}
}
```

The  `@EnableBinding`  annotation takes one or more interfaces as parameters (in this case, the parameter is a single  `Sink`  interface). An interface declares input and output channels. Spring Cloud Stream provides the  `Source` ,  `Sink` , and  `Processor`  interfaces. You can also define your own interfaces.

The following listing shows the definition of the  `Sink`  interface:

```java
public interface Sink {
String INPUT = "input";

@Input(Sink.INPUT)
SubscribableChannel input();
}
```

The  `@Input`  annotation identifies an input channel, through which received messages enter the application. The  `@Output`  annotation identifies an output channel, through which published messages leave the application. The  `@Input`  and  `@Output`  annotations can take a channel name as a parameter. If a name is not provided, the name of the annotated method is used.

Spring Cloud Stream creates an implementation of the interface for you. You can use this in the application by autowiring it, as shown in the following example (from a test case):

```java
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = VoteRecordingSinkApplication.class)
@WebAppConfiguration
@DirtiesContext
public class StreamApplicationTests {

@Autowired
private Sink sink;

@Test
public void contextLoads() {
assertNotNull(this.sink.input());
}
}
```

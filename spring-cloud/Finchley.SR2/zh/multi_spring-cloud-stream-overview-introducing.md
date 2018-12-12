## 25.介绍Spring Cloud Stream

Spring Cloud Stream是一个用于构建消息驱动的微服务应用程序的框架. Spring Cloud Stream构建于Spring Boot之上，用于创建独立的生产环境级Spring应用程序，并使用Spring Integration提供与消息代理的连接.它提供了来自多个供应商的中间件的固定配置，介绍了持久性发布 - 订阅语义，使用者组和分区的概念.

您可以将 `@EnableBinding` 注释添加到应用程序以立即连接到消息代理，并且可以将 `@StreamListener` 添加到方法以使其接收流处理事件.以下示例显示了接收外部消息的接收器应用程序：

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

`@EnableBinding` 注释将一个或多个接口作为参数（在本例中，参数是单个 `Sink` 接口）.接口声明输入和输出通道. Spring Cloud Stream提供 `Source` ， `Sink` 和 `Processor` 接口.您还可以定义自己的界面.

以下清单显示了 `Sink` 接口的定义：

```java
public interface Sink {
String INPUT = "input";

@Input(Sink.INPUT)
SubscribableChannel input();
}
```

`@Input` 注释标识输入通道，接收的消息通过该通道进入应用程序.  `@Output` 注释标识输出通道，已发布的消息通过该通道离开应用程序.  `@Input` 和 `@Output` 注释可以将通道名称作为参数.如果未提供名称，则使用带注释的方法的名称.

Spring Cloud Stream为您创建了一个接口实现.您可以通过自动装配在应用程序中使用它，如以下示例所示（来自测试用例）：

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

## 1. Spring批量集成

XML Java


### 1.1. Spring Batch Integration简介

许多Spring Batch用户可能会遇到超出Spring Batch范围的需求，但可以使用Spring Integration高效，简洁地实现这些需求 . 相反，Spring Integration用户可能会遇到Spring Batch需求，并且需要一种有效集成两个框架的方法 . 在这种情况下，出现了几种模式和用例，Spring Batch Integration解决了这些需求 . 

Spring Batch和Spring Integration之间的界限并不总是很清楚，但有两条建议可以提供帮助：考虑粒度，并应用常见模式 . 本参考手册部分介绍了其中一些常见模式 . 

向批处理流程添加消息传递可实现操作的自动化以及关键问题的分离和策略 . 例如，消息可能触发要执行的作业，然后可以以各种方式公开消息的发送 . 或者，当作业完成或失败时，该事件可能会触发要发送的消息，并且这些消息的使用者可能具有与应用程序本身无关的操作问题 . 消息传递也可以嵌入到作业中（例如，读取或写入项目以通过通道进行处理） . 远程分区和远程分块提供了分配工作负载的方法工作人员 . 

本节介绍以下主要概念：


- 
[Namespace Support](#namespace-support)


- 
[Launching Batch Jobs through Messages](#launching-batch-jobs-through-messages)


- 
[Providing Feedback with Informational Messages](#providing-feedback-with-informational-messages)


- 
[Asynchronous Processors](#asynchronous-processors)


- 
[Externalizing Batch Process Execution](#externalizing-batch-process-execution)


#### 1.1.1.命名空间支持

自Spring Batch Integration 1.3以来，添加了专用的XML Namespace支持，旨在提供更简单的配置体验 . 要激活命名空间，请将以下命名空间声明添加到Spring XML Application Context文件中：


```xml
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:batch-int="http://www.springframework.org/schema/batch-integration"
  xsi:schemaLocation="
    http://www.springframework.org/schema/batch-integration
    http://www.springframework.org/schema/batch-integration/spring-batch-integration.xsd">

    ...

</beans>
```


用于Spring Batch Integration的完全配置的Spring XML Application Context文件可能如下所示：


```xml
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xmlns:int="http://www.springframework.org/schema/integration"
  xmlns:batch="http://www.springframework.org/schema/batch"
  xmlns:batch-int="http://www.springframework.org/schema/batch-integration"
  xsi:schemaLocation="
    http://www.springframework.org/schema/batch-integration
    http://www.springframework.org/schema/batch-integration/spring-batch-integration.xsd
    http://www.springframework.org/schema/batch
    http://www.springframework.org/schema/batch/spring-batch.xsd
    http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd
    http://www.springframework.org/schema/integration
    http://www.springframework.org/schema/integration/spring-integration.xsd">

    ...

</beans>
```


也允许在引用的XSD文件中附加版本号，但是，由于无版本声明始终使用最新的模式，因此我们通常不建议将版本号附加到XSD名称 . 在更新Spring Batch Integration依赖项时添加版本号可能会产生问题，因为它们可能需要更新版本的XML架构 . 


#### 1.1.2.通过消息启动批处理作业

使用核心Spring Batch API启动批处理作业时，基本上有两个选项：


- 
从命令行，使用 `CommandLineJobRunner` 


- 
以编程方式，使用 `JobOperator.start()` 或 `JobLauncher.run()` 

例如，您可能希望在使用shell脚本调用批处理作业时使用 `CommandLineJobRunner`  . 或者，您可以直接使用 `JobOperator` （例如，将Spring Batch用作Web应用程序的一部分时） . 但是，更复杂的用例呢？也许您需要轮询远程（S）FTP服务器以检索批处理作业的数据，或者您的应用程序必须同时支持多个不同的数据源 . 例如，您不仅可以从Web接收数据文件，还可以从FTP和其他来源接收数据文件 . 在调用Spring Batch之前，可能需要对输入文件进行额外的转换 . 

因此，使用Spring Integration及其众多适配器执行批处理作业会更强大 . 例如，您可以使用文件入站通道适配器来监视文件系统中的目录，并在输入文件到达后立即启动批处理作业 . 此外，您可以创建使用多个不同适配器的Spring Integration流，仅使用配置即可同时从多个源轻松获取批处理作业的数据 . 使用Spring Integration实现所有这些场景很简单，因为它允许 `JobLauncher` 的解耦，事件驱动执行 . 

Spring Batch Integration提供了可用于启动批处理作业的 `JobLaunchingMessageHandler` 类 .   `JobLaunchingMessageHandler` 的输入由Spring Integration消息提供，该消息的有效负载为 `JobLaunchRequest`  . 这个类是 `Job` 的包装器，它需要在启动批处理作业所需的 `JobParameters` 周围启动 . 

下图说明了启动批处理作业的典型Spring Integration消息流 .  [EIP (Enterprise Integration Patterns) website](http://www.eaipatterns.com/toc.html)提供了消息传递图标及其描述的完整概述 . 


![Launch Batch Job](https://www.docs4dev.com/images/ca0a4780-9bf5-4b20-ba96-14ac3f161f68.png)


图1.启动批处理作业


##### 将文件转换为JobLaunchRequest


```java
package io.spring.sbi;

import org.springframework.batch.core.Job;
import org.springframework.batch.core.JobParametersBuilder;
import org.springframework.batch.integration.launch.JobLaunchRequest;
import org.springframework.integration.annotation.Transformer;
import org.springframework.messaging.Message;

import java.io.File;

public class FileMessageToJobRequest {
    private Job job;
    private String fileParameterName;

    public void setFileParameterName(String fileParameterName) {
        this.fileParameterName = fileParameterName;
    }

    public void setJob(Job job) {
        this.job = job;
    }

    @Transformer
    public JobLaunchRequest toRequest(Message<File> message) {
        JobParametersBuilder jobParametersBuilder =
            new JobParametersBuilder();

        jobParametersBuilder.addString(fileParameterName,
            message.getPayload().getAbsolutePath());

        return new JobLaunchRequest(job, jobParametersBuilder.toJobParameters());
    }
}
```



##### JobExecution响应

执行批处理作业时，将返回 `JobExecution` 实例 . 此实例可用于确定执行的状态 . 如果能够成功创建 `JobExecution` ，则无论实际执行是否成功，都会始终返回 . 

返回 `JobExecution` 实例的确切行为取决于提供的 `TaskExecutor`  . 如果使用 `synchronous` （单线程） `TaskExecutor` 实现，则只返回 `JobExecution` 响应 `after` 作业完成 . 使用 `asynchronous`   `TaskExecutor` 时，会立即返回 `JobExecution` 实例 . 然后，用户可以使用 `id`   `JobExecution` 实例（使用 `JobExecution.getJobId()` ）并使用 `JobExplorer` 查询 `JobRepository` 以获取作业的更新状态 . 有关更多信息，请参阅[Querying the Repository](job.html#queryingRepository)上的Spring Batch参考文档 . 


##### Spring批量集成配置

以下配置创建文件 `inbound-channel-adapter` 以侦听提供的目录中的CSV文件，将它们移交给我们的变换器（ `FileMessageToJobRequest` ），通过作业启动网关启动作业，然后使用 `logging-channel-adapter` 记录 `JobExecution` 的输出 . 

XML配置


```xml
<int:channel id="inboundFileChannel"/>
<int:channel id="outboundJobRequestChannel"/>
<int:channel id="jobLaunchReplyChannel"/>

<int-file:inbound-channel-adapter id="filePoller"
    channel="inboundFileChannel"
    directory="file:/tmp/myfiles/"
    filename-pattern="*.csv">
  <int:poller fixed-rate="1000"/>
</int-file:inbound-channel-adapter>

<int:transformer input-channel="inboundFileChannel"
    output-channel="outboundJobRequestChannel">
  <bean class="io.spring.sbi.FileMessageToJobRequest">
    <property name="job" ref="personJob"/>
    <property name="fileParameterName" value="input.file.name"/>
  </bean>
</int:transformer>

<batch-int:job-launching-gateway request-channel="outboundJobRequestChannel"
    reply-channel="jobLaunchReplyChannel"/>

<int:logging-channel-adapter channel="jobLaunchReplyChannel"/>
```


Java配置


```java
@Bean
public FileMessageToJobRequest fileMessageToJobRequest() {
    FileMessageToJobRequest fileMessageToJobRequest = new FileMessageToJobRequest();
    fileMessageToJobRequest.setFileParameterName("input.file.name");
    fileMessageToJobRequest.setJob(personJob());
    return fileMessageToJobRequest;
}

@Bean
public JobLaunchingGateway jobLaunchingGateway() {
    SimpleJobLauncher simpleJobLauncher = new SimpleJobLauncher();
    simpleJobLauncher.setJobRepository(jobRepository);
    simpleJobLauncher.setTaskExecutor(new SyncTaskExecutor());
    JobLaunchingGateway jobLaunchingGateway = new JobLaunchingGateway(simpleJobLauncher);

    return jobLaunchingGateway;
}

@Bean
public IntegrationFlow integrationFlow(JobLaunchingGateway jobLaunchingGateway) {
    return IntegrationFlows.from(Files.inboundAdapter(new File("/tmp/myfiles")).
                    filter(new SimplePatternFileListFilter("*.csv")),
            c -> c.poller(Pollers.fixedRate(1000).maxMessagesPerPoll(1))).
            handle(fileMessageToJobRequest()).
            handle(jobLaunchingGateway).
            log(LoggingHandler.Level.WARN, "headers.id + ': ' + payload").
            get();
}
```



##### Example ItemReader配置

现在我们正在轮询文件并启动作业，我们需要配置Spring Batch  `ItemReader` （例如）以使用在名为"input.file.name"的作业参数定义的位置找到的文件，如以下bean配置所示：

XML配置


```xml
<bean id="itemReader" class="org.springframework.batch.item.file.FlatFileItemReader"
    scope="step">
  <property name="resource" value="file://#{jobParameters['input.file.name']}"/>
    ...
</bean>
```


Java配置


```java
@Bean
@StepScope
public ItemReader sampleReader(@Value("#{jobParameters[input.file.name]}") String resource) {
...
    FlatFileItemReader flatFileItemReader = new FlatFileItemReader();
    flatFileItemReader.setResource(new FileSystemResource(resource));
...
    return flatFileItemReader;
}
```


前面示例中的主要兴趣点是将 `#{jobParameters['input.file.name']}` 的值作为Resource属性值注入，并将 `ItemReader` bean设置为具有Step作用域 . 将bean设置为具有步骤作用域利用了后期绑定支持，允许访问 `jobParameters` 变量 . 


### 1.2.作业启动网关的可用属性

作业启动网关具有以下属性可以设置控制工作：


- 
 `id` ：标识基础Spring bean定义，它是以下任一个的实例：


- 
 `EventDrivenConsumer` 


- 
 `PollingConsumer` （确切的实现取决于组件的输入通道是 `SubscribableChannel` 还是 `PollableChannel`  . ）


- 
 `auto-startup` ：布尔标志，指示 endpoints 应在启动时自动启动 . 默认值为true . 


- 
 `request-channel` ：此 endpoints 的输入 `MessageChannel`  . 


- 
 `reply-channel` ： `MessageChannel` 将生成的 `JobExecution` 有效负载发送到该地址 . 


- 
 `reply-timeout` ：允许您指定此网关在抛出异常之前等待回复消息成功发送到回复通道的时间（以毫秒为单位） . 此属性仅在通道可能阻塞时应用（例如，使用当前已满的有界队列通道时） . 另外，请记住，当发送到 `DirectChannel` 时，调用发生在发送方的线程中 . 因此，发送操作的失败可能是由更下游的其他组件引起的 .   `reply-timeout` 属性映射到基础 `MessagingTemplate` 实例的 `sendTimeout` 属性 . 如果未指定，则属性默认为<emphasis> -1 </ emphasis>，这意味着，默认情况下， `Gateway` 无限期等待 . 


- 
 `job-launcher` ：可选 . 接受自定义 `JobLauncher`  bean引用 . 如果未指定，则适配器将重新使用在 `id`   `jobLauncher` 下注册的实例 . 如果不存在默认实例，则抛出异常 . 


- 
 `order` ：指定此 endpoints 作为订户连接到 `SubscribableChannel` 时的调用顺序 . 


### 1.3.子元素

当 `Gateway` 从 `PollableChannel` 接收消息时，您必须提供全局默认 `Poller` 或向 `Job Launching Gateway` 提供 `Poller` 子元素，如以下示例所示：

XML配置


```xml
<batch-int:job-launching-gateway request-channel="queueChannel"
    reply-channel="replyChannel" job-launcher="jobLauncher">
  <int:poller fixed-rate="1000">
</batch-int:job-launching-gateway>
```


Java配置


```java
@Bean
@ServiceActivator(inputChannel = "queueChannel", poller = @Poller(fixedRate="1000"))
public JobLaunchingGateway sampleJobLaunchingGateway() {
    JobLaunchingGateway jobLaunchingGateway = new JobLaunchingGateway(jobLauncher());
    jobLaunchingGateway.setOutputChannel(replyChannel());
    return jobLaunchingGateway;
}
```



#### 1.3.1.提供信息性消息的反馈

由于Spring Batch作业可以运行很长时间，因此提供进度信息通常很关键 . 例如，如果批处理作业的某些或所有部分发生故障，则可能希望通知利益相关者 .  Spring Batch通过以下方式为收集的信息提供支持：


- 
积极的民意调查


- 
事件驱动的听众

异步启动Spring Batch作业时（例如，使用 `Job Launching Gateway` ），将返回 `JobExecution` 实例 . 因此， `JobExecution.getJobId()` 可用于通过使用 `JobExplorer` 从 `JobRepository` 检索 `JobExecution` 的更新实例来连续轮询状态更新 . 但是，这被认为是次优的，应该首选事件驱动的方法 . 

因此，Spring Batch提供了一些监听器，包括三个最常用的监听器：


- 
StepListener


- 
ChunkListener


- 
JobExecutionListener

在下图所示的示例中，Spring Batch作业已配置 `StepExecutionListener`  . 因此，Spring Integration接收并处理事件之前或之后的任何步骤 . 例如，可以使用 `Router` 检查收到的 `StepExecution`  . 根据检查结果，可能会发生各种事情（例如将消息路由到邮件出站通道适配器），以便根据某些条件发送电子邮件通知 . 


![Handling Informational Messages](https://www.docs4dev.com/images/5ce6e8af-28b2-4c65-a36a-42d61e236868.png)


图2.处理信息性消息

以下两部分示例显示了如何将侦听器配置为向 `Gateway` 事件发送消息并将其输出记录到 `logging-channel-adapter`  . 

首先，创建通知集成bean：

XML配置


```xml
<int:channel id="stepExecutionsChannel"/>

<int:gateway id="notificationExecutionsListener"
    service-interface="org.springframework.batch.core.StepExecutionListener"
    default-request-channel="stepExecutionsChannel"/>

<int:logging-channel-adapter channel="stepExecutionsChannel"/>
```


Java配置


```java
@Bean
@ServiceActivator(inputChannel = "stepExecutionsChannel")
public LoggingHandler loggingHandler() {
    LoggingHandler adapter = new LoggingHandler(LoggingHandler.Level.WARN);
    adapter.setLoggerName("TEST_LOGGER");
    adapter.setLogExpressionString("headers.id + ': ' + payload");
    return adapter;
}

@MessagingGateway(name = "notificationExecutionsListener", defaultRequestChannel = "stepExecutionsChannel")
public interface NotificationExecutionListener extends StepExecutionListener {}
```



> 您需要将 `@IntegrationComponentScan` 注释添加到配置中 . 

其次，修改您的作业以添加步骤级侦听器：

XML配置


```xml
<job id="importPayments">
    <step id="step1">
        <tasklet ../>
            <chunk ../>
            <listeners>
                <listener ref="notificationExecutionsListener"/>
            </listeners>
        </tasklet>
        ...
    </step>
</job>
```


Java配置


```java
public Job importPaymentsJob() {
    return jobBuilderFactory.get("importPayments")
        .start(stepBuilderFactory.get("step1")
                .chunk(200)
                .listener(notificationExecutionsListener())
                ...
}
```



#### 1.3.2.异步处理器

异步处理器可帮助您扩展项目的处理 . 在异步处理器用例中， `AsyncItemProcessor` 用作调度程序，为新线程上的项执行 `ItemProcessor` 的逻辑 . 项目完成后， `Future` 将传递给 `AsynchItemWriter` 进行写入 . 

因此，您可以通过使用异步项处理来提高性能，基本上允许您实现fork-join场景 .   `AsyncItemWriter` 收集结果并在所有结果可用后立即写回块 . 

以下示例显示如何配置 `AsyncItemProcessor` ：

XML配置


```xml
<bean id="processor"
    class="org.springframework.batch.integration.async.AsyncItemProcessor">
  <property name="delegate">
    <bean class="your.ItemProcessor"/>
  </property>
  <property name="taskExecutor">
    <bean class="org.springframework.core.task.SimpleAsyncTaskExecutor"/>
  </property>
</bean>
```


Java配置


```java
@Bean
public AsyncItemProcessor processor(ItemProcessor itemProcessor, TaskExecutor taskExecutor) {
    AsyncItemProcessor asyncItemProcessor = new AsyncItemProcessor();
    asyncItemProcessor.setTaskExecutor(taskExecutor);
    asyncItemProcessor.setDelegate(itemProcessor);
    return asyncItemProcessor;
}
```


 `delegate` 属性引用您的 `ItemProcessor`  bean， `taskExecutor` 属性引用您选择的 `TaskExecutor`  . 

以下示例显示如何配置 `AsyncItemWriter` ：

XML配置


```xml
<bean id="itemWriter"
    class="org.springframework.batch.integration.async.AsyncItemWriter">
  <property name="delegate">
    <bean id="itemWriter" class="your.ItemWriter"/>
  </property>
</bean>
```


Java配置


```java
@Bean
public AsyncItemWriter writer(ItemWriter itemWriter) {
    AsyncItemWriter asyncItemWriter = new AsyncItemWriter();
    asyncItemWriter.setDelegate(itemWriter);
    return asyncItemWriter;
}
```


同样， `delegate` 属性实际上是对 `ItemWriter`  bean的引用 . 


#### 1.3.3.外部化批处理执行

到目前为止讨论的集成方法建议使用Spring Integration将Spring Batch包装成外壳的用例 . 但是，Spring Batch也可以在内部使用Spring Integration . 使用这种方法，Spring Batch用户可以将项目或甚至块的处理委托给外部进程 . 这允许您卸载复杂的处理 .  Spring Batch Integration为以下方面提供专门支持：


- 
远程分块


- 
远程分区


##### Remote Chunking


![Remote Chunking](https://www.docs4dev.com/images/8df07d08-db5c-4c1a-99d8-1edfb0ea4f49.png)


图3.远程分块

更进一步，还可以使用 `ChunkMessageChannelItemWriter` （由Spring Batch Integration提供）将块处理外部化，从而将项目发送出去并收集结果 . 一旦发送，Spring Batch将继续读取和分组项目，而无需等待结果 . 相反， `ChunkMessageChannelItemWriter` 负责收集结果并将它们集成回Spring批处理过程 . 

使用Spring Integration，您可以完全控制进程的并发性（例如，使用 `QueueChannel` 而不是 `DirectChannel` ） . 此外，通过依赖Spring Integration丰富的通道适配器集合（例如JMS和AMQP），您可以将批处理作业的块分发到外部系统进行处理 . 

具有远程分块步骤的简单作业可能具有与以下类似的配置：

XML配置


```xml
<job id="personJob">
  <step id="step1">
    <tasklet>
      <chunk reader="itemReader" writer="itemWriter" commit-interval="200"/>
    </tasklet>
    ...
  </step>
</job>
```


Java配置


```java
public Job chunkJob() {
     return jobBuilderFactory.get("personJob")
             .start(stepBuilderFactory.get("step1")
                     .<Person, Person>chunk(200)
                     .reader(itemReader())
                     .writer(itemWriter())
                     .build())
             .build();
 }
```


 `ItemReader` 引用指向要用于读取主数据上的数据的bean .   `ItemWriter` 引用指向一个特殊的 `ItemWriter` （称为 `ChunkMessageChannelItemWriter` ），如上所述 . 处理器（如果有）不在主配置中，因为它在worker上配置 . 以下配置提供基本主设置 . 在实现用例时，应检查任何其他组件属性，例如节流限制等 . 

XML配置


```xml
<bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
  <property name="brokerURL" value="tcp://localhost:61616"/>
</bean>

<int-jms:outbound-channel-adapter id="jmsRequests" destination-name="requests"/>

<bean id="messagingTemplate"
    class="org.springframework.integration.core.MessagingTemplate">
  <property name="defaultChannel" ref="requests"/>
  <property name="receiveTimeout" value="2000"/>
</bean>

<bean id="itemWriter"
    class="org.springframework.batch.integration.chunk.ChunkMessageChannelItemWriter"
    scope="step">
  <property name="messagingOperations" ref="messagingTemplate"/>
  <property name="replyChannel" ref="replies"/>
</bean>

<int:channel id="replies">
  <int:queue/>
</int:channel>

<int-jms:message-driven-channel-adapter id="jmsReplies"
    destination-name="replies"
    channel="replies"/>
```


Java配置


```java
@Bean
public org.apache.activemq.ActiveMQConnectionFactory connectionFactory() {
    ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory();
    factory.setBrokerURL("tcp://localhost:61616");
    return factory;
}

/*
 * Configure outbound flow (requests going to workers)
 */
@Bean
public DirectChannel requests() {
    return new DirectChannel();
}

@Bean
public IntegrationFlow outboundFlow(ActiveMQConnectionFactory connectionFactory) {
    return IntegrationFlows
            .from(requests())
            .handle(Jms.outboundAdapter(connectionFactory).destination("requests"))
            .get();
}

/*
 * Configure inbound flow (replies coming from workers)
 */
@Bean
public QueueChannel replies() {
    return new QueueChannel();
}

@Bean
public IntegrationFlow inboundFlow(ActiveMQConnectionFactory connectionFactory) {
    return IntegrationFlows
            .from(Jms.messageDrivenChannelAdapter(connectionFactory).destination("replies"))
            .channel(replies())
            .get();
}

/*
 * Configure the ChunkMessageChannelItemWriter
 */
@Bean
public ItemWriter<Integer> itemWriter() {
    MessagingTemplate messagingTemplate = new MessagingTemplate();
    messagingTemplate.setDefaultChannel(requests());
    messagingTemplate.setReceiveTimeout(2000);
    ChunkMessageChannelItemWriter<Integer> chunkMessageChannelItemWriter
            = new ChunkMessageChannelItemWriter<>();
    chunkMessageChannelItemWriter.setMessagingOperations(messagingTemplate);
    chunkMessageChannelItemWriter.setReplyChannel(replies());
    return chunkMessageChannelItemWriter;
}
```


前面的配置为我们提供了许多bean . 我们使用ActiveMQ和Spring Integration提供的入站/出站JMS适配器配置我们的消息传递中间件 . 如图所示，我们的作业步骤引用的 `itemWriter`  bean使用 `ChunkMessageChannelItemWriter` 在已配置的中间件上编写块 . 

现在我们可以继续进行worker配置，如下例所示：

XML配置


```xml
<bean id="connectionFactory" class="org.apache.activemq.ActiveMQConnectionFactory">
  <property name="brokerURL" value="tcp://localhost:61616"/>
</bean>

<int:channel id="requests"/>
<int:channel id="replies"/>

<int-jms:message-driven-channel-adapter id="incomingRequests"
    destination-name="requests"
    channel="requests"/>

<int-jms:outbound-channel-adapter id="outgoingReplies"
    destination-name="replies"
    channel="replies">
</int-jms:outbound-channel-adapter>

<int:service-activator id="serviceActivator"
    input-channel="requests"
    output-channel="replies"
    ref="chunkProcessorChunkHandler"
    method="handleChunk"/>

<bean id="chunkProcessorChunkHandler"
    class="org.springframework.batch.integration.chunk.ChunkProcessorChunkHandler">
  <property name="chunkProcessor">
    <bean class="org.springframework.batch.core.step.item.SimpleChunkProcessor">
      <property name="itemWriter">
        <bean class="io.spring.sbi.PersonItemWriter"/>
      </property>
      <property name="itemProcessor">
        <bean class="io.spring.sbi.PersonItemProcessor"/>
      </property>
    </bean>
  </property>
</bean>
```


Java配置


```java
@Bean
public org.apache.activemq.ActiveMQConnectionFactory connectionFactory() {
    ActiveMQConnectionFactory factory = new ActiveMQConnectionFactory();
    factory.setBrokerURL("tcp://localhost:61616");
    return factory;
}

/*
 * Configure inbound flow (requests coming from the master)
 */
@Bean
public DirectChannel requests() {
    return new DirectChannel();
}

@Bean
public IntegrationFlow inboundFlow(ActiveMQConnectionFactory connectionFactory) {
    return IntegrationFlows
            .from(Jms.messageDrivenChannelAdapter(connectionFactory).destination("requests"))
            .channel(requests())
            .get();
}

/*
 * Configure outbound flow (replies going to the master)
 */
@Bean
public DirectChannel replies() {
    return new DirectChannel();
}

@Bean
public IntegrationFlow outboundFlow(ActiveMQConnectionFactory connectionFactory) {
    return IntegrationFlows
            .from(replies())
            .handle(Jms.outboundAdapter(connectionFactory).destination("replies"))
            .get();
}

/*
 * Configure the ChunkProcessorChunkHandler
 */
@Bean
@ServiceActivator(inputChannel = "requests", outputChannel = "replies")
public ChunkProcessorChunkHandler<Integer> chunkProcessorChunkHandler() {
    ChunkProcessor<Integer> chunkProcessor
            = new SimpleChunkProcessor<>(itemProcessor(), itemWriter());
    ChunkProcessorChunkHandler<Integer> chunkProcessorChunkHandler
            = new ChunkProcessorChunkHandler<>();
    chunkProcessorChunkHandler.setChunkProcessor(chunkProcessor);
    return chunkProcessorChunkHandler;
}
```


主配置中的大多数配置项应该看起来很熟悉 . 工作人员不需要访问Spring Batch  `JobRepository` 也不需要访问实际的作业配置文件 . 感兴趣的主要  beans 是 `chunkProcessorChunkHandler`  .   `ChunkProcessorChunkHandler` 的 `chunkProcessor` 属性采用已配置的 `SimpleChunkProcessor` ，您可以在此处提供对 `ItemWriter` （以及可选地，您的 `ItemProcessor` ）的引用，该引用将在从主服务器接收块时在工作程序上运行 . 

有关更多信息，请参阅[Remote Chunking](https://docs.spring.io/spring-batch/reference/html/scalability.html#remoteChunking)的"Scalability"章节 . 

从版本4.1开始，Spring Batch Integration引入了 `@EnableBatchIntegration` 注释，可用于简化远程分块设置 . 此批注提供了两个可以在应用程序上下文中自动装配的bean：


- 
 `RemoteChunkingMasterStepBuilderFactory` ：用于配置主步骤


- 
 `RemoteChunkingWorkerBuilder` ：用于配置远程工作者集成流程

这些API负责配置许多组件，如下图所示：


![Remote Chunking Configuration](https://www.docs4dev.com/images/343624b3-e002-412e-aa2b-f9536e3481b1.png)


图4.远程分块配置

在主控方面， `RemoteChunkingMasterStepBuilderFactory` 允许您通过声明来配置主步骤：


- 
项目读者阅读项目并将其发送给 Worker 


- 
输出通道（“传出请求”）向工作人员发送请求


- 
输入渠道（“传入回复”）以接收 Worker 的回复

不需要显式配置 `ChunkMessageChannelItemWriter` 和 `MessagingTemplate` （如果需要，仍然可以显式配置它们） . 

在工作方， `RemoteChunkingWorkerBuilder` 允许您将工作程序配置为：


- 
收听主设备在输入通道上发送的请求（“传入请求”）


- 
使用已配置的 `ItemProcessor` 和 `ItemWriter` 为每个请求调用 `handleChunk` 的 `handleChunk` 方法


- 
将输出通道上的回复（“外发回复”）发送给主站

无需显式配置 `SimpleChunkProcessor` 和 `ChunkProcessorChunkHandler` （如果需要，可以显式配置它们） . 

以下示例显示了如何使用这些API：


```java
@EnableBatchIntegration
@EnableBatchProcessing
public class RemoteChunkingJobConfiguration {

    @Configuration
    public static class MasterConfiguration {

        @Autowired
        private RemoteChunkingMasterStepBuilderFactory masterStepBuilderFactory;

        @Bean
        public TaskletStep masterStep() {
            return this.masterStepBuilderFactory.get("masterStep")
                       .chunk(100)
                       .reader(itemReader())
                       .outputChannel(requests()) // requests sent to workers
                       .inputChannel(replies())   // replies received from workers
                       .build();
        }

        // Middleware beans setup omitted

    }

    @Configuration
    public static class WorkerConfiguration {

        @Autowired
        private RemoteChunkingWorkerBuilder workerBuilder;

        @Bean
        public IntegrationFlow workerFlow() {
            return this.workerBuilder
                       .itemProcessor(itemProcessor())
                       .itemWriter(itemWriter())
                       .inputChannel(requests()) // requests received from the master
                       .outputChannel(replies()) // replies sent to the master
                       .build();
        }

        // Middleware beans setup omitted

    }

}
```


您可以找到远程分块作业的完整示例[here](https://github.com/spring-projects/spring-batch/tree/master/spring-batch-samples#remote-chunking-sample) . 


##### Remote Partitioning


![Remote Partitioning](https://www.docs4dev.com/images/66b16566-bff1-44fb-b443-679c650585c5.png)


图5.远程分区

另一方面，远程分区在不是项目处理而是导致瓶颈的相关I / O时非常有用 . 使用远程分区，可以将工作分配给执行完整Spring Batch步骤的工作人员 . 因此，每个 Worker 都有自己的 `ItemReader` ， `ItemProcessor` 和 `ItemWriter`  . 为此，Spring Batch Integration提供 `MessageChannelPartitionHandler`  . 

 `PartitionHandler` 接口的此实现使用 `MessageChannel` 实例向远程工作人员发送指令并接收他们的响应 . 这提供了用于与远程工作者通信的传输（例如JMS和AMQP）的良好抽象 . 

"Scalability"章节中涉及[remote partitioning](scalability.html#partitioning)的部分概述了配置远程分区所需的概念和组件，并显示了在单独的本地执行线程中使用缺省 `TaskExecutorPartitionHandler` 分区的示例 . 对于远程分区到多个JVM，需要两个额外的组件：


- 
远程织物或网格环境


- 
支持所需远程处理结构或网格环境的 `PartitionHandler` 实现

与远程分块类似，JMS可以用作"remoting fabric" . 在这种情况下，使用 `MessageChannelPartitionHandler` 实例作为 `PartitionHandler` 实现，如上所述 . 以下示例假定现有的分区作业，并侧重于 `MessageChannelPartitionHandler` 和JMS配置：

XML配置


```xml
<bean id="partitionHandler"
   class="org.springframework.batch.integration.partition.MessageChannelPartitionHandler">
  <property name="stepName" value="step1"/>
  <property name="gridSize" value="3"/>
  <property name="replyChannel" ref="outbound-replies"/>
  <property name="messagingOperations">
    <bean class="org.springframework.integration.core.MessagingTemplate">
      <property name="defaultChannel" ref="outbound-requests"/>
      <property name="receiveTimeout" value="100000"/>
    </bean>
  </property>
</bean>

<int:channel id="outbound-requests"/>
<int-jms:outbound-channel-adapter destination="requestsQueue"
    channel="outbound-requests"/>

<int:channel id="inbound-requests"/>
<int-jms:message-driven-channel-adapter destination="requestsQueue"
    channel="inbound-requests"/>

<bean id="stepExecutionRequestHandler"
    class="org.springframework.batch.integration.partition.StepExecutionRequestHandler">
  <property name="jobExplorer" ref="jobExplorer"/>
  <property name="stepLocator" ref="stepLocator"/>
</bean>

<int:service-activator ref="stepExecutionRequestHandler" input-channel="inbound-requests"
    output-channel="outbound-staging"/>

<int:channel id="outbound-staging"/>
<int-jms:outbound-channel-adapter destination="stagingQueue"
    channel="outbound-staging"/>

<int:channel id="inbound-staging"/>
<int-jms:message-driven-channel-adapter destination="stagingQueue"
    channel="inbound-staging"/>

<int:aggregator ref="partitionHandler" input-channel="inbound-staging"
    output-channel="outbound-replies"/>

<int:channel id="outbound-replies">
  <int:queue/>
</int:channel>

<bean id="stepLocator"
    class="org.springframework.batch.integration.partition.BeanFactoryStepLocator" />
```


Java配置


```java
/*
 * Configuration of the master side
 */
@Bean
public PartitionHandler partitionHandler() {
    MessageChannelPartitionHandler partitionHandler = new MessageChannelPartitionHandler();
    partitionHandler.setStepName("step1");
    partitionHandler.setGridSize(3);
    partitionHandler.setReplyChannel(outboundReplies());
    MessagingTemplate template = new MessagingTemplate();
    template.setDefaultChannel(outboundRequests());
    template.setReceiveTimeout(100000);
    partitionHandler.setMessagingOperations(template);
    return partitionHandler;
}

@Bean
public QueueChannel outboundReplies() {
    return new QueueChannel();
}

@Bean
public DirectChannel outboundRequests() {
    return new DirectChannel();
}

@Bean
public IntegrationFlow outboundJmsRequests() {
    return IntegrationFlows.from("outboundRequests")
            .handle(Jms.outboundGateway(connectionFactory())
                    .requestDestination("requestsQueue"))
            .get();
}

@Bean
@ServiceActivator(inputChannel = "inboundStaging")
public AggregatorFactoryBean partitioningMessageHandler() throws Exception {
    AggregatorFactoryBean aggregatorFactoryBean = new AggregatorFactoryBean();
    aggregatorFactoryBean.setProcessorBean(partitionHandler());
    aggregatorFactoryBean.setOutputChannel(outboundReplies());
    // configure other propeties of the aggregatorFactoryBean
    return aggregatorFactoryBean;
}

@Bean
public DirectChannel inboundStaging() {
    return new DirectChannel();
}

@Bean
public IntegrationFlow inboundJmsStaging() {
    return IntegrationFlows
            .from(Jms.messageDrivenChannelAdapter(connectionFactory())
                    .configureListenerContainer(c -> c.subscriptionDurable(false))
                    .destination("stagingQueue"))
            .channel(inboundStaging())
            .get();
}

/*
 * Configuration of the worker side
 */
@Bean
public StepExecutionRequestHandler stepExecutionRequestHandler() {
    StepExecutionRequestHandler stepExecutionRequestHandler = new StepExecutionRequestHandler();
    stepExecutionRequestHandler.setJobExplorer(jobExplorer);
    stepExecutionRequestHandler.setStepLocator(stepLocator());
    return stepExecutionRequestHandler;
}

@Bean
@ServiceActivator(inputChannel = "inboundRequests", outputChannel = "outboundStaging")
public StepExecutionRequestHandler serviceActivator() throws Exception {
    return stepExecutionRequestHandler();
}

@Bean
public DirectChannel inboundRequests() {
    return new DirectChannel();
}

public IntegrationFlow inboundJmsRequests() {
    return IntegrationFlows
            .from(Jms.messageDrivenChannelAdapter(connectionFactory())
                    .configureListenerContainer(c -> c.subscriptionDurable(false))
                    .destination("requestsQueue"))
            .channel(inboundRequests())
            .get();
}

@Bean
public DirectChannel outboundStaging() {
    return new DirectChannel();
}

@Bean
public IntegrationFlow outboundJmsStaging() {
    return IntegrationFlows.from("outboundStaging")
            .handle(Jms.outboundGateway(connectionFactory())
                    .requestDestination("stagingQueue"))
            .get();
}
```


您还必须确保分区 `handler` 属性映射到 `partitionHandler`  bean，如以下示例所示：

XML配置


```xml
<job id="personJob">
  <step id="step1.master">
    <partition partitioner="partitioner" handler="partitionHandler"/>
    ...
  </step>
</job>
```


Java配置


```java
public Job personJob() {
                return jobBuilderFactory.get("personJob")
                                .start(stepBuilderFactory.get("step1.master")
                                                .partitioner("step1.worker", partitioner())
                                                .partitionHandler(partitionHandler())
                                                .build())
                                .build();
        }
```


您可以找到远程分区作业的完整示例[here](https://github.com/spring-projects/spring-batch/tree/master/spring-batch-samples#remote-partitioning-sample) . 

 `@EnableBatchIntegration` 注释可用于简化远程分区设置 . 此批注提供了两个对远程分区有用的bean：


- 
 `RemotePartitioningMasterStepBuilderFactory` ：用于配置主步骤


- 
 `RemotePartitioningWorkerStepBuilderFactory` ：用于配置工作人员步骤

这些API负责配置许多组件，如下图所示：


![Remote Partitioning Configuration (with job repository polling)](https://www.docs4dev.com/images/107973a0-3523-44b6-8f7e-07609a637ede.png)


图6.远程分区配置（带有作业存储库轮询）


![Remote Partitioning Configuration (with replies aggregation)](https://www.docs4dev.com/images/520a689b-13b4-4f48-8de4-c99d0822ebcd.png)


图7.远程分区配置（带有回复聚合）

在主控方面， `RemotePartitioningMasterStepBuilderFactory` 允许您通过声明来配置主步骤：


- 
 `Partitioner` 用于分区数据


- 
输出通道（“传出请求”）向工作人员发送请求


- 
输入通道（“传入回复”）以接收来自工作人员的回复（配置回复聚合时）


- 
轮询间隔和超时参数（配置作业存储库轮询时）

不需要显式配置 `MessageChannelPartitionHandler` 和 `MessagingTemplate` （如果需要，仍然可以显式配置） . 

在工作方， `RemotePartitioningWorkerStepBuilderFactory` 允许您将工作程序配置为：


- 
收听主设备在输入通道上发送的请求（“传入请求”）


- 
为每个请求调用 `StepExecutionRequestHandler` 的 `handle` 方法


- 
将输出通道上的回复（“外发回复”）发送给主站

无需显式配置 `StepExecutionRequestHandler` （如果需要，可以显式配置） . 

以下示例显示了如何使用这些API：


```java
@Configuration
@EnableBatchProcessing
@EnableBatchIntegration
public class RemotePartitioningJobConfiguration {

    @Configuration
    public static class MasterConfiguration {

        @Autowired
        private RemotePartitioningMasterStepBuilderFactory masterStepBuilderFactory;

        @Bean
        public Step masterStep() {
                 return this.masterStepBuilderFactory
                    .get("masterStep")
                    .partitioner("workerStep", partitioner())
                    .gridSize(10)
                    .outputChannel(outgoingRequestsToWorkers())
                    .inputChannel(incomingRepliesFromWorkers())
                    .build();
        }

        // Middleware beans setup omitted

    }

    @Configuration
    public static class WorkerConfiguration {

        @Autowired
        private RemotePartitioningWorkerStepBuilderFactory workerStepBuilderFactory;

        @Bean
        public Step workerStep() {
                 return this.workerStepBuilderFactory
                    .get("workerStep")
                    .inputChannel(incomingRequestsFromMaster())
                    .outputChannel(outgoingRepliesToMaster())
                    .chunk(100)
                    .reader(itemReader())
                    .processor(itemProcessor())
                    .writer(itemWriter())
                    .build();
        }

        // Middleware beans setup omitted

    }

}
```
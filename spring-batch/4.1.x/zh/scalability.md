## 1.缩放和并行处理

XML Java

单线程，单个进程作业可以解决许多批处理问题，因此在考虑更复杂的实现之前，最好先检查一下是否满足您的需求 . 测量实际工作的性能，看看最简单的实现是否满足您的需求 . 即使使用标准硬件，您也可以在一分钟内读取和写入几百兆字节的文件 . 

当您准备开始使用某些并行处理来实现作业时，Spring Batch提供了一系列选项，这些选项将在本章中介绍，尽管某些功能在其他地方有所介绍 . 在较高的层次上，有两种并行处理模式：


- 
单进程，多线程


- 
多进程

这些也分为几类，如下：


- 
多线程步骤（单个进程）


- 
并行步骤（单个过程）


- 
远程分块步骤（多进程）


- 
分区步骤（单个或多个过程）

首先，我们审查单一流程选项 . 然后我们回顾一下多进程选项 . 


### 1.1.多线程步骤

最简单的启动并行处理的方法是在步骤配置中添加 `TaskExecutor`  . 

例如，您可以添加 `tasklet` 的属性，如以下示例所示：


```xml
<step id="loading">
    <tasklet task-executor="taskExecutor">...</tasklet>
</step>
```


使用java配置时，可以在步骤中添加 `TaskExecutor` ，如以下示例所示：

Java配置


```java
@Bean
public TaskExecutor taskExecutor(){
    return new SimpleAsyncTaskExecutor("spring_batch");
}

@Bean
public Step sampleStep(TaskExecutor taskExecutor) {
        return this.stepBuilderFactory.get("sampleStep")
                                .<String, String>chunk(10)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .taskExecutor(taskExecutor)
                                .build();
}
```


在此示例中， `taskExecutor` 是对另一个实现 `TaskExecutor` 接口的bean定义的引用 .  [TaskExecutor](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/core/task/TaskExecutor.html)是标准的Spring接口，因此请参阅Spring用户指南以获取可用实现的详细信息 . 最简单的多线程 `TaskExecutor` 是 `SimpleAsyncTaskExecutor`  . 

上述配置的结果是 `Step` 通过在单独的执行线程中读取，处理和写入每个项目块（每个提交间隔）来执行 . 请注意，这意味着要处理的项目没有固定的顺序，并且块可能包含与单线程案例相比非连续的项目 . 除了任务执行程序提供的任何限制（例如它是否由线程池支持）之外，tasklet配置中还有一个限制，默认为4.您可能需要增加此限制以确保线程池是充分利用 . 

例如，您可能会增加限制限制，如以下示例所示：


```xml
<step id="loading"> <tasklet
    task-executor="taskExecutor"
    throttle-limit="20">...</tasklet>
</step>
```


使用java配置时，构建器提供对限制的限制：

Java配置


```java
@Bean
public Step sampleStep(TaskExecutor taskExecutor) {
        return this.stepBuilderFactory.get("sampleStep")
                                .<String, String>chunk(10)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .taskExecutor(taskExecutor)
                                .throttleLimit(20)
                                .build();
}
```


另请注意，步骤中使用的任何池化资源可能会限制并发性，例如 `DataSource`  . 确保在这些资源中使池至少与步骤中所需的并发线程数一样大 . 

对于一些常见的批处理用例，使用多线程 `Step` 实现存在一些实际限制 .   `Step` （如读者和作家）的许多参与者都是有状态的 . 如果状态没有被线程隔离，那么这些组件在多线程 `Step` 中不可用 . 特别是，Spring Batch的大多数现成的读者和编写者都不是为多线程使用而设计的 . 但是，可以使用无状态或线程安全的读取器和编写器，并且[Spring Batch Samples](https://github.com/spring-projects/spring-batch/tree/master/spring-batch-samples)中有一个示例（称为 `parallelJob` ），它显示了使用过程指示器（请参阅[Preventing State Persistence](readersAndWriters.html#process-indicator)）来跟踪已处理的项目在数据库输入表中 . 

Spring Batch提供了 `ItemWriter` 和 `ItemReader` 的一些实现 . 通常，他们在Javadoc中说他们是否是线程安全的，或者你需要做些什么来避免并发环境中的问题 . 如果Javadoc中没有信息，您可以检查实现以查看是否存在任何状态 . 如果读者不是线程安全的，您可以使用提供的 `SynchronizedItemStreamReader` 来装饰它，或者在您自己的同步委托器中使用它 . 您可以将调用同步到 `read()` ，只要处理和写入是块中最昂贵的部分，您的步骤可能仍然比在单线程配置中完成得快得多 . 


### 1.2.平行步骤

只要需要并行化的应用程序逻辑可以分成不同的职责并分配给各个步骤，那么它就可以在一个进程中并行化 . 并行步执行易于配置和使用 . 

例如，与_14799并行执行步骤 `(step1,step2)` 非常简单，如以下示例所示：


```xml
<job id="job1">
    <split id="split1" task-executor="taskExecutor" next="step4">
        <flow>
            <step id="step1" parent="s1" next="step2"/>
            <step id="step2" parent="s2"/>
        </flow>
        <flow>
            <step id="step3" parent="s3"/>
        </flow>
    </split>
    <step id="step4" parent="s4"/>
</job>

<beans:bean id="taskExecutor" class="org.spr...SimpleAsyncTaskExecutor"/>
```


使用java配置时，与 `step3` 并行执行步骤 `(step1,step2)` 非常简单，如以下示例所示：

Java配置


```java
@Bean
public Job job() {
    return jobBuilderFactory.get("job")
        .start(splitFlow())
        .next(step4())
        .build()        //builds FlowJobBuilder instance
        .build();       //builds Job instance
}

@Bean
public Flow splitFlow() {
    return new FlowBuilder<SimpleFlow>("splitFlow")
        .split(taskExecutor())
        .add(flow1(), flow2())
        .build();
}

@Bean
public Flow flow1() {
    return new FlowBuilder<SimpleFlow>("flow1")
        .start(step1())
        .next(step2())
        .build();
}

@Bean
public Flow flow2() {
    return new FlowBuilder<SimpleFlow>("flow2")
        .start(step3())
        .build();
}

@Bean
public TaskExecutor taskExecutor(){
    return new SimpleAsyncTaskExecutor("spring_batch");
}
```


可配置任务执行程序用于指定应该使用哪个 `TaskExecutor` 实现来执行各个流 . 默认值为 `SyncTaskExecutor` ，但需要异步 `TaskExecutor` 才能并行运行这些步骤 . 请注意，作业确保在聚合退出状态和转换之前，拆分中的每个流都完成 . 

有关详细信息，请参阅[Split Flows](step.html#split-flows)部分 . 


### 1.3.远程分块

在远程分块中， `Step` 处理分为多个进程，通过一些中间件相互通信 . 下图显示了该模式：


![Remote Chunking](https://www.docs4dev.com/images/9f8424b9-635e-4737-a172-11d5467a6222.png)


图1.远程分块

主组件是单个进程，从属组件是多个远程进程 . 如果主设备不是瓶颈，这种模式效果最好，因此处理必须比读取项目更昂贵（在实践中通常是这种情况） . 

master是Spring Batch  `Step` 的一个实现， `ItemWriter` 被一个通用版本取代，该版本知道如何将消息块作为消息发送到中间件 . 对于任何正在使用的中间件（例如，使用JMS，它们将是 `MesssageListener` 实现），从属是标准侦听器，并且它们的作用是使用标准 `ItemWriter` 或 `ItemProcessor` 加 `ItemWriter` 通过 `ChunkProcessor` 接口处理项目块 . 使用这种模式的一个优点是阅读器，处理器和写入器组件是现成的（与用于本地执行步骤的相同） . 这些项目是动态划分的，工作通过中间件共享，因此，如果监听器都是渴望消费者，那么负载 balancer 是自动的 . 

中间件必须经久耐用，保证交付，每个消息都有一个消费者 .  JMS是显而易见的候选者，但其他选项（如JavaSpaces）存在于网格计算和共享内存产品领域 . 

有关详细信息，请参阅[Spring Batch Integration - Remote Chunking](spring-batch-integration.html#remote-chunking)部分 . 


### 1.4.分区

Spring Batch还提供了一个SPI，用于分区 `Step` 执行并远程执行 . 在这种情况下，远程参与者是 `Step` 实例，可以很容易地配置并用于本地处理 . 下图显示了该模式：


![Partitioning Overview](https://www.docs4dev.com/images/6d29332a-8ec4-48f2-8134-2cee9e79371b.png)


图2.分区

 `Job` 在左侧作为一系列 `Step` 实例运行，其中一个 `Step` 实例标记为主实例 . 这张照片中的奴隶都是 `Step` 的相同实例，实际上可以代替主人，导致 `Job` 的结果相同 . 从属服务器通常是远程服务，但也可以是本地执行线程 . 主设备以此模式发送给从设备的消息不需要经久耐用或保证交付 .   `JobRepository` 中的Spring Batch元数据确保每个从属执行一次，每次只执行一次 `Job`  . 

Spring Batch中的SPI由 `Step` （称为 `PartitionStep` ）的特殊实现和两个需要针对特定环境实现的策略接口组成 . 策略接口是 `PartitionHandler` 和 `StepExecutionSplitter` ，它们的作用如下面的序列图所示：


![Partitioning SPI](https://www.docs4dev.com/images/f3b6d276-fdaf-4941-9d34-3b3965e533c8.png)


图3.分区SPI

在这种情况下，右边的 `Step` 是"remote"奴隶，因此，可能有许多对象和/或进程正在扮演这个角色，并且 `PartitionStep` 被显示为驱动执行 . 

以下示例显示了 `PartitionStep` 配置：


```xml
<step id="step1.master">
    <partition step="step1" partitioner="partitioner">
        <handler grid-size="10" task-executor="taskExecutor"/>
    </partition>
</step>
```


以下示例显示了使用java配置的 `PartitionStep` 配置：

Java配置


```java
@Bean
public Step step1Master() {
    return stepBuilderFactory.get("step1.master")
        .<String, String>partitioner("step1", partitioner())
        .step(step1())
        .gridSize(10)
        .taskExecutor(taskExecutor())
        .build();
}
```


与多线程步骤的 `throttle-limit` 属性类似， `grid-size` 属性可防止任务执行程序饱和来自单个步骤的请求 . 

有一个简单的例子可以在单元测试套件中复制和扩展[Spring Batch Samples](https://github.com/spring-projects/spring-batch/tree/master/spring-batch-samples/src/main/resources/jobs)（参见 `Partition*Job.xml` 配置） . 

Spring Batch为名为"step1:partition0"的分区创建步骤执行，依此类推 . 许多人更喜欢将主步骤"step1:master"称为一致性 . 您可以为步骤使用别名（通过指定 `name` 属性而不是 `id` 属性） . 


#### 1.4.1. PartitionHandler

 `PartitionHandler` 是了解远程处理或网格环境结构的组件 . 它能够将 `StepExecution` 个请求发送到远程 `Step` 实例，并以特定于结构的格式包装，如DTO . 它不必知道如何拆分输入数据或如何聚合多个 `Step` 执行的结果 . 一般来说，它可能也不需要了解弹性或故障转移，因为在许多情况下这些都是结构的特征 . 无论如何，Spring Batch始终提供独立于结构的可重启性 . 失败的 `Job` 总是可以重新启动，只有失败的 `Steps` 被重新执行 . 

 `PartitionHandler` 接口可以具有各种结构类型的专用实现，包括简单的RMI远程处理，EJB远程处理，自定义Web服务，JMS，Java空间，共享内存网格（如Terracotta或Coherence）和网格执行结构（如GridGain） .  Spring Batch不包含任何专有网格或远程结构的实现 . 

但是，Spring Batch提供了一个有用的 `PartitionHandler` 实现，它使用Spring的 `TaskExecutor` 策略在不同的执行线程中本地执行 `Step` 实例 . 该实现称为 `TaskExecutorPartitionHandler`  . 

 `TaskExecutorPartitionHandler` 是使用前面显示的XML命名空间配置的步骤的默认值 . 它也可以显式配置，如以下示例所示：


```xml
<step id="step1.master">
    <partition step="step1" handler="handler"/>
</step>

<bean class="org.spr...TaskExecutorPartitionHandler">
    <property name="taskExecutor" ref="taskExecutor"/>
    <property name="step" ref="step1" />
    <property name="gridSize" value="10" />
</bean>
```


可以在java配置中显式配置 `TaskExecutorPartitionHandler` ，如以下示例所示：

Java配置


```java
@Bean
public Step step1Master() {
    return stepBuilderFactory.get("step1.master")
        .partitioner("step1", partitioner())
        .partitionHandler(partitionHandler())
        .build();
}

@Bean
public PartitionHandler partitionHandler() {
    TaskExecutorPartitionHandler retVal = new TaskExecutorPartitionHandler();
    retVal.setTaskExecutor(taskExecutor());
    retVal.setStep(step1());
    retVal.setGridSize(10);
    return retVal;
}
```


 `gridSize` 属性确定要创建的单独步骤执行的数量，因此可以匹配 `TaskExecutor` 中线程池的大小 . 或者，可以将其设置为大于可用线程数，这使得工作块更小 . 

 `TaskExecutorPartitionHandler` 对IO密集型 `Step` 实例非常有用，例如复制大量文件或将文件系统复制到内容管理系统中 . 它还可以通过提供 `Step` 实现来远程执行，该实现是远程调用的代理（例如使用Spring Remoting） . 


#### 1.4.2.分区员

 `Partitioner` 有一个更简单的责任：生成执行上下文作为新步骤执行的输入参数只（不用担心重启） . 它有一个方法，如以下接口定义所示：


```java
public interface Partitioner {
    Map<String, ExecutionContext> partition(int gridSize);
}
```


此方法的返回值将每个步骤执行的唯一名称（ `String` ）与 `ExecutionContext` 形式的输入参数相关联 . 这些名称稍后会在批处理元数据中显示为分区 `StepExecutions` 中的步骤名称 .   `ExecutionContext` 只是一包名称 - 值对，因此它可能包含一系列主键，行号或输入文件的位置 . 然后，远程 `Step` 通常使用 `#{…}` 占位符（步骤作用域中的后期绑定）绑定到上下文输入，如下一节所示 . 

步骤执行的名称（ `Partitioner` 返回的 `Map` 中的键）在 `Job` 的步骤执行中必须是唯一的，但没有任何其他特定要求 . 最简单的方法（并使名称对用户有意义）是使用前缀后缀命名约定，其中前缀是正在执行的步骤的名称（它本身在 `Job` 中是唯一的），以及后缀只是一个柜台 . 框架中有一个 `SimplePartitioner` 使用此约定 . 

可以使用名为 `PartitionNameProvider` 的可选接口与分区本身分开提供分区名称 . 如果 `Partitioner` 实现此接口，则在重新启动时，仅查询名称 . 如果分区很昂贵，这可能是一个有用的优化 .   `PartitionNameProvider` 提供的名称必须与 `Partitioner` 提供的名称相匹配 . 


#### 1.4.3.将输入数据绑定到步骤

 `PartitionHandler` 执行的步骤具有相同的配置，并且它们的输入参数在运行时从 `ExecutionContext` 绑定，这非常有效 . 使用Spring Batch的StepScope功能很容易做到（在[Late Binding](step.html#late-binding)一节中有更详细的介绍） . 例如，如果 `Partitioner` 使用名为 `fileName` 的属性键创建 `ExecutionContext` 实例，指向每个步骤调用的不同文件（或目录），则 `Partitioner` 输出可能类似于下表的内容：

||
|步骤执行名称（键）| ExecutionContext（value）|
| ---- | ---- |
| filecopy：partition0 | fileName = / home / data / one |
| filecopy：partition1 | fileName = / home / data / two |
| filecopy：partition2 | fileName = / home / data / three |

然后，可以使用后期绑定到执行上下文将文件名绑定到步骤，如以下示例所示：

XML配置


```xml
<bean id="itemReader" scope="step"
      class="org.spr...MultiResourceItemReader">
    <property name="resources" value="#{stepExecutionContext[fileName]}/*"/>
</bean>
```


Java配置


```java
@Bean
public MultiResourceItemReader itemReader(
        @Value("#{stepExecutionContext['fileName']}/*") Resource [] resources) {
        return new MultiResourceItemReaderBuilder<String>()
                        .delegate(fileReader())
                        .name("itemReader")
                        .resources(resources)
                        .build();
}
```
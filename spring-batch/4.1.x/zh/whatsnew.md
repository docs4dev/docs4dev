## 1. Spring Batch 4.1中的新功能

Spring Batch 4.1版本增加了以下功能：


- 
一个新的 `@SpringBatchTest` 注释，用于简化测试批处理组件


- 
一个新的 `@EnableBatchIntegration` 注释，用于简化远程分块和分区配置


- 
一个新的 `JsonItemReader` 和 `JsonFileItemWriter` 来支持JSON格式


- 
添加对使用Bean Validation API验证项目的支持


- 
添加对JSR-305注释的支持


- 
 `FlatFileItemWriterBuilder`  API的增强功能


### 1.1. @SpringBatchTest Annotation

Spring Batch提供了一些不错的实用程序类（例如 `JobLauncherTestUtils` 和 `JobRepositoryTestUtils` ）和测试执行侦听器（ `StepScopeTestExecutionListener` 和 `JobScopeTestExecutionListener` ）来测试批处理组件 . 但是，要使用这些实用程序，必须显式配置它们 . 此版本引入了一个名为 `@SpringBatchTest` 的新注释，它自动将实用程序bean和侦听器添加到测试上下文中，并使它们可用于自动装配，如以下示例所示：


```java
@RunWith(SpringRunner.class)
@SpringBatchTest
@ContextConfiguration(classes = {JobConfiguration.class})
public class JobTest {

   @Autowired
   private JobLauncherTestUtils jobLauncherTestUtils;

   @Autowired
   private JobRepositoryTestUtils jobRepositoryTestUtils;


   @Before
   public void clearMetadata() {
      jobRepositoryTestUtils.removeJobExecutions();
   }

   @Test
   public void testJob() throws Exception {
      // given
      JobParameters jobParameters =
            jobLauncherTestUtils.getUniqueJobParameters();

      // when
      JobExecution jobExecution =
            jobLauncherTestUtils.launchJob(jobParameters);

      // then
      Assert.assertEquals(ExitStatus.COMPLETED,
                          jobExecution.getExitStatus());
   }

}
```


有关此新注释的更多详细信息，请参阅[Unit Testing](testing.html#creatingUnitTestClass)一章 . 


### 1.2. @EnableBatchIntegration注释

设置远程分块作业需要定义多个bean：


- 
用于从消息传递中间件（JMS，AMQP和其他）获取连接的连接工厂


- 
 `MessagingTemplate` 从主服务器向工作人员发送请求，然后再返回


- 
Spring Integration的输入通道和输出通道，用于从消息传递中间件获取消息


- 
主方上的特殊项目编写器（ `ChunkMessageChannelItemWriter` ），它知道如何将数据块发送给工作人员进行处理和写入


- 
工作端的消息监听器（ `ChunkProcessorChunkHandler` ），用于从主服务器接收数据

乍一看，这可能有点令人生畏 . 此版本引入了一个名为 `@EnableBatchIntegration` 的新注释以及新的API（ `RemoteChunkingMasterStepBuilder` 和 `RemoteChunkingWorkerBuilder` ）以简化配置 . 以下示例显示如何使用新注释和API：


```java
@Configuration
@EnableBatchProcessing
@EnableBatchIntegration
public class RemoteChunkingAppConfig {

   @Autowired
   private RemoteChunkingMasterStepBuilderFactory masterStepBuilderFactory;

   @Autowired
   private RemoteChunkingWorkerBuilder workerBuilder;

   @Bean
   public TaskletStep masterStep() {
         return this.masterStepBuilderFactory
                         .get("masterStep")
                         .chunk(100)
                         .reader(itemReader())
                         .outputChannel(outgoingRequestsToWorkers())
                         .inputChannel(incomingRepliesFromWorkers())
                         .build();
   }

   @Bean
   public IntegrationFlow worker() {
         return this.workerBuilder
                         .itemProcessor(itemProcessor())
                         .itemWriter(itemWriter())
                         .inputChannel(incomingRequestsFromMaster())
                         .outputChannel(outgoingRepliesToMaster())
                         .build();
   }

   // Middleware beans setup omitted
}
```


这个新的注释和构建器负责配置基础架构bean的繁重工作 . 您现在可以在工作方轻松配置主步骤和Spring Integration流程 . 您可以在[samples module](https://github.com/spring-projects/spring-batch/tree/master/spring-batch-samples#remote-chunking-sample)中找到使用这些新API的远程分块示例以及[Spring Batch Integration](spring-batch-integration.html#remote-chunking)章节中的更多详细信息 . 

就像远程分块配置简化一样，此版本还引入了新的API来简化远程分区设置： `RemotePartitioningMasterStepBuilder` 和 `RemotePartitioningWorkerStepBuilder`  . 如果存在 `@EnableBatchIntegration` ，则可以在配置类中自动装配这些，如以下示例所示：


```java
@Configuration
@EnableBatchProcessing
@EnableBatchIntegration
public class RemotePartitioningAppConfig {

   @Autowired
   private RemotePartitioningMasterStepBuilderFactory masterStepBuilderFactory;

   @Autowired
   private RemotePartitioningWorkerStepBuilderFactory workerStepBuilderFactory;

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
```


您可以在[Spring Batch Integration](spring-batch-integration.html#remote-partitioning)章节中找到有关这些新API的更多详细信息 . 


### 1.3. JSON支持

Spring Batch 4.1增加了对JSON格式的支持 . 此版本引入了一个新的项目阅读器，可以按以下格式读取JSON资源：


```java
[
  {
    "isin": "123",
    "quantity": 1,
    "price": 1.2,
    "customer": "foo"
  },
  {
    "isin": "456",
    "quantity": 2,
    "price": 1.4,
    "customer": "bar"
  }
]
```


类似于 `StaxEventItemReader`  for XML，新的 `JsonItemReader` 使用流API来读取块中的JSON对象 .  Spring Batch支持两个库：


- 
[Jackson](https://github.com/FasterXML/jackson)


- 
[Gson](https://github.com/google/gson)

要添加其他库，可以实现 `JsonObjectReader` 接口 . 

通过 `JsonFileItemWriter` 也支持编写JSON数据 . 有关JSON支持的更多详细信息，请参阅[ItemReaders and ItemWriters](readersAndWriters.html#jsonReadingWriting)一章 . 


### 1.4. Bean Validation API支持

此版本带来了一个名为 `BeanValidatingItemProcessor` 的新 `ValidatingItemProcessor` 实现，它允许您验证使用Bean Validation API（JSR-303）注释注释的项目 . 例如，给定以下类型 `Person` ：


```java
class Person {

    @NotEmpty
    private String name;

    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

}
```


您可以通过在应用程序上下文中声明 `BeanValidatingItemProcessor`  bean来验证项目，并在面向块的步骤中将其注册为处理器：


```java
@Bean
public BeanValidatingItemProcessor<Person> beanValidatingItemProcessor() throws Exception {
        BeanValidatingItemProcessor<Person> beanValidatingItemProcessor = new BeanValidatingItemProcessor<>();
        beanValidatingItemProcessor.setFilter(true);

        return beanValidatingItemProcessor;
}
```



### 1.5. JSR-305支持

此版本增加了对JSR-305注释的支持 . 它利用Spring Framework的[Null-safety](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#null-safety)注释，并将它们添加到Spring Batch的所有公共API上 . 

这些注释不仅在使用Spring Batch API时强制执行null安全性，而且还可以由IDE用于提供与可空性相关的有用信息 . 例如，如果用户想要实现 `ItemReader` 接口，则任何支持JSR-305注释的IDE都将生成如下内容：


```java
public class MyItemReader implements ItemReader<String> {

        @Nullable
        public String read() throws Exception {
                return null;
        }

}
```


 `read` 方法上的 `@Nullable` 注释清楚地表明此方法的 Contract 表明它可能返回 `null`  . 这强制执行其Javadoc中的内容， `read` 方法应在数据源耗尽时返回 `null`  . 


### 1.6. FlatFileItemWriterBuilder增强功能

此版本中添加的另一个小功能是简化了平面文件写入的配置 . 具体来说，这些更新简化了分隔和固定宽度文件的配置 . 以下是更改前后的示例 . 


```java
// Before
@Bean
public FlatFileItemWriter<Item> itemWriter(Resource resource) {
        BeanWrapperFieldExtractor<Item> fieldExtractor =
            new BeanWrapperFieldExtractor<Item>();
        fieldExtractor.setNames(new String[] {"field1", "field2", "field3"});
        fieldExtractor.afterPropertiesSet();

        DelimitedLineAggregator aggregator = new DelimitedLineAggregator();
        aggregator.setFieldExtractor(fieldExtractor);
        aggregator.setDelimiter(";");

        return new FlatFileItemWriterBuilder<Item>()
                        .name("itemWriter")
                        .resource(resource)
                        .lineAggregator(aggregator)
                        .build();
}

// After
@Bean
public FlatFileItemWriter<Item> itemWriter(Resource resource) {
        return new FlatFileItemWriterBuilder<Item>()
                        .name("itemWriter")
                        .resource(resource)
                        .delimited()
                        .delimiter(";")
                        .names(new String[] {"field1", "field2", "field3"})
                        .build();
}
```
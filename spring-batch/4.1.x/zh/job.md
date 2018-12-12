## 1.配置和运行作业

XML Java

在[domain section](domain.html#domainLanguageOfBatch)中，讨论了整体架构设计，使用下图作为指南：


![Figure 2.1: Batch Stereotypes](https://www.docs4dev.com/images/8330b29a-7b13-4ab0-a8f9-4b56f1426691.png)


图1.批量刻板印象

虽然 `Job` 对象可能看起来像是一个简单的步骤容器，但开发人员必须了解许多配置选项 . 此外，关于如何运行 `Job` 以及如何在该运行期间存储其元数据有许多考虑因素 . 本章将解释 `Job` 的各种配置选项和运行时问题 . 


### 1.1.配置作业

[Job](#configureJob)接口有多种实现，但构建器抽象出配置上的差异 . 


```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .start(playerLoad())
                     .next(gameLoad())
                     .next(playerSummarization())
                     .end()
                     .build();
}
```


 `Job` （通常是其中的任何 `Step` ）需要 `JobRepository`  .   `JobRepository` 的配置通过[BatchConfigurer](#javaConfig)处理 . 

上面的示例说明了一个 `Job` ，它由三个 `Step` 实例组成 . 与作业相关的构建器还可以包含有助于并行化（ `Split` ），声明性流控制（ `Decision` ）和流定义外部化（ `Flow` ）的其他元素 . 

[Job](#configureJob)接口有多种实现，但是，命名空间抽象出配置上的差异 . 它只有三个必需的依赖项：名称， `JobRepository` 和 `Step` 实例列表 . 


```xml
<job id="footballJob">
    <step id="playerload"          parent="s1" next="gameLoad"/>
    <step id="gameLoad"            parent="s2" next="playerSummarization"/>
    <step id="playerSummarization" parent="s3"/>
</job>
```


这里的示例使用父bean定义来创建步骤;请参阅[step configuration](step.html#configureStep)部分以获取更多选项，以内联方式声明特定步骤详细信息 . XML命名空间默认引用ID为“jobRepository”的存储库，这是一个合理的默认值 . 但是，这可以明确地重写：


```xml
<job id="footballJob" job-repository="specialRepository">
    <step id="playerload"          parent="s1" next="gameLoad"/>
    <step id="gameLoad"            parent="s3" next="playerSummarization"/>
    <step id="playerSummarization" parent="s3"/>
</job>
```


除了步骤之外，作业配置还可以包含有助于并行化（ `<split>` ）的其他元素，声明性流控制（ `<decision>` ）和流定义的外部化（ `<flow/>` ） . 


#### 1.1.1.可重启性

执行批处理作业时的一个关键问题涉及 `Job` 重新启动时的行为 . 如果特定 `JobInstance` 已存在 `JobExecution` ，则启动 `Job` 被视为“重启” . 理想情况下，所有工作应该能够从他们中断的地方开始，但有些情况下这是不可能的 . 完全取决于开发人员确保在此方案中创建新的 `JobInstance`  . 但是，Spring Batch确实提供了一些帮助 . 如果永远不应重新启动 `Job` ，但应始终作为新 `JobInstance` 的一部分运行，则可重新启动的属性可能设置为“false”：

XML配置


```xml
<job id="footballJob" restartable="false">
    ...
</job>
```


Java配置


```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .preventRestart()
                     ...
                     .build();
}
```


换句话说，将restartable设置为false意味着“此 `Job` 不支持再次启动” . 重新启动不可重新启动的 `Job` 将导致抛出 `JobRestartException` ：


```java
Job job = new SimpleJob();
job.setRestartable(false);

JobParameters jobParameters = new JobParameters();

JobExecution firstExecution = jobRepository.createJobExecution(job, jobParameters);
jobRepository.saveOrUpdate(firstExecution);

try {
    jobRepository.createJobExecution(job, jobParameters);
    fail();
}
catch (JobRestartException e) {
    // expected
}
```


这段JUnit代码显示了如何第一次为不可重新启动的作业创建 `JobExecution` 将不会导致任何问题 . 但是，第二次尝试将抛出 `JobRestartException`  . 


#### 1.1.2.拦截作业执行

在执行作业的过程中，在其生命周期中通知各种事件以便可以执行自定义代码可能是有用的 .   `SimpleJob` 允许在适当的时候调用 `JobListener` ：


```java
public interface JobExecutionListener {

    void beforeJob(JobExecution jobExecution);

    void afterJob(JobExecution jobExecution);

}
```


 `JobListeners` 可以通过作业上的listeners元素添加到 `SimpleJob` ：

XML配置


```xml
<job id="footballJob">
    <step id="playerload"          parent="s1" next="gameLoad"/>
    <step id="gameLoad"            parent="s2" next="playerSummarization"/>
    <step id="playerSummarization" parent="s3"/>
    <listeners>
        <listener ref="sampleListener"/>
    </listeners>
</job>
```


Java配置


```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                     .listener(sampleListener())
                     ...
                     .build();
}
```


应该注意的是，无论作业成功与否，都会调用 `afterJob`  . 如果需要确定成功或失败，可以从 `JobExecution` 获取：


```java
public void afterJob(JobExecution jobExecution){
    if( jobExecution.getStatus() == BatchStatus.COMPLETED ){
        //job success
    }
    else if(jobExecution.getStatus() == BatchStatus.FAILED){
        //job failure
    }
}
```


与此接口对应的注释是：


- 
 `@BeforeJob` 


- 
 `@AfterJob` 


#### 1.1.3.从父作业继承

如果一组作业共享相似但不相同的配置，那么定义具体作业可以继承属性的"parent"  `Job` 可能会有所帮助 . 与Java中的类继承类似，"child"  `Job` 将其元素和属性与父元素组合在一起 . 

在以下示例中，"baseJob"是一个抽象 `Job` 定义，仅定义一个侦听器列表 .   `Job`  "job1"是一个具体的定义，它继承了"baseJob"的侦听器列表，并将其与自己的侦听器列表合并，以生成具有两个侦听器和一个 `Step` ，"step1"的 `Job`  . 


```xml
<job id="baseJob" abstract="true">
    <listeners>
        <listener ref="listenerOne"/>
    <listeners>
</job>

<job id="job1" parent="baseJob">
    <step id="step1" parent="standaloneStep"/>

    <listeners merge="true">
        <listener ref="listenerTwo"/>
    <listeners>
</job>
```


有关更多详细信息，请参阅[Inheriting from a Parent Step](step.html#inheritingFromParentStep)部分 . 


#### 1.1.4. JobParametersValidator

在XML命名空间中声明的作业或使用 `AbstractJob` 的任何子类可以选择在运行时为作业参数声明验证器 . 例如，当您需要声明作业以其所有必需参数启动时，这非常有用 . 有一个 `DefaultJobParametersValidator` 可用于约束简单强制参数和可选参数的组合，对于更复杂的约束，您可以自己实现接口 . 

XML命名空间通过作业的子元素支持验证器的配置，例如：


```xml
<job id="job1" parent="baseJob3">
    <step id="step1" parent="standaloneStep"/>
    <validator ref="parametersValidator"/>
</job>
```


验证器可以指定为引用（如上所述）或bean命名空间中的嵌套bean定义 . 

通过java构建器支持验证器的配置，例如：


```java
@Bean
public Job job1() {
    return this.jobBuilderFactory.get("job1")
                     .validator(parametersValidator())
                     ...
                     .build();
}
```



### 1.2. Java配置

除了XML之外，Spring 3还提供了通过java配置应用程序的能力 . 从Spring Batch 2.2.0开始，可以使用相同的java配置来配置批处理作业 . 基于java的配置有两个组件： `@EnableBatchProcessing` 注释和两个构建器 . 

 `@EnableBatchProcessing` 的工作方式与Spring系列中的其他@Enable *注释类似 . 在这种情况下， `@EnableBatchProcessing` 提供了用于构建批处理作业的基本配置 . 在此基本配置中，除了可用于自动装配的多个bean之外，还创建了 `StepScope` 的实例：


- 
 `JobRepository`   -  bean name "jobRepository"


- 
 `JobLauncher`   -  bean name "jobLauncher"


- 
 `JobRegistry`   -  bean name "jobRegistry"


- 
 `PlatformTransactionManager`   -  bean name "transactionManager"


- 
 `JobBuilderFactory`   -  bean name "jobBuilders"


- 
 `StepBuilderFactory`   -  bean name "stepBuilders"

此配置的核心接口是 `BatchConfigurer`  . 默认实现提供了上面提到的bean，并且需要 `DataSource` 作为要提供的上下文中的bean .  JobRepository将使用此数据源 . 您可以通过创建 `BatchConfigurer` 接口的自定义实现来自定义任何这些bean . 通常，扩展 `DefaultBatchConfigurer` （如果未找到 `BatchConfigurer` 则提供）并覆盖所需的吸气剂就足够了 . 但是，可能需要从头开始实现自己的 . 以下示例显示如何提供自定义事务管理器：


```java
@Bean
public BatchConfigurer batchConfigurer() {
        return new DefaultBatchConfigurer() {
                @Override
                public PlatformTransactionManager getTransactionManager() {
                        return new MyTransactionManager();
                }
        };
}
```



> 

只有一个配置类需要 `@EnableBatchProcessing` 注释 . 一旦你有一个用它注释的类，你将拥有以上所有可用的 . 

使用基本配置后，用户可以使用提供的构建器工厂来配置作业 . 以下是通过 `JobBuilderFactory` 和 `StepBuilderFactory` 配置的两步作业的示例 . 


```java
@Configuration
@EnableBatchProcessing
@Import(DataSourceConfiguration.class)
public class AppConfig {

    @Autowired
    private JobBuilderFactory jobs;

    @Autowired
    private StepBuilderFactory steps;

    @Bean
    public Job job(@Qualifier("step1") Step step1, @Qualifier("step2") Step step2) {
        return jobs.get("myJob").start(step1).next(step2).build();
    }

    @Bean
    protected Step step1(ItemReader<Person> reader,
                         ItemProcessor<Person, Person> processor,
                         ItemWriter<Person> writer) {
        return steps.get("step1")
            .<Person, Person> chunk(10)
            .reader(reader)
            .processor(processor)
            .writer(writer)
            .build();
    }

    @Bean
    protected Step step2(Tasklet tasklet) {
        return steps.get("step2")
            .tasklet(tasklet)
            .build();
    }
}
```



### 1.3.配置JobRepository

使用 `@EnableBatchProcessing` 时，为您提供开箱即用的 `JobRepository`  . 本节介绍如何配置您自己的 . 

如前所述，[JobRepository](#configureJob)用于Spring Batch中各种持久域对象的基本CRUD操作，例如 `JobExecution` 和 `StepExecution`  . 它是许多主要框架功能所必需的，例如 `JobLauncher` ， `Job` 和 `Step` . 

批处理命名空间抽象出 `JobRepository` 实现及其协作者的许多实现细节 . 但是，仍有一些配置选项可用：

XML配置


```xml
<job-repository id="jobRepository"
    data-source="dataSource"
    transaction-manager="transactionManager"
    isolation-level-for-create="SERIALIZABLE"
    table-prefix="BATCH_"
        max-varchar-length="1000"/>
```


除id之外，不需要上面列出的任何配置选项 . 如果未设置，将使用上面显示的默认值 . 出于意识目的，它们在上面显示 .   `max-varchar-length` 默认为2500，这是[sample schema scripts](schema-appendix.html#metaDataSchemaOverview)中长 `VARCHAR` 列的长度

使用java配置时，会为您提供 `JobRepository`  . 如果提供 `DataSource` ，则提供基于JDBC的一个，如果没有，则基于 `Map`  . 但是，您可以通过 `BatchConfigurer` 接口的实现自定义 `JobRepository` 的配置 . 

Java配置


```java
...
// This would reside in your BatchConfigurer implementation
@Override
protected JobRepository createJobRepository() throws Exception {
    JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
    factory.setDataSource(dataSource);
    factory.setTransactionManager(transactionManager);
    factory.setIsolationLevelForCreate("ISOLATION_SERIALIZABLE");
    factory.setTablePrefix("BATCH_");
    factory.setMaxVarCharLength(1000);
    return factory.getObject();
}
...
```


除dataSource和之外，不需要上面列出的任何配置选项transactionManager的 . 如果未设置，将使用上面显示的默认值 . 出于意识目的，它们在上面显示 .  max varchar length默认为2500，这是[sample schema scripts](schema-appendix.html#metaDataSchemaOverview)中长 `VARCHAR` 列的长度


#### 1.3.1. JobRepository的事务配置

如果使用命名空间或提供的 `FactoryBean` ，将在存储库周围自动创建事务建议 . 这是为了确保批处理元数据（包括失败后重新启动所需的状态）正确保留 . 如果存储库方法不是事务性的，则框架的行为没有很好地定义 .   `create*` 方法属性中的隔离级别是单独指定的，以确保在启动作业时，如果两个进程同时尝试启动同一作业，则只有一个成功 . 该方法的默认隔离级别是SERIALIZABLE，这是非常激进的：READ_COMMITTED也可以正常工作;如果两个进程不可能以这种方式发生冲突，READ_UNCOMMITTED就没问题了 . 但是，由于对 `create*` 方法的调用非常短，因此只要数据库平台支持SERIALIZED，就不太可能导致问题 . 但是，这可以被覆盖：

XML配置


```xml
<job-repository id="jobRepository"
                isolation-level-for-create="REPEATABLE_READ" />
```


Java配置


```java
// This would reside in your BatchConfigurer implementation
@Override
protected JobRepository createJobRepository() throws Exception {
    JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
    factory.setDataSource(dataSource);
    factory.setTransactionManager(transactionManager);
    factory.setIsolationLevelForCreate("ISOLATION_REPEATABLE_READ");
    return factory.getObject();
}
```


如果未使用命名空间或工厂bean，则使用AOP配置存储库的事务行为也很重要：

XML配置


```xml
<aop:config>
    <aop:advisor
           pointcut="execution(* org.springframework.batch.core..*Repository+.*(..))"/>
    <advice-ref="txAdvice" />
</aop:config>

<tx:advice id="txAdvice" transaction-manager="transactionManager">
    <tx:attributes>
        <tx:method name="*" />
    </tx:attributes>
</tx:advice>
```


这个片段可以按原样使用，几乎没有变化 . 还要记住包含适当的命名空间声明，并确保spring-tx和spring-aop（或整个spring）都在类路径中 . 

Java配置


```java
@Bean
public TransactionProxyFactoryBean baseProxy() {
        TransactionProxyFactoryBean transactionProxyFactoryBean = new TransactionProxyFactoryBean();
        Properties transactionAttributes = new Properties();
        transactionAttributes.setProperty("*", "PROPAGATION_REQUIRED");
        transactionProxyFactoryBean.setTransactionAttributes(transactionAttributes);
        transactionProxyFactoryBean.setTarget(jobRepository());
        transactionProxyFactoryBean.setTransactionManager(transactionManager());
        return transactionProxyFactoryBean;
}
```



#### 1.3.2.更改表格前缀

 `JobRepository` 的另一个可修改属性是元数据表的表前缀 . 默认情况下，它们都以BATCH_开头 .  BATCH_JOB_EXECUTION和BATCH_STEP_EXECUTION是两个例子 . 但是，有可能会修改此前缀 . 如果需要在模式名称前添加模式名称，或者在同一模式中需要多组元数据表，则需要更改表前缀：

XML配置


```xml
<job-repository id="jobRepository"
                table-prefix="SYSTEM.TEST_" />
```


Java配置


```java
// This would reside in your BatchConfigurer implementation
@Override
protected JobRepository createJobRepository() throws Exception {
    JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
    factory.setDataSource(dataSource);
    factory.setTransactionManager(transactionManager);
    factory.setTablePrefix("SYSTEM.TEST_");
    return factory.getObject();
}
```


鉴于上述更改，对元数据表的每个查询都将以“SYSTEM.TEST_”作为前缀 .  BATCH_JOB_EXECUTION将被称为SYSTEM.TEST_JOB_EXECUTION . 


> 

只有表前缀是可配置的 . 表和列名称不是 . 


#### 1.3.3.内存存储库

在某些情况下，您可能不希望将域对象持久保存到数据库中 . 一个原因可能是速度;在每个提交点存储域对象需要额外的时间 . 另一个原因可能是您不需要为特定工作保留状态 . 因此，Spring批处理提供了作业存储库的内存中Map版本：

XML配置


```xml
<bean id="jobRepository"
  class="org.springframework.batch.core.repository.support.MapJobRepositoryFactoryBean">
    <property name="transactionManager" ref="transactionManager"/>
</bean>
```


Java配置


```java
// This would reside in your BatchConfigurer implementation
@Override
protected JobRepository createJobRepository() throws Exception {
    JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
    factory.setDataSource(dataSource);
    factory.setTransactionManager(transactionManager);
    factory.setIsolationLevelForCreate("ISOLATION_REPEATABLE_READ");
    return factory.getObject();
}
```


请注意，内存存储库是易失性的，因此不允许在JVM实例之间重新启动 . 它也不能保证同时启动具有相同参数的两个作业实例，并且不适合在多线程作业或本地分区 `Step` 中使用 . 因此，只要您需要这些功能，请使用存储库的数据库版本 . 

但是它确实需要定义事务管理器，因为存储库中存在回滚语义，并且因为业务逻辑可能仍然是事务性的（例如RDBMS访问） . 出于测试目的，许多人发现 `ResourcelessTransactionManager` 很有用 . 


#### 1.3.4.存储库中的非标准数据库类型

如果您使用的数据库平台不在受支持的平台列表中，那么如果SQL变量足够接近，您可以使用其中一种受支持的类型 . 为此，您可以使用原始 `JobRepositoryFactoryBean` 而不是命名空间快捷方式，并使用它将数据库类型设置为最接近的匹配：

XML配置


```xml
<bean id="jobRepository" class="org...JobRepositoryFactoryBean">
    <property name="databaseType" value="db2"/>
    <property name="dataSource" ref="dataSource"/>
</bean>
```


Java配置


```java
// This would reside in your BatchConfigurer implementation
@Override
protected JobRepository createJobRepository() throws Exception {
    JobRepositoryFactoryBean factory = new JobRepositoryFactoryBean();
    factory.setDataSource(dataSource);
    factory.setDatabaseType("db2");
    factory.setTransactionManager(transactionManager);
    return factory.getObject();
}
```


（如果未指定， `JobRepositoryFactoryBean` 会尝试从 `DataSource` 自动检测数据库类型 . ）平台之间的主要区别主要在于增加主键的策略，因此通常可能需要覆盖 `incrementerFactory`  （使用Spring Framework中的一个标准实现） . 

如果即使这样做不起作用，或者你没有使用RDBMS，那么唯一的选择可能是实现 `SimpleJobRepository` 所依赖的各种 `Dao` 接口，并以正常的Spring方式手动连接一个接口 . 


### 1.4.配置JobLauncher

使用 `@EnableBatchProcessing` 时，为您提供开箱即用的 `JobRegistry`  . 本节介绍如何配置您自己的 . 

 `JobLauncher` 接口的最基本实现是 `SimpleJobLauncher`  . 它唯一需要的依赖是 `JobRepository` ，以获得执行：

XML配置


```xml
<bean id="jobLauncher"
      class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
    <property name="jobRepository" ref="jobRepository" />
</bean>
```


Java配置


```java
...
// This would reside in your BatchConfigurer implementation
@Override
protected JobLauncher createJobLauncher() throws Exception {
        SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
        jobLauncher.setJobRepository(jobRepository);
        jobLauncher.afterPropertiesSet();
        return jobLauncher;
}
...
```


一旦获得[JobExecution](domain.html#domainLanguageOfBatch)，就会传递给执行Job的方法，最终将 `JobExecution` 返回给调用者：


![Job Launcher Sequence](https://www.docs4dev.com/images/06d1d7db-4fd7-4994-a5dd-58ddddbd6186.png)


图2.作业启动程序序列

序列很简单，从调度程序启动时效果很好 . 但是，尝试从HTTP请求启动时会出现问题 . 在这种情况下，启动需要异步完成，以便 `SimpleJobLauncher` 立即返回其调用者 . 这是因为在长时间运行的进程（如批处理）所需的时间内保持HTTP请求打开是不好的做法 . 示例序列如下：


![Async Job Launcher Sequence](https://www.docs4dev.com/images/d0bc7142-697d-4a28-a1f4-3514ad184c1f.png)


图3.异步作业启动程序序列

通过配置 `TaskExecutor` ，可以轻松配置 `SimpleJobLauncher` 以允许此方案：

XML配置


```xml
<bean id="jobLauncher"
      class="org.springframework.batch.core.launch.support.SimpleJobLauncher">
    <property name="jobRepository" ref="jobRepository" />
    <property name="taskExecutor">
        <bean class="org.springframework.core.task.SimpleAsyncTaskExecutor" />
    </property>
</bean>
```


Java配置


```java
@Bean
public JobLauncher jobLauncher() {
        SimpleJobLauncher jobLauncher = new SimpleJobLauncher();
        jobLauncher.setJobRepository(jobRepository());
        jobLauncher.setTaskExecutor(new SimpleAsyncTaskExecutor());
        jobLauncher.afterPropertiesSet();
        return jobLauncher;
}
```


spring  `TaskExecutor` 接口的任何实现都可用于控制作业异步执行的方式 . 


### 1.5.正在运行一份工作

至少，启动批处理作业需要两件事： `Job` 要启动， `JobLauncher`  . 两者都可以包含在相同的上下文或不同的上下文中 . 例如，如果从命令行启动作业，则将为每个作业实例化一个新JVM，因此每个作业都将拥有自己的 `JobLauncher`  . 但是，如果在 `HttpRequest` 范围内的Web容器内运行，则通常会有一个 `JobLauncher` （配置为异步作业启动），将调用多个请求以启动其作业 . 


#### 1.5.1.从命令行运行作业

对于想要从企业调度程序运行其作业的用户，命令行是主要接口 . 这是因为大多数调度程序（Quartz除外，除非使用NativeJob）直接使用操作系统进程，主要是使用shell脚本启动 . 除了shell脚本之外，还有许多方法可以启动Java进程，例如Perl，Ruby，甚至是诸如ant或maven之类的“构建工具” . 但是，由于大多数人都熟悉shell脚本，因此本示例将重点介绍它们 . 


##### The CommandLineJobRunner

因为启动作业的脚本必须启动Java虚拟机，所以需要一个带有main方法的类作为主要入口点 .  Spring Batch提供了一个实现此目的的实现： `CommandLineJobRunner`  . 重要的是要注意，这只是引导应用程序的一种方法，但是有很多方法可以启动Java进程，而且这个类绝不应该被视为确定的 .   `CommandLineJobRunner` 执行四项任务：


- 
加载适当的 `ApplicationContext` 


- 
将命令行参数解析为 `JobParameters` 


- 
根据参数找到适当的作业


- 
使用应用程序上下文中提供的 `JobLauncher` 来启动作业 . 

所有这些任务都只使用传入的参数完成 . 以下是必需的参数：

||
| jobPath |将用于创建 `ApplicationContext` 的XML文件的位置 . 此文件应包含运行完整作业所需的所有内容
| ---- | ---- |
| jobName |要运行的作业的名称 .  |

这些参数必须首先传入路径，然后传递名称第二个 . 这些之后的所有参数都被认为是 `JobParameters` ，并且必须采用'name = value'的格式：


```xml
<bash$ java CommandLineJobRunner endOfDayJob.xml endOfDay schedule.date(date)=2007/05/05
```



```xml
<bash$ java CommandLineJobRunner io.spring.EndOfDayJobConfiguration endOfDay schedule.date(date)=2007/05/05
```


在大多数情况下，您可能希望使用清单在jar中声明主类，但为简单起见，该类是直接使用的 . 此示例使用[domainLanguageOfBatch](domain.html#domainLanguageOfBatch)中的相同“EndOfDay”示例 . 第一个参数是'endOfDayJob.xml'，它是包含Job的Spring  `ApplicationContext`  . 第二个参数'endOfDay'代表作业名称 . 最后一个参数'schedule.date（date）= 2007/05/05'将转换为 `JobParameters` . 下面是XML配置的一个示例：


```xml
<job id="endOfDay">
    <step id="step1" parent="simpleStep" />
</job>

<!-- Launcher details removed for clarity -->
<beans:bean id="jobLauncher"
         class="org.springframework.batch.core.launch.support.SimpleJobLauncher" />
```


在大多数情况下，您可能希望使用清单在jar中声明主类，但为简单起见，该类是直接使用的 . 此示例使用[domainLanguageOfBatch](domain.html#domainLanguageOfBatch)中的相同“EndOfDay”示例 . 第一个参数是'io.spring.EndOfDayJobConfiguration'，它是包含Job的配置类的完全限定类名 . 第二个参数'endOfDay'代表作业名称 . 最后一个参数'schedule.date（date）= 2007/05 05'将转换为JobParameters . java配置的一个例子如下：


```java
@Configuration
@EnableBatchProcessing
public class EndOfDayJobConfiguration {

    @Autowired
    private JobBuilderFactory jobBuilderFactory;

    @Autowired
    private StepBuilderFactory stepBuilderFactory;

    @Bean
    public Job endOfDay() {
        return this.jobBuilderFactory.get("endOfDay")
                                    .start(step1())
                                    .build();
    }

    @Bean
    public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                    .tasklet((contribution, chunkContext) -> null)
                                    .build();
    }
}
```


这个例子过于简单了，因为在Spring Batch中运行批处理作业有很多要求，但它用于显示 `CommandLineJobRunner` 的两个主要要求： `Job` 和 `JobLauncher` 


##### ExitCodes

从命令行启动批处理作业时，通常使用企业调度程序 . 大多数调度程序都相当愚蠢，只能在进程级别工作 . 这意味着他们只知道某些操作系统进程，例如他们正在调用的shell脚本 . 在这种情况下，通过返回代码与作业成功或失败进行通信的唯一方法 . 返回码是由进程返回到调度程序的数字，指示运行的结果 . 在最简单的情况下：0表示成功，1表示失败 . 但是，可能存在更复杂的情况：如果作业A返回4启动作业B，并且如果它返回5启动作业C.这种类型的行为是在调度程序级别配置的，但重要的是处理框架如Spring Batch提供了一种返回特定批处理作业的“退出代码”的数字表示的方法 . 在Spring Batch中，它封装在 `ExitStatus` 中，第5章将对此进行更详细的介绍 . 为了讨论退出代码，唯一需要知道的是 `ExitStatus` 具有由框架设置的退出代码属性（或者开发人员）并作为 `JobLauncher` 返回的 `JobExecution` 的一部分返回 .   `CommandLineJobRunner` 使用 `ExitCodeMapper` 接口将此字符串值转换为数字：


```java
public interface ExitCodeMapper {

    public int intValue(String exitCode);

}
```


 `ExitCodeMapper` 的基本 Contract 是，给定字符串退出代码，将返回数字表示 . 作业运行程序使用的默认实现是 `SimpleJvmExitCodeMapper` ，它返回0表示完成，1表示一般错误，2表示任何作业运行程序错误，例如无法在提供的上下文中找到 `Job`  . 如果需要比上面3个值更复杂的东西，则必须提供 `ExitCodeMapper` 接口的自定义实现 . 因为 `CommandLineJobRunner` 是创建 `ApplicationContext` 的类，因此无法“连接在一起”，所以需要覆盖任何需要覆盖的值都必须自动装配 . 这意味着如果在 `BeanFactory` 中找到 `ExitCodeMapper` 的实现，则会在创建上下文后将其注入到运行程序中 . 为提供自己的 `ExitCodeMapper` 而需要做的就是将实现声明为根级别bean并确保它是由运行者加载的 `ApplicationContext` 的一部分 . 


#### 1.5.2.从Web容器中运行作业

历史上，如上所述，已从命令行启动诸如批处理作业的离线处理 . 但是，在很多情况下，从 `HttpRequest` 启动是更好的选择 . 许多此类用例包括报告，临时作业运行和Web应用程序支持 . 因为按定义批处理作业长时间运行，所以最重要的问题是确保以异步方式启动作业：


![Async Job Launcher Sequence from web container](https://www.docs4dev.com/images/5c86b0b6-6fde-4f09-bf2b-5a6bb694c8dd.png)


图4. Web容器中的异步作业启动器序列

在这种情况下，控制器是一个Spring MVC控制器 . 有关Spring MVC的更多信息，请访问：[https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc](https://docs.spring.io/spring/docs/current/spring-framework-reference/web.html#mvc) . 控制器使用 `JobLauncher` 启动 `Job` ，该 `JobLauncher` 已配置为启动[asynchronously](#runningJobsFromWebContainer)，立即返回 `JobExecution`  .  `Job` 可能仍在运行，但是，这种非阻塞行为允许控制器立即返回，这在处理 `HttpRequest` 时是必需的 . 一个例子如下：


```java
@Controller
public class JobLauncherController {

    @Autowired
    JobLauncher jobLauncher;

    @Autowired
    Job job;

    @RequestMapping("/jobLauncher.html")
    public void handle() throws Exception{
        jobLauncher.run(job, new JobParameters());
    }
}
```



### 1.6.高级元数据使用

到目前为止，已经讨论了 `JobLauncher` 和 `JobRepository` 接口 . 它们共同代表了简单的作业启动和批处理域对象的基本CRUD操作：


![Job Repository](https://www.docs4dev.com/images/a512d764-ad5a-480e-beb8-f626b802e4dd.png)


图5.作业存储库

 `JobLauncher` 使用 `JobRepository` 创建新的 `JobExecution` 对象并运行它们 .   `Job` 和 `Step` 实现稍后在作业运行期间使用相同的 `JobRepository` 进行相同执行的基本更新 . 基本操作足以满足简单场景，但在具有数百个批处理作业和复杂调度要求的大型批处理环境中，需要更高级的元数据访问：


![Job Repository Advanced](https://www.docs4dev.com/images/3bb32167-14af-470a-a736-647752c2f22e.png)


图6.高级作业存储库访问

 `JobExplorer` 和 `JobOperator` 接口（将在下面讨论）添加了用于查询和控制元数据的附加功能 . 


#### 1.6.1.查询存储库

任何高级功能之前的最基本需求是能够查询存储库以查找现有执行 . 此功能由 `JobExplorer` 接口提供：


```java
public interface JobExplorer {

    List<JobInstance> getJobInstances(String jobName, int start, int count);

    JobExecution getJobExecution(Long executionId);

    StepExecution getStepExecution(Long jobExecutionId, Long stepExecutionId);

    JobInstance getJobInstance(Long instanceId);

    List<JobExecution> getJobExecutions(JobInstance jobInstance);

    Set<JobExecution> findRunningJobExecutions(String jobName);
}
```


从上面的方法签名可以看出， `JobExplorer` 是 `JobRepository` 的只读版本，与 `JobRepository` 类似，可以通过工厂bean轻松配置：

XML配置


```xml
<bean id="jobExplorer" class="org.spr...JobExplorerFactoryBean"
      p:dataSource-ref="dataSource" />
```


Java配置


```java
...
// This would reside in your BatchConfigurer implementation
@Override
public JobExplorer getJobExplorer() throws Exception {
        JobExplorerFactoryBean factoryBean = new JobExplorerFactoryBean();
        factoryBean.setDataSource(this.dataSource);
        return factoryBean.getObject();
}
...
```


[Earlier in this chapter](#repositoryTablePrefix)，有人提到可以修改 `JobRepository` 的表前缀以允许不同的版本或模式 . 因为 `JobExplorer` 正在使用相同的表，所以它也需要能够设置前缀：

XML配置


```xml
<bean id="jobExplorer" class="org.spr...JobExplorerFactoryBean"
                p:tablePrefix="SYSTEM."/>
```


Java配置


```java
...
// This would reside in your BatchConfigurer implementation
@Override
public JobExplorer getJobExplorer() throws Exception {
        JobExplorerFactoryBean factoryBean = new JobExplorerFactoryBean();
        factoryBean.setDataSource(this.dataSource);
        factoryBean.setTablePrefix("SYSTEM.");
        return factoryBean.getObject();
}
...
```



#### 1.6.2. JobRegistry

 `JobRegistry` （及其父接口 `JobLocator` ）不是必需的，但如果要跟踪上下文中可用的作业，它可能很有用 . 当在其他地方（例如在子环境中）创建作业时，它对于在应用程序上下文中集中收集作业也是有用的 . 自定义 `JobRegistry` 实现还可用于操作已注册作业的名称和其他属性 . 框架只提供了一个实现，它基于从作业名称到作业实例的简单映射 . 


```xml
<bean id="jobRegistry" class="org.springframework.batch.core.configuration.support.MapJobRegistry" />
```


使用 `@EnableBatchProcessing` 时，为您提供开箱即用的 `JobRegistry`  . 如果你想配置你的拥有：


```java
...
// This is already provided via the @EnableBatchProcessing but can be customized via
// overriding the getter in the SimpleBatchConfiguration
@Override
@Bean
public JobRegistry jobRegistry() throws Exception {
        return new MapJobRegistry();
}
...
```


有两种方法可以自动填充 `JobRegistry` ：使用bean后处理器和使用注册器生命周期组件 . 以下各节介绍了这两种机制 . 


##### JobRegistryBeanPostProcessor

这是一个bean后处理器，可以在创建时注册所有作业：

XML配置


```xml
<bean id="jobRegistryBeanPostProcessor" class="org.spr...JobRegistryBeanPostProcessor">
    <property name="jobRegistry" ref="jobRegistry"/>
</bean>
```


Java配置


```java
@Bean
public JobRegistryBeanPostProcessor jobRegistryBeanPostProcessor() {
    JobRegistryBeanPostProcessor postProcessor = new JobRegistryBeanPostProcessor();
    postProcessor.setJobRegistry(jobRegistry());
    return postProcessor;
}
```


尽管不是严格必要的，但是示例中的后处理器已经被赋予id，以便它可以被包括在子上下文中（例如，作为父bean定义）并且使得在那里创建的所有作业也被自动注册 . 


##### AutomaticJobRegistrar

这是一个生命周期组件，可以创建子上下文，并在创建这些上下文时注册它们 . 这样做的一个优点是，虽然子上下文中的作业名称仍然必须在注册表中是全局唯一的，但它们的依赖项可以具有"natural"名称 . 因此，例如，您可以创建一组XML配置文件，每个文件只有一个Job，但都具有不同的 `ItemReader` 定义，具有相同的bean名称，例如"reader" . 如果所有这些文件都被导入到同一个上下文中，那么读者定义会相互冲突并相互覆盖，但是使用自动注册器可以避免这种情况 . 这样可以更轻松地集成由应用程序的单独模块提供的作业 . 

XML配置


```xml
<bean class="org.spr...AutomaticJobRegistrar">
   <property name="applicationContextFactories">
      <bean class="org.spr...ClasspathXmlApplicationContextsFactoryBean">
         <property name="resources" value="classpath*:/config/job*.xml" />
      </bean>
   </property>
   <property name="jobLoader">
      <bean class="org.spr...DefaultJobLoader">
         <property name="jobRegistry" ref="jobRegistry" />
      </bean>
   </property>
</bean>
```


Java配置


```java
@Bean
public AutomaticJobRegistrar registrar() {

    AutomaticJobRegistrar registrar = new AutomaticJobRegistrar();
    registrar.setJobLoader(jobLoader());
    registrar.setApplicationContextFactories(applicationContextFactories());
    registrar.afterPropertiesSet();
    return registrar;

}
```


注册器有两个必需属性，一个是 `ApplicationContextFactory` 的数组（这里是从方便的工厂bean创建的），另一个是 `JobLoader`  .   `JobLoader` 负责管理子上下文的生命周期并在 `JobRegistry` 中注册作业 . 

 `ApplicationContextFactory` 负责创建子上下文，最常见的用法如上所述使用 `ClassPathXmlApplicationContextFactory`  . 此工厂的一个功能是默认情况下，它会将一些配置从父上下文复制到子项 . 因此，例如，您不必在子节点中重新定义 `PropertyPlaceholderConfigurer` 或AOP配置，如果它应该与父节点相同 . 

如果需要， `AutomaticJobRegistrar` 可以与 `JobRegistryBeanPostProcessor` 一起使用（只要使用 `DefaultJobLoader` ） . 例如，如果在主要父上下文中以及子位置中定义了作业，则可能需要这样做 . 


#### 1.6.3. JobOperator

如前所述， `JobRepository` 对元数据提供CRUD操作， `JobExplorer` 对元数据提供只读操作 . 但是，这些操作在一起使用以执行常见监视任务（例如停止，重新启动或汇总作业）时非常有用，这通常由批处理操作员执行 .  Spring Batch通过 `JobOperator` 接口提供这些类型的操作：


```java
public interface JobOperator {

    List<Long> getExecutions(long instanceId) throws NoSuchJobInstanceException;

    List<Long> getJobInstances(String jobName, int start, int count)
          throws NoSuchJobException;

    Set<Long> getRunningExecutions(String jobName) throws NoSuchJobException;

    String getParameters(long executionId) throws NoSuchJobExecutionException;

    Long start(String jobName, String parameters)
          throws NoSuchJobException, JobInstanceAlreadyExistsException;

    Long restart(long executionId)
          throws JobInstanceAlreadyCompleteException, NoSuchJobExecutionException,
                  NoSuchJobException, JobRestartException;

    Long startNextInstance(String jobName)
          throws NoSuchJobException, JobParametersNotFoundException, JobRestartException,
                 JobExecutionAlreadyRunningException, JobInstanceAlreadyCompleteException;

    boolean stop(long executionId)
          throws NoSuchJobExecutionException, JobExecutionNotRunningException;

    String getSummary(long executionId) throws NoSuchJobExecutionException;

    Map<Long, String> getStepExecutionSummaries(long executionId)
          throws NoSuchJobExecutionException;

    Set<String> getJobNames();

}
```


上述操作表示来自许多不同接口的方法，例如 `JobLauncher` ， `JobRepository` ， `JobExplorer` 和 `JobRegistry`  . 因此， `JobOperator` ， `SimpleJobOperator` 的提供实现具有许多依赖关系：


```xml
<bean id="jobOperator" class="org.spr...SimpleJobOperator">
    <property name="jobExplorer">
        <bean class="org.spr...JobExplorerFactoryBean">
            <property name="dataSource" ref="dataSource" />
        </bean>
    </property>
    <property name="jobRepository" ref="jobRepository" />
    <property name="jobRegistry" ref="jobRegistry" />
    <property name="jobLauncher" ref="jobLauncher" />
</bean>
```



```java
/**
  * All injected dependencies for this bean are provided by the @EnableBatchProcessing
  * infrastructure out of the box.
  */
 @Bean
 public SimpleJobOperator jobOperator(JobExplorer jobExplorer,
                                JobRepository jobRepository,
                                JobRegistry jobRegistry) {

        SimpleJobOperator jobOperator = new SimpleJobOperator();

        jobOperator.setJobExplorer(jobExplorer);
        jobOperator.setJobRepository(jobRepository);
        jobOperator.setJobRegistry(jobRegistry);
        jobOperator.setJobLauncher(jobLauncher);

        return jobOperator;
 }
```



> 

如果在作业存储库上设置表前缀，请不要忘记在作业资源管理器上设置它 . 


#### 1.6.4. JobParametersIncrementer

 `JobOperator` 上的大多数方法都是不言自明的，可以在[javadoc of the interface](https://docs.spring.io/spring-batch/apidocs/org/springframework/batch/core/launch/JobOperator.html)上找到更详细的解释 . 但是， `startNextInstance` 方法值得注意 . 此方法将始终启动Job的新实例 . 如果 `JobExecution` 中存在严重问题并且需要从头开始重新执行作业，这可能非常有用 . 与 `JobLauncher` 不同，它需要一个新的 `JobParameters` 对象，如果参数与任何先前的参数集不同，将触发新的 `JobInstance` ， `startNextInstance` 方法将使用 `JobParametersIncrementer` 绑定到 `Job` 强制 `Job` 到新实例：


```java
public interface JobParametersIncrementer {

    JobParameters getNext(JobParameters parameters);

}
```


 `JobParametersIncrementer` 的 Contract 是，给定一个[JobParameters](#jobParameters)对象，它将通过递增它可能包含的任何必要值来返回'下一个'JobParameters对象 . 这个策略很有用，因为框架无法知道对 `JobParameters` 的哪些更改使其成为“下一个”实例 . 例如，如果 `JobParameters` 中的唯一值是日期，并且应该创建下一个实例，那么该值是否应该增加一天？或一周（如果工作是每周一次）？对于有助于识别作业的任何数值，也可以这样说，如下所示：


```java
public class SampleIncrementer implements JobParametersIncrementer {

    public JobParameters getNext(JobParameters parameters) {
        if (parameters==null || parameters.isEmpty()) {
            return new JobParametersBuilder().addLong("run.id", 1L).toJobParameters();
        }
        long id = parameters.getLong("run.id",1L) + 1;
        return new JobParametersBuilder().addLong("run.id", id).toJobParameters();
    }
}
```


在此示例中，使用键“run.id”的值用于区分 `JobInstances`  . 如果传入的 `JobParameters` 为null，则可以假定 `Job` 之前从未运行过，因此可以返回其初始状态 . 但是，如果不是，则获取旧值，递增1并返回 . 

增量器可以通过命名空间中的“incrementmenter”属性与 `Job` 关联：


```xml
<job id="footballJob" incrementer="sampleIncrementer">
    ...
</job>
```


增量器可以通过构建器中提供的 `incrementer` 方法与“作业”关联：


```java
@Bean
public Job footballJob() {
    return this.jobBuilderFactory.get("footballJob")
                                     .incrementer(sampleIncrementer())
                                     ...
                     .build();
}
```



#### 1.6.5.停止工作

 `JobOperator` 最常见的用例之一是优雅地停止作业：


```java
Set<Long> executions = jobOperator.getRunningExecutions("sampleJob");
jobOperator.stop(executions.iterator().next());
```


关机不是立即，因为没有办法强制立即关闭，特别是如果执行当前是框架无法控制的开发人员代码，例如业务服务 . 但是，只要控件返回到框架，它就会将当前 `StepExecution` 的状态设置为 `BatchStatus.STOPPED` ，保存它，然后在完成之前对 `JobExecution` 执行相同操作 . 


#### 1.6.6.中止工作

可以重新启动 `FAILED` 的作业执行（如果 `Job` 可重新启动） . 框架不会重新启动状态为 `ABANDONED` 的作业执行 .   `ABANDONED` 状态也用于步骤执行，以在重新启动的作业执行中将它们标记为可跳过：如果作业正在执行并遇到在先前失败的作业执行中标记为 `ABANDONED` 的步骤，则它将继续执行下一步骤（由作业流程定义和步骤执行退出状态确定） . 

如果该过程死亡（ `"kill -9"` 或服务器故障），该作业当然没有运行，但 `JobRepository` 无法知道，因为在该过程死亡之前没有人告诉它 . 您必须手动告诉它您知道执行失败或应该被视为中止（将其状态更改为 `FAILED` 或 `ABANDONED` ） - 这是一个业务决策，并且无法自动执行 . 如果不可重新启动，或者您知道重启数据有效，则仅将状态更改为 `FAILED`  .  Spring Batch Admin  `JobService` 中有一个实用程序可以中止作业执行 .
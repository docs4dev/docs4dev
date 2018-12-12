## 1.配置步骤

XML Java

正如[the domain chapter](domain.html#domainLanguageOfBatch)中所讨论的， `Step` 是一个域对象，它封装了批处理作业的独立顺序阶段，并包含定义和控制实际批处理所需的所有信息 . 这是一个必然模糊的描述，因为任何给定 `Step` 的内容由开发人员自行决定编写 `Job`  .  `Step` 可以像开发者所希望的那样简单或复杂 . 一个简单的 `Step` 可能会将文件中的数据加载到数据库中，几乎不需要代码（取决于所使用的实现） . 更复杂的 `Step` 可能具有复杂的业务规则，这些规则作为处理的一部分应用，如下图所示：


![Step](https://www.docs4dev.com/images/b8aa8255-d865-4ca3-aeb2-81e752214494.png)


图1.步骤


### 1.1.面向块的处理

Spring Batch在其最常见的实现中使用“Chunk-oriented”处理样式 . 面向块的处理是指一次读取一个数据并创建在事务边界内写出的“块” . 从 `ItemReader` 读取一个项目，传递给 `ItemProcessor` ，并进行聚合 . 一旦读取的项目数等于提交间隔，整个块将由 `ItemWriter` 写出，然后提交事务 . 下图显示了该过程：


![Chunk Oriented Processing](https://www.docs4dev.com/images/f4e80af7-41c9-4064-b8f0-8a84862ec800.png)


图2.面向块的处理

以下代码显示了相同的概念：


```java
List items = new Arraylist();
for(int i = 0; i < commitInterval; i++){
    Object item = itemReader.read()
    Object processedItem = itemProcessor.process(item);
    items.add(processedItem);
}
itemWriter.write(items);
```



#### 1.1.1.配置步骤

尽管 `Step` 所需的依赖项列表相对较短，但它是一个极其复杂的类，可能包含许多协作者 . 

为了简化配置，可以使用Spring Batch命名空间，如以下示例所示：

XML配置


```xml
<job id="sampleJob" job-repository="jobRepository">
    <step id="step1">
        <tasklet transaction-manager="transactionManager">
            <chunk reader="itemReader" writer="itemWriter" commit-interval="10"/>
        </tasklet>
    </step>
</job>
```


使用Java配置时，可以使用Spring Batch构建器，如以下示例所示：

Java配置


```java
/**
 * Note the JobRepository is typically autowired in and not needed to be explicitly
 * configured
 */
@Bean
public Job sampleJob(JobRepository jobRepository, Step sampleStep) {
    return this.jobBuilderFactory.get("sampleJob")
                            .repository(jobRepository)
                .start(sampleStep)
                .build();
}

/**
 * Note the TransactionManager is typically autowired in and not needed to be explicitly
 * configured
 */
@Bean
public Step sampleStep(PlatformTransactionManager transactionManager) {
        return this.stepBuilderFactory.get("sampleStep")
                                .transactionManager(transactionManager)
                                .<String, String>chunk(10)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .build();
}
```


上面的配置包括创建面向项目的步骤所需的唯一依赖项：


- 
 `reader` ：提供处理项目的 `ItemReader`  . 


- 
 `writer` ： `ItemWriter` 处理 `ItemReader` 提供的项目 . 


- 
 `transaction-manager` ：Spring的 `PlatformTransactionManager` 开始并在处理期间提交事务 . 


- 
 `transactionManager` ：Spring的 `PlatformTransactionManager` 开始并在处理期间提交事务 . 


- 
 `job-repository` ： `JobRepository` 在处理期间（就在提交之前）定期存储 `StepExecution` 和 `ExecutionContext`  . 对于内联<step />（在<job />中定义的一个），它是<job />元素的属性 . 对于独立步骤，它被定义为<tasklet />的属性 . 


- 
 `repository` ： `JobRepository` 在处理期间（就在提交之前）定期存储 `StepExecution` 和 `ExecutionContext`  . 


- 
 `commit-interval` ：提交事务之前要处理的项目数 . 


- 
 `chunk` ：表示这是基于项目的步骤以及在提交事务之前要处理的项目数 . 

应该注意 `job-repository` 默认为 `jobRepository` ， `transaction-manager` 默认为 `transactionManger`  . 此外， `ItemProcessor` 是可选的，因为该项可以直接从阅读器传递给编写器 . 

应该注意的是 `repository` 默认为 `jobRepository` ， `transactionManager` 默认为 `transactionManger` （所有这些都是通过 `@EnableBatchProcessing` 的基础设施提供的） . 此外， `ItemProcessor` 是可选的，因为该项可以直接从阅读器传递给编写器 . 


#### 1.1.2.从父步骤继承

如果一组 `Steps` 共享相似的配置，那么定义一个"parent"  `Step` 可能会有所帮助具体 `Steps` 可以继承属性 . 与Java中的类继承类似，"child"  `Step` 将其元素和属性与父元素组合在一起 . 孩子也会覆盖任何父母的 `Steps`  . 

在以下示例中， `Step` ，"concreteStep1"继承自"parentStep" . 它使用'itemReader'，'itemProcessor'，'itemWriter'， `startLimit=5` 和 `allowStartIfComplete=true` 进行实例化 . 此外， `commitInterval` 为'5'，因为它被"concreteStep1" _13350重写，如以下示例所示：


```xml
<step id="parentStep">
    <tasklet allow-start-if-complete="true">
        <chunk reader="itemReader" writer="itemWriter" commit-interval="10"/>
    </tasklet>
</step>

<step id="concreteStep1" parent="parentStep">
    <tasklet start-limit="5">
        <chunk processor="itemProcessor" commit-interval="5"/>
    </tasklet>
</step>
```


作业元素中的步骤仍然需要 `id` 属性 . 这有两个原因：


- 
 `id` 在持久化 `StepExecution` 时用作步骤名称 . 如果在作业中的多个步骤中引用了相同的独立步骤，则会发生错误 . 


- 
创建作业流时，如本章后面所述， `next` 属性应该引用流程中的步骤，而不是独立步骤 . 


##### Abstract Step

有时，可能需要定义不是完整 `Step` 配置的父 `Step`  . 例如，如果 `reader` ， `writer` 和 `tasklet` 属性不在 `Step` 配置中，则初始化失败 . 如果必须在没有这些属性的情况下定义父级，则应使用 `abstract` 属性 .   `abstract`   `Step` 仅被扩展，从未实例化 . 

在以下示例中，如果未将 `Step`   `abstractParentStep` 声明为抽象，则不会将其实例化 .   `Step` ，"concreteStep2"具有'itemReader'，'itemWriter'和commit-interval = 10 . 


```xml
<step id="abstractParentStep" abstract="true">
    <tasklet>
        <chunk commit-interval="10"/>
    </tasklet>
</step>

<step id="concreteStep2" parent="abstractParentStep">
    <tasklet>
        <chunk reader="itemReader" writer="itemWriter"/>
    </tasklet>
</step>
```



##### Merging列表

 `Steps` 上的一些可配置元素是列表，例如 `<listeners/>` 元素 . 如果父级和子级 `Steps` 都声明 `<listeners/>` 元素，则子级列表将覆盖父级列表 . 为了允许子级向父级定义的列表添加其他侦听器，每个列表元素都具有 `merge` 属性 . 如果元素指定 `merge="true"` ，那么子列表将与父项组合，而不是覆盖它 . 

在以下示例中，使用两个侦听器创建 `Step`  "concreteStep3"： `listenerOne` 和 `listenerTwo` ：


```xml
<step id="listenersParentStep" abstract="true">
    <listeners>
        <listener ref="listenerOne"/>
    <listeners>
</step>

<step id="concreteStep3" parent="listenersParentStep">
    <tasklet>
        <chunk reader="itemReader" writer="itemWriter" commit-interval="5"/>
    </tasklet>
    <listeners merge="true">
        <listener ref="listenerTwo"/>
    <listeners>
</step>
```



#### 1.1.3.提交间隔

如前所述，一个步骤读入和写出项目，使用提供的 `PlatformTransactionManager` 定期提交 .   `commit-interval` 为1时，它会在写完每个项目后提交 . 这在许多情况下并不理想，因为开始和提交交易是昂贵的 . 理想情况下，最好在每个事务中处理尽可能多的项目，这完全取决于正在处理的数据类型和步骤与之交互的资源 . 因此，可以配置在提交中处理的项目数 . 以下示例显示 `step` ，其 `tasklet` 的 `commit-interval` 值为10 . 

XML配置


```xml
<job id="sampleJob">
    <step id="step1">
        <tasklet>
            <chunk reader="itemReader" writer="itemWriter" commit-interval="10"/>
        </tasklet>
    </step>
</job>
```


Java配置


```java
@Bean
public Job sampleJob() {
    return this.jobBuilderFactory.get("sampleJob")
                     .start(step1())
                     .end()
                     .build();
}

@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(10)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .build();
}
```


在前面的示例中，每个事务中处理10个项目 . 在处理开始时，开始交易 . 此外，每次在 `ItemReader` 上调用 `read` 时，计数器都会递增 . 当它达到10时，聚合项目列表将传递给 `ItemWriter` ，并提交事务 . 


#### 1.1.4.配置重启步骤

在“[Configuring and Running a Job](job.html#configureJob)”部分中，讨论了重新启动 `Job` . 重启对步骤有很多影响，因此可能需要一些特定的配置 . 

_0005设置开始限制

在许多情况下，您可能希望控制 `Step` 可以启动的次数 . 例如，可能需要配置特定的 `Step` ，以便它只运行一次，因为它会使某些必须手动修复的资源无效，然后才能再次运行 . 这可以在步骤级别进行配置，因为不同的步骤可能有不同的要求 . 只能执行一次的 `Step` 可以作为 `Job` 的一部分存在，可以无限运行 `Step`  . 以下代码片段显示了启动限制配置的示例：

XML配置


```xml
<step id="step1">
    <tasklet start-limit="1">
        <chunk reader="itemReader" writer="itemWriter" commit-interval="10"/>
    </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(10)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .startLimit(1)
                                .build();
}
```


上面的步骤只能运行一次 . 试图再次运行会导致 `StartLimitExceededException` 被抛出 . 请注意，起始限制的默认值为 `Integer.MAX_VALUE`  . 


##### 重新启动已完成的步骤

在可重新启动的作业的情况下，可能总是应该运行一个或多个步骤，无论它们是否第一次成功 . 一个示例可能是验证步骤或 `Step` ，它在处理之前清理资源 . 在重新启动的作业的正常处理期间，将跳过状态为“COMPLETED”的任何步骤，这意味着它已成功完成 . 将 `allow-start-if-complete` 设置为"true"将覆盖此步骤，以便始终运行该步骤，如以下示例所示：

XML配置


```xml
<step id="step1">
    <tasklet allow-start-if-complete="true">
        <chunk reader="itemReader" writer="itemWriter" commit-interval="10"/>
    </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(10)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .allowStartIfComplete(true)
                                .build();
}
```



##### Step重新启动配置示例

以下示例显示如何配置作业以使步骤可以重新启动：

XML配置


```xml
<job id="footballJob" restartable="true">
    <step id="playerload" next="gameLoad">
        <tasklet>
            <chunk reader="playerFileItemReader" writer="playerWriter"
                   commit-interval="10" />
        </tasklet>
    </step>
    <step id="gameLoad" next="playerSummarization">
        <tasklet allow-start-if-complete="true">
            <chunk reader="gameFileItemReader" writer="gameWriter"
                   commit-interval="10"/>
        </tasklet>
    </step>
    <step id="playerSummarization">
        <tasklet start-limit="2">
            <chunk reader="playerSummarizationSource" writer="summaryWriter"
                   commit-interval="10"/>
        </tasklet>
    </step>
</job>
```


Java的组态


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

@Bean
public Step playerLoad() {
        return this.stepBuilderFactory.get("playerLoad")
                        .<String, String>chunk(10)
                        .reader(playerFileItemReader())
                        .writer(playerWriter())
                        .build();
}

@Bean
public Step gameLoad() {
        return this.stepBuilderFactory.get("gameLoad")
                        .allowStartIfComplete(true)
                        .<String, String>chunk(10)
                        .reader(gameFileItemReader())
                        .writer(gameWriter())
                        .build();
}

@Bean
public Step playerSummarization() {
        return this.stepBuilderFactor.get("playerSummarization")
                        .startLimit(2)
                        .<String, String>chunk(10)
                        .reader(playerSummarizationSource())
                        .writer(summaryWriter())
                        .build();
}
```


前面的示例配置适用于加载有关足球比赛的信息并对其进行总结的作业 . 它包含三个步骤： `playerLoad` ， `gameLoad` 和 `playerSummarization`  .   `playerLoad` 步骤从平面文件加载播放器信息，而 `gameLoad` 步骤对游戏执行相同操作 . 最后一步， `playerSummarization` ，然后根据提供的游戏总结每个玩家的统计数据 . 假设 `playerLoad` 加载的文件只能加载一次，但 `gameLoad` 可以加载在特定目录中找到的任何游戏，在成功加载到数据库后删除它们 . 因此， `playerLoad` 步骤不包含其他配置 . 它可以启动任意次，如果完成，则跳过 . 但是，如果自上次运行以来添加了额外文件，则每次都需要运行 `gameLoad` 步骤 . 它将'allow-start-if-complete'设置为'true'以便始终启动 .  （假设加载的数据库表游戏上有一个过程指示器，以确保摘要步骤可以正确找到新游戏） . 摘要步骤是作业中最重要的步骤，配置为启动限制为2.这很有用，因为如果步骤不断失败，则会向控制作业执行的操作员返回一个新的退出代码，它可以在手动干预之前不要重新开始 . 


> 

此作业提供了此文档的示例，与示例项目中的 `footballJob` 不同 . 

本节的其余部分描述了 `footballJob` 示例的三次运行中发生的情况 . 

运行1：

playerLoad成功运行并完成，将400名玩家添加到'PLAYERS'表中 .  gameLoad运行并处理11个文件的游戏数据，将其内容加载到'GAMES'表中 .  playerSummarization开始处理并在5分钟后失败 . 

运行2：

playerLoad没有运行，因为它已经成功完成，并且allow-start-if-complete是'false'（默认值） .  gameLoad再次运行并处理另外两个文件，同时将它们的内容加载到'GAMES'表中（带有指示它们尚未处理的过程指示符） .  playerSummarization开始处理所有剩余的游戏数据（使用过程指示器过滤）并在30分钟后再次失败 . 

运行3：

playerLoad没有运行，因为它已经成功完成，并且allow-start-if-complete是'false'（默认值） .  gameLoad再次运行并处理另外两个文件，同时将它们的内容加载到'GAMES'表中（带有指示它们尚未处理的过程指示符） .  playerSummarization未启动且作业立即被终止，因为这是playerSummarization的第三次执行，其限制仅为2.必须提高限制或者必须将Job作为新的JobInstance执行 . 


#### 1.1.5.配置跳过逻辑

在许多情况下，处理过程中遇到的错误不应导致_13424失败，但应该跳过 . 这通常是一个必须由了解数据本身及其含义的人做出的决定 . 例如，财务数据可能无法跳过，因为它会导致资金转移，这需要完全准确 . 另一方面，加载供应商列表可能允许跳过 . 如果由于格式不正确或缺少必要信息而未加载供应商，则可能没有问题 . 通常，这些不良记录也会被记录下来，这在讨论听众时会有所涉及 . 

以下示例显示了使用跳过限制的示例：

XML配置


```xml
<step id="step1">
   <tasklet>
      <chunk reader="flatFileItemReader" writer="itemWriter"
             commit-interval="10" skip-limit="10">
         <skippable-exception-classes>
            <include class="org.springframework.batch.item.file.FlatFileParseException"/>
         </skippable-exception-classes>
      </chunk>
   </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(10)
                                .reader(flatFileItemReader())
                                .writer(itemWriter())
                                .faultTolerant()
                                .skipLimit(10)
                                .skip(FlatFileParseException.class)
                                .build();
}
```


在前面的示例中，使用了 `FlatFileItemReader`  . 如果在任何时候抛出了 `FlatFileParseException` ，则跳过该项并计算总跳过限制为10.在步执行中读取，处理和写入时跳过单独计数，但该限制适用于所有跳过 . 达到跳过限制后，发现下一个异常会导致步骤失败 . 换句话说，第十一次跳过触发异常，而不是第十次 . 

前面示例的一个问题是除了 `FlatFileParseException` 之外的任何其他异常都会导致 `Job` 失败 . 在某些情况下，这可能是正确的行为 . 但是，在其他情况下，可能更容易识别哪些异常应导致失败并跳过其他所有内容，如以下示例所示：

XML配置


```xml
<step id="step1">
    <tasklet>
        <chunk reader="flatFileItemReader" writer="itemWriter"
               commit-interval="10" skip-limit="10">
            <skippable-exception-classes>
                <include class="java.lang.Exception"/>
                <exclude class="java.io.FileNotFoundException"/>
            </skippable-exception-classes>
        </chunk>
    </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(10)
                                .reader(flatFileItemReader())
                                .writer(itemWriter())
                                .faultTolerant()
                                .skipLimit(10)
                                .skip(Exception.class)
                                .noSkip(FileNotFoundException.class)
                                .build();
}
```


通过将 `java.lang.Exception` 标识为可跳过的异常类，配置表明所有 `Exceptions` 都是可跳过的 . 但是，通过'排除' `java.io.FileNotFoundException` ，配置将可跳过的异常类列表细化为 `Exceptions` ，除了 `FileNotFoundException`  . 遇到任何排除的异常类都是致命的（也就是说，它们不会被跳过） . 

对于遇到的任何异常，可跳过性由最近的超类确定在类层次结构中 . 任何未分类的异常都被视为“致命” . 

 `<include/>` 和 `<exclude/>` 元素的顺序无关紧要 . 

 `skip` 和 `noSkip` 调用的顺序无关紧要 . 


#### 1.1.6.配置重试逻辑

在大多数情况下，您希望异常导致跳过或 `Step` 失败 . 但是，并非所有例外都是确定性的 . 如果在读取时遇到 `FlatFileParseException` ，则始终为该记录抛出 . 重置 `ItemReader` 无济于事 . 但是，对于其他异常，例如 `DeadlockLoserDataAccessException` ，表示当前进程已尝试更新另一个进程持有锁定的记录，等待并再次尝试可能会导致成功 . 在这种情况下，重试应配置如下：


```xml
<step id="step1">
   <tasklet>
      <chunk reader="itemReader" writer="itemWriter"
             commit-interval="2" retry-limit="3">
         <retryable-exception-classes>
            <include class="org.springframework.dao.DeadlockLoserDataAccessException"/>
         </retryable-exception-classes>
      </chunk>
   </tasklet>
</step>
```



```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .faultTolerant()
                                .retryLimit(3)
                                .retry(DeadlockLoserDataAccessException.class)
                                .build();
}
```


 `Step` 允许对可以重试单个项目的次数进行限制，并允许对“可重试”的异常列表进行限制 . 有关重试如何工作的更多详细信息，请参阅[retry](retry.html#retry) . 


#### 1.1.7.控制回滚

默认情况下，无论是重试还是跳过，从 `ItemWriter` 抛出的任何异常都会导致 `Step` 控制的事务回滚 . 如果如前所述配置skip，则 `ItemReader` 抛出的异常不会导致回滚 . 但是，在许多情况下，从 `ItemWriter` 抛出的异常不应导致回滚，因为没有采取任何措施使事务无效 . 因此， `Step` 可以配置一个不应导致回滚的异常列表，如以下示例所示：

XML配置


```xml
<step id="step1">
   <tasklet>
      <chunk reader="itemReader" writer="itemWriter" commit-interval="2"/>
      <no-rollback-exception-classes>
         <include class="org.springframework.batch.item.validator.ValidationException"/>
      </no-rollback-exception-classes>
   </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .faultTolerant()
                                .noRollback(ValidationException.class)
                                .build();
}
```



##### Transactional读者

 `ItemReader` 的基本 Contract 是它只是向前的 . 该步骤缓冲读取器输入，因此在回滚的情况下，不需要从读取器重新读取项目 . 但是，在某些情况下，阅读器构建在事务资源之上，例如JMS队列 . 在这种情况下，由于队列与回滚的事务相关联，因此将从队列中提取的消息重新打开 . 因此，可以将步骤配置为不缓冲项目，如以下示例所示：

XML配置


```xml
<step id="step1">
    <tasklet>
        <chunk reader="itemReader" writer="itemWriter" commit-interval="2"
               is-reader-transactional-queue="true"/>
    </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .readerIsTransactionalQueue()
                                .build();
}
```



#### 1.1.8.交易属性

事务属性可用于控制 `isolation` ， `propagation` 和 `timeout` 设置 . 有关设置事务属性的更多信息，请参见[Spring core documentation](https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#transaction) . 以下示例设置 `isolation` ， `propagation` 和 `timeout` 事务属性：

XML配置


```xml
<step id="step1">
    <tasklet>
        <chunk reader="itemReader" writer="itemWriter" commit-interval="2"/>
        <transaction-attributes isolation="DEFAULT"
                                propagation="REQUIRED"
                                timeout="30"/>
    </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        DefaultTransactionAttribute attribute = new DefaultTransactionAttribute();
        attribute.setPropagationBehavior(Propagation.REQUIRED.value());
        attribute.setIsolationLevel(Isolation.DEFAULT.value());
        attribute.setTimeout(30);

        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(itemReader())
                                .writer(itemWriter())
                                .transactionAttribute(attribute)
                                .build();
}
```



#### 1.1.9.使用Step注册ItemStream

该步骤必须在其生命周期的必要点处理 `ItemStream` 回调（有关 `ItemStream` 接口的更多信息，请参阅[ItemStream](readersAndWriters.html#itemStream)） . 如果步骤失败并且可能需要重新启动，这是至关重要的，因为 `ItemStream` 接口是步骤获取执行之间持久状态所需的信息的位置 . 

如果 `ItemReader` ， `ItemProcessor` 或 `ItemWriter` 本身实现 `ItemStream` 接口，则会自动注册这些接口 . 任何其他流需要单独注册 . 通常情况下，间接依赖关系（如委托）会被注入到读写器中 . 可以通过'streams'元素在 `Step` 上注册流，如以下示例所示：

XML配置


```xml
<step id="step1">
    <tasklet>
        <chunk reader="itemReader" writer="compositeWriter" commit-interval="2">
            <streams>
                <stream ref="fileItemWriter1"/>
                <stream ref="fileItemWriter2"/>
            </streams>
        </chunk>
    </tasklet>
</step>

<beans:bean id="compositeWriter"
            class="org.springframework.batch.item.support.CompositeItemWriter">
    <beans:property name="delegates">
        <beans:list>
            <beans:ref bean="fileItemWriter1" />
            <beans:ref bean="fileItemWriter2" />
        </beans:list>
    </beans:property>
</beans:bean>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(itemReader())
                                .writer(compositeItemWriter())
                                .stream(fileItemWriter1())
                                .stream(fileItemWriter2())
                                .build();
}

/**
 * In Spring Batch 4, the CompositeItemWriter implements ItemStream so this isn't
 * necessary, but used for an example.
 */
@Bean
public CompositeItemWriter compositeItemWriter() {
        List<ItemWriter> writers = new ArrayList<>(2);
        writers.add(fileItemWriter1());
        writers.add(fileItemWriter2());

        CompositeItemWriter itemWriter = new CompositeItemWriter();

        itemWriter.setDelegates(writers);

        return itemWriter;
}
```


在上面的示例中， `CompositeItemWriter` 不是 `ItemStream` ，但它的两个代理都是 . 因此，必须将两个委托编写器显式注册为流，以便框架正确处理它们 .   `ItemReader` 不需要显式注册为流，因为它是 `Step` 的直接属性 . 此步骤现在可以重新启动，并且在发生故障时正确保持读写器的状态 . 


#### 1.1.10.拦截步骤执行

与 `Job` 一样，在执行_13490期间有许多事件，用户可能需要执行某些功能 . 例如，为了写出需要页脚的平面文件， `Step` 需要在 `Step` 完成时得到通知，以便可以写入页脚 . 这可以通过许多 `Step` 范围的侦听器之一来完成 . 

任何实现 `StepListener` 的扩展之一的类（但不是那个接口本身，因为它是空的）可以应用于 `listeners` 元素的一个步骤 .   `listeners` 元素在步骤，tasklet或块声明中有效 . 建议您在其函数应用的级别声明侦听器，或者，如果它是多功能的（例如 `StepExecutionListener` 和 `ItemReadListener` ），则在应用它的最精细级别声明它 . 以下示例显示了在块级别应用的侦听器：

XML配置


```xml
<step id="step1">
    <tasklet>
        <chunk reader="reader" writer="writer" commit-interval="10"/>
        <listeners>
            <listener ref="chunkListener"/>
        </listeners>
    </tasklet>
</step>
```


Java配置


```java
@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(10)
                                .reader(reader())
                                .writer(writer())
                                .listener(chunkListener())
                                .build();
}
```


 `ItemReader` ， `ItemWriter` 或 `ItemProcessor` 本身实现了 `StepListener` 接口之一已注册如果使用命名空间 `<step>` 元素或其中一个 `*StepFactoryBean` 工厂，则自动使用 `Step`  . 这仅适用于直接注入 `Step` 的组件 . 如果侦听器嵌套在另一个组件中，则需要显式注册（如前面[Registering ItemStream with a Step](#registeringItemStreams)中所述） . 

除了 `StepListener` 接口之外，还提供了注释来解决相同的问题 . 普通的旧Java对象可以使用带有这些注释的方法，然后将这些注释转换为相应的 `StepListener` 类型 . 注释块组件的自定义实现（例如 `ItemReader` 或 `ItemWriter` 或 `Tasklet` ）也很常见 .  XML解析器为 `<listener/>` 元素分析注释，并在构建器中使用 `listener` 方法注册，因此您只需使用XML命名空间或构建器通过步骤注册侦听器 . 


##### StepExecutionListener

 `StepExecutionListener` 表示 `Step` 执行的最通用侦听器 . 它允许在 `Step` 开始之前和结束之后进行通知，无论它是正常结束还是失败，如以下示例所示：


```java
public interface StepExecutionListener extends StepListener {

    void beforeStep(StepExecution stepExecution);

    ExitStatus afterStep(StepExecution stepExecution);

}
```


 `ExitStatus` 是 `afterStep` 的返回类型，以便允许侦听器修改完成 `Step` 时返回的退出代码 . 

与此接口对应的注释是：


- 
 `@BeforeStep` 


- 
 `@AfterStep` 


##### ChunkListener

块被定义为在事务范围内处理的项 . 在每个提交间隔提交一个事务，提交一个“块” .   `ChunkListener` 可用于在块开始处理之前或块成功完成之后执行逻辑，如以下接口定义所示：


```java
public interface ChunkListener extends StepListener {

    void beforeChunk(ChunkContext context);
    void afterChunk(ChunkContext context);
    void afterChunkError(ChunkContext context);

}
```


在事务启动之后但在 `ItemReader` 上调用read之前调用beforeChunk方法 . 相反，在提交了块之后调用 `afterChunk` （如果存在回滚则根本不调用） . 

与此接口对应的注释是：


- 
 `@BeforeChunk` 


- 
 `@AfterChunk` 


- 
 `@AfterChunkError` 

当没有块声明时，可以应用 `ChunkListener`  .   `TaskletStep` 负责调用 `ChunkListener` ，因此它也适用于非面向项目的tasklet（在tasklet之前和之后调用） . 


##### ItemReadListener

在讨论先前的跳过逻辑时，有人提到记录跳过的记录可能是有益的，以便以后可以处理它们 . 在读取错误的情况下，可以使用 `ItemReaderListener` 来完成此操作，如以下接口定义所示：


```java
public interface ItemReadListener<T> extends StepListener {

    void beforeRead();
    void afterRead(T item);
    void onReadError(Exception ex);

}
```


在每次调用 `ItemReader` 之前调用 `beforeRead` 方法 . 在每次成功调用read之后调用 `afterRead` 方法，并传递已读取的项 . 如果读取时出错，则调用 `onReadError` 方法 . 提供了遇到的异常，以便可以记录它 . 

与此接口对应的注释是：


- 
 `@BeforeRead` 


- 
 `@AfterRead` 


- 
 `@OnReadError` 


##### ItemProcessListener

与 `ItemReadListener` 一样，可以“监听”项目的处理，如以下界面定义所示：


```java
public interface ItemProcessListener<T, S> extends StepListener {

    void beforeProcess(T item);
    void afterProcess(T item, S result);
    void onProcessError(T item, Exception e);

}
```


 `beforeProcess` 方法在 `ItemProcessor` 上的 `process` 之前调用，并被传递给要处理的项目 . 在成功处理项目后调用 `afterProcess` 方法 . 如果处理时出错，则调用 `onProcessError` 方法 . 遇到异常并且提供了尝试处理的项目，以便可以记录它们 . 

与此接口对应的注释是：


- 
 `@BeforeProcess` 


- 
 `@AfterProcess` 


- 
 `@OnProcessError` 


##### ItemWriteListener

可以使用 `ItemWriteListener` “监听”项目的写入，如以下接口定义所示：


```java
public interface ItemWriteListener<S> extends StepListener {

    void beforeWrite(List<? extends S> items);
    void afterWrite(List<? extends S> items);
    void onWriteError(Exception exception, List<? extends S> items);

}
```


 `beforeWrite` 方法在 `ItemWriter` 之前的 `write` 之前调用，并被传递给写入的项目列表 . 在成功写入项目后调用 `afterWrite` 方法 . 如果写入时出错，则调用 `onWriteError` 方法 . 遇到异常并尝试写入项目，以便记录它们 . 

与此接口对应的注释是：


- 
 `@BeforeWrite` 


- 
 `@AfterWrite` 


- 
 `@OnWriteError` 


##### SkipListener

 `ItemReadListener` ， `ItemProcessListener` 和 `ItemWriteListener` 都提供了通知错误的机制，但没有一个通知您实际上已经跳过了一条记录 . 例如，即使项目被重试并成功，也会调用 `onWriteError`  . 因此，有一个单独的界面用于跟踪跳过的项目，如以下界面定义所示：


```java
public interface SkipListener<T,S> extends StepListener {

    void onSkipInRead(Throwable t);
    void onSkipInProcess(T item, Throwable t);
    void onSkipInWrite(S item, Throwable t);

}
```


只要在阅读时跳过某个项目，就会调用 `onSkipInRead`  . 应该注意的是，回滚可能导致同一项目被注册为多次跳过 . 在写入时跳过项目时会调用 `onSkipInWrite`  . 由于该项已成功读取（并且未被跳过），因此还将项目本身作为参数提供 . 

与此接口对应的注释是：


- 
 `@OnSkipInRead` 


- 
 `@OnSkipInWrite` 


-  `@OnSkipInProcess` 


###### SkipListeners和Transactions

 `SkipListener` 最常见的用例之一是注销跳过的项目，以便可以使用另一个批处理过程甚至人工过程来评估和修复导致跳过的问题 . 因为有许多情况可以回滚原始事务，所以Spring Batch提供两个保证：

每个项目只调用一次适当的跳过方法（取决于发生错误的时间） . 始终在提交事务之前调用SkipListener . 这是为了确保侦听器调用的任何事务资源不会因ItemWriter中的失败而回滚 . 


### 1.2. TaskletStep

[Chunk-oriented processing](#chunkOrientedProcessing)不是 `Step` 中处理的唯一方法 . 如果 `Step` 必须包含简单的存储过程调用怎么办？您可以将该调用实现为 `ItemReader` ，并在该过程完成后返回null . 但是，这样做有点不自然，因为需要一个no-op  `ItemWriter`  .  Spring Batch为此方案提供了 `TaskletStep` . 

 `Tasklet` 是一个简单的接口，它有一个方法 `execute` ，由 `TaskletStep` 重复调用，直到它返回 `RepeatStatus.FINISHED` 或抛出异常来表示失败 . 每次调用_13587都包含在一个事务中 .   `Tasklet` 实现者可能会调用存储过程，脚本或简单的SQL更新语句 . 

要创建 `TaskletStep` ，<tasklet />元素的'ref'属性应引用定义 `Tasklet` 对象的bean . 不应在<tasklet />中使用<chunk />元素 . 以下示例显示了一个简单的tasklet：


```xml
<step id="step1">
    <tasklet ref="myTasklet"/>
</step>
```


要创建 `TaskletStep` ，传递给构建器的 `tasklet` 方法的bean应实现 `Tasklet` 接口 . 构建 `TaskletStep` 时不应调用 `chunk`  . 以下示例显示了一个简单的tasklet：


```java
@Bean
public Step step1() {
    return this.stepBuilderFactory.get("step1")
                            .tasklet(myTasklet())
                            .build();
}
```



> 

 `TaskletStep` 如果实现 `StepListener` 接口，则自动将tasklet注册为 `StepListener`  . 


#### 1.2.1. TaskletAdapter

与 `ItemReader` 和 `ItemWriter` 接口的其他适配器一样， `Tasklet` 接口包含一个允许自身适应任何预先存在的类的实现： `TaskletAdapter`  . 这可能有用的示例是现有的DAO，用于更新一组记录上的标志 .   `TaskletAdapter` 可用于调用此类，而无需为 `Tasklet` 接口编写适配器，如以下示例所示：

XML配置


```xml
<bean id="myTasklet" class="o.s.b.core.step.tasklet.MethodInvokingTaskletAdapter">
    <property name="targetObject">
        <bean class="org.mycompany.FooDao"/>
    </property>
    <property name="targetMethod" value="updateFoo" />
</bean>
```


Java配置


```java
@Bean
public MethodInvokingTaskletAdapter myTasklet() {
        MethodInvokingTaskletAdapter adapter = new MethodInvokingTaskletAdapter();

        adapter.setTargetObject(fooDao());
        adapter.setTargetMethod("updateFoo");

        return adapter;
}
```



#### 1.2.2.示例Tasklet实现

许多批处理作业包含必须在主处理开始之前完成的步骤，以便设置各种资源或在处理完成后清理这些资源 . 如果作业对文件有很大影响，通常需要在将某些文件成功上载到其他位置后在本地删除它们 . 以下示例（取自[Spring Batch samples project](https://github.com/spring-projects/spring-batch/tree/master/spring-batch-samples)）是一个 `Tasklet` 实现，只有这样的责任：


```java
public class FileDeletingTasklet implements Tasklet, InitializingBean {

    private Resource directory;

    public RepeatStatus execute(StepContribution contribution,
                                ChunkContext chunkContext) throws Exception {
        File dir = directory.getFile();
        Assert.state(dir.isDirectory());

        File[] files = dir.listFiles();
        for (int i = 0; i < files.length; i++) {
            boolean deleted = files[i].delete();
            if (!deleted) {
                throw new UnexpectedJobExecutionException("Could not delete file " +
                                                          files[i].getPath());
            }
        }
        return RepeatStatus.FINISHED;
    }

    public void setDirectoryResource(Resource directory) {
        this.directory = directory;
    }

    public void afterPropertiesSet() throws Exception {
        Assert.notNull(directory, "directory must be set");
    }
}
```


前面的 `Tasklet` 实现删除给定目录中的所有文件 . 应该注意， `execute` 方法只被调用一次 . 剩下的就是从 `Step` 引用 `Tasklet` ：

XML配置


```xml
<job id="taskletJob">
    <step id="deleteFilesInDir">
       <tasklet ref="fileDeletingTasklet"/>
    </step>
</job>

<beans:bean id="fileDeletingTasklet"
            class="org.springframework.batch.sample.tasklet.FileDeletingTasklet">
    <beans:property name="directoryResource">
        <beans:bean id="directory"
                    class="org.springframework.core.io.FileSystemResource">
            <beans:constructor-arg value="target/test-outputs/test-dir" />
        </beans:bean>
    </beans:property>
</beans:bean>
```


Java配置


```java
@Bean
public Job taskletJob() {
        return this.jobBuilderFactory.get("taskletJob")
                                .start(deleteFilesInDir())
                                .build();
}

@Bean
public Step deleteFilesInDir() {
        return this.stepBuilderFactory.get("deleteFilesInDir")
                                .tasklet(fileDeletingTasklet())
                                .build();
}

@Bean
public FileDeletingTasklet fileDeletingTasklet() {
        FileDeletingTasklet tasklet = new FileDeletingTasklet();

        tasklet.setDirectoryResource(new FileSystemResource("target/test-outputs/test-dir"));

        return tasklet;
}
```



### 1.3.控制步骤流程

由于能够在拥有的工作中将步骤组合在一起，因此需要能够控制作业从一个步骤到另一个步骤的工作方式 .   `Step` 的失败并不一定意味着 `Job` 应该失败 . 此外，可能有多种类型的“成功”决定了接下来应该执行哪个 `Step`  . 根据一组 `Steps` 的配置方式，某些步骤甚至可能根本不会被处理 . 


#### 1.3.1.顺序流量

最简单的流程方案是所有步骤按顺序执行的作业，如下图所示：


![Sequential Flow](https://www.docs4dev.com/images/fd677697-82f6-4a0e-9cf1-14ca74a7161d.png)


图3.顺序流程

这可以通过使用step元素的'next'属性来实现，如以下示例所示：

XML配置


```xml
<job id="job">
    <step id="stepA" parent="s1" next="stepB" />
    <step id="stepB" parent="s2" next="stepC"/>
    <step id="stepC" parent="s3" />
</job>
```


Java配置


```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                                .start(stepA())
                                .next(stepB())
                                .next(stepC())
                                .build();
}
```


在上面的场景中，'step A'首先运行，因为它是列出的第一个 `Step`  . 如果'步骤A'正常完成，则'步骤B'运行，依此类推 . 但是，如果“步骤A”失败，则整个 `Job` 失败并且“步骤B”不执行 . 


> 

使用Spring Batch命名空间，配置中列出的第一步始终是 `Job` 运行的第一步 . 其他步骤元素的顺序无关紧要，但第一步必须始终首先出现在xml中 . 


#### 1.3.2.条件流程

在上面的例子中，只有两种可能性：

步骤成功，应执行下一步 . 步骤失败，因此，作业应该失败 . 

在许多情况下，这可能就足够了 . 但是，如果 `Step` 的失败应该触发不同的_13635而不是导致失败呢？下图显示了这样的流程：


![Conditional Flow](https://www.docs4dev.com/images/3c26fd24-d773-4ba9-8bc1-b1cd6818491a.png)


图4.条件流

为了为了处理更复杂的场景，Spring Batch命名空间允许在step元素中定义过渡元素 . 一个这样的过渡是 `next` 元素 . 与 `next` 属性一样， `next` 元素告诉 `Job` 下一个 `Step` 执行 . 但是，与属性不同，在给定的 `Step` 上允许任意数量的 `next` 元素，并且在失败的情况下不存在默认行为 . 这意味着，如果使用过渡元素，则必须明确定义 `Step` 过渡的所有行为 . 另请注意，单个步骤不能同时具有 `next` 属性和 `transition` 元素 . 

 `next` 元素指定要匹配的模式和下一步执行的步骤，如以下示例所示：

XML配置


```xml
<job id="job">
    <step id="stepA" parent="s1">
        <next on="*" to="stepB" />
        <next on="FAILED" to="stepC" />
    </step>
    <step id="stepB" parent="s2" next="stepC" />
    <step id="stepC" parent="s3" />
</job>
```


Java配置


```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                                .start(stepA())
                                .on("*").to(stepB())
                                .from(stepA()).on("FAILED").to(stepC())
                                .end()
                                .build();
}
```


使用XML配置时，transition元素的 `on` 属性使用简单的模式匹配方案来匹配 `Step` 的执行所产生的 `ExitStatus`  . 

使用java配置时， `on` 方法使用简单的模式匹配方案来匹配 `ExitStatus` 执行产生的 `ExitStatus`  . 

模式中只允许使用两个特殊字符：


- 
“*”匹配零个或多个字符


- 
“？”恰好匹配一个字符

例如，“c * t”匹配“cat”和“count”，而“c？t”匹配“cat”但不匹配“count” . 

虽然 `Step` 上的转换元素数量没有限制，但如果 `Step` 执行导致 `ExitStatus` 未被元素覆盖，则框架抛出异常并且 `Job` 失败 . 框架自动命令从最具体到最不具体的转换 . 这意味着，即使在上面的示例中为"stepA"交换了排序， `ExitStatus` 的 `ExitStatus` 仍然会转到"stepC" . 


##### Batch状态与退出状态

为条件流配置 `Job` 时，了解 `BatchStatus` 和 `ExitStatus` 之间的区别非常重要 .   `BatchStatus` 是一个枚举，它是 `JobExecution` 和 `StepExecution` 的属性，并由框架用于记录 `Job` 或 `Step` 的状态 . 它可以是以下值之一： `COMPLETED` ， `STARTING` ， `STARTED` ， `BatchStatus` ， `STOPPED` ， `FAILED` ， `ABANDONED` 或 `UNKNOWN`  . 其中大多数是自解释的： `COMPLETED` 是步骤或作业成功完成时设置的状态，失败时设置 `FAILED` ，依此类推 . 

以下示例在使用XML配置时包含“next”元素：


```xml
<next on="FAILED" to="stepB" />
```


以下示例在使用Java配置时包含'on'元素：


```java
...
.from(stepA()).on("FAILED").to(stepB())
...
```


乍一看，似乎'on'引用了它所属的 `BatchStatus` 的 `BatchStatus`  . 但是，它实际上引用了 `Step` 的 `ExitStatus`  . 顾名思义， `ExitStatus` 表示完成执行后 `Step` 的状态 . 

更具体地说，在使用XML配置时，前面的XML配置示例中显示的“next”元素引用了 `ExitStatus` 的退出代码 . 

使用Java配置时，前面的Java配置示例中显示的“on”方法引用 `ExitStatus` 的退出代码 . 

在英语中，它说：“如果退出代码是 `FAILED` ，则转到步骤B” . 默认情况下，退出代码始终与 `Step` 的 `BatchStatus` 相同，这就是上面的条目有效的原因 . 但是，如果退出代码需要不同呢？一个很好的例子来自示例项目中的跳过示例作业：

XML配置


```xml
<step id="step1" parent="s1">
    <end on="FAILED" />
    <next on="COMPLETED WITH SKIPS" to="errorPrint1" />
    <next on="*" to="step2" />
</step>
```


Java配置


```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                        .start(step1()).on("FAILED").end()
                        .from(step1()).on("COMPLETED WITH SKIPS").to(errorPrint1())
                        .from(step1()).on("*").to(step2())
                        .end()
                        .build();
}
```


 `step1` 有三种可能性：

步骤失败，在这种情况下作业应该失败 . 步骤成功完成 . 步骤已成功完成，但退出代码为“COMPLETED WITH SKIPS” . 在这种情况下，应该运行不同的步骤来处理错误 . 

以上配置有效 . 但是，某些内容需要根据跳过记录的执行条件更改退出代码，如以下示例所示：


```java
public class SkipCheckingListener extends StepExecutionListenerSupport {
    public ExitStatus afterStep(StepExecution stepExecution) {
        String exitCode = stepExecution.getExitStatus().getExitCode();
        if (!exitCode.equals(ExitStatus.FAILED.getExitCode()) &&
              stepExecution.getSkipCount() > 0) {
            return new ExitStatus("COMPLETED WITH SKIPS");
        }
        else {
            return null;
        }
    }
}
```


上面的代码是 `StepExecutionListener` ，它首先检查以确保 `Step` 成功，然后检查 `StepExecution` 上的跳过计数是否高于0.如果满足这两个条件，则返回退出代码为 `COMPLETED WITH SKIPS` 的新 `ExitStatus`   . 


#### 1.3.3.配置停止

在讨论[BatchStatus and ExitStatus](#batchStatusVsExitStatus)之后，人们可能想知道 `BatchStatus` 和 `ExitStatus` 是如何为 `Job` 确定的 . 虽然这些状态是由执行的代码确定的 `Step` ，但 `Job` 的状态是根据配置确定的 . 

到目前为止，所讨论的所有作业配置至少有一个没有转换的 `Step`  . 例如，执行以下步骤后， `Job` 结束，如以下示例所示：


```xml
<step id="stepC" parent="s3"/>
```



```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                                .start(step1())
                                .build();
}
```


如果没有为 `Step` 定义转换，则 `Job` 的状态定义如下：


- 
如果 `Step` 以 `ExitStatus`  FAILED结束，那么 `Job` 的 `BatchStatus` 和 `ExitStatus` 都是 `FAILED`  . 


- 
否则， `Job` 的 `BatchStatus` 和 `ExitStatus` 都是 `COMPLETED`  . 

虽然这种终止批处理作业的方法对于某些批处理作业（例如简单的顺序步骤作业）已足够，但可能需要自定义的作业停止方案 . 为此，Spring Batch提供了三个过渡元素来停止 `Job` （除了我们之前讨论过的[next element](#nextElement)） . 这些停止元素中的每一个都使用特定的 `BatchStatus` 来停止 `Job`  . 重要的是要注意，停止转换元素对 `Job` 中任何 `Steps` 的 `BatchStatus` 或 `ExitStatus` 都没有影响 . 这些元素仅影响 `Job` 的最终状态 . 例如，作业中的每个步骤都可能具有 `FAILED` 的状态，但作业的状态为 `COMPLETED` . 


##### 结束一步

配置步骤结束指示 `Job` 以 `BatchStatus`   `COMPLETED` 停止 . 已完成状态 `COMPLETED` 的 `Job` 无法重新启动（框架抛出 `JobInstanceAlreadyCompleteException` ） . 

使用XML配置时，'end'元素用于此任务 .   `end` 元素还允许使用可选的“退出代码”属性，该属性可用于自定义 `Job` 的 `ExitStatus`  . 如果没有给出'exit-code'属性，那么 `ExitStatus` 默认为 `COMPLETED` ，以匹配 `BatchStatus`  . 

使用Java配置时，'end'方法用于此任务 .   `end` 方法还允许使用可选的'exitStatus'参数，该参数可用于自定义 `Job` 的 `ExitStatus`  . 如果未提供“exitStatus”值，则 `ExitStatus` 默认为 `COMPLETED` ，以匹配 `BatchStatus`  . 

在以下方案中，如果 `step2` 失败，则 `Job` 将停止 `BatchStatus`   `COMPLETED`   `ExitStatus`   `COMPLETED` 和 `step3` 不运行 . 否则，执行移至 `step3`  . 请注意，如果 `step2` 失败，则 `Job` 不可重新启动（因为状态为 `COMPLETED` ） . 


```xml
<step id="step1" parent="s1" next="step2">

<step id="step2" parent="s2">
    <end on="FAILED"/>
    <next on="*" to="step3"/>
</step>

<step id="step3" parent="s3">
```



```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                                .start(step1())
                                .next(step2())
                                .on("FAILED").end()
                                .from(step2()).on("*").to(step3())
                                .end()
                                .build();
}
```



##### Failing a Step

在给定点配置失败步骤会指示 `Job` 以 `BatchStatus`   `FAILED` 停止 . 与end不同， `Job` 的失败并不会阻止 `Job` 重新启动 . 

使用XML配置时，'fail'元素还允许可选的'exit-code'属性，该属性可用于自定义 `Job` 的 `ExitStatus`  . 如果没有给出'exit-code'属性，那么 `ExitStatus` 默认为 `FAILED` ，以匹配 `BatchStatus`  . 

在以下方案中，如果 `step2` 失败，则 `Job` 以 `BatchStatus` 为 `FAILED` 停止， `ExitStatus` 为 `EARLY TERMINATION` 且 `step3` 不执行 . 否则，执行将移至 `step3`  . 此外，如果 `step2` 失败并重新启动 `Job` ，则会再次在 `step2` 上执行 . 

XML配置


```xml
<step id="step1" parent="s1" next="step2">

<step id="step2" parent="s2">
    <fail on="FAILED" exit-code="EARLY TERMINATION"/>
    <next on="*" to="step3"/>
</step>

<step id="step3" parent="s3">
```


Java配置


```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                        .start(step1())
                        .next(step2()).on("FAILED").fail()
                        .from(step2()).on("*").to(step3())
                        .end()
                        .build();
}
```


_0005在给定步骤中执行作业

配置作业在特定步骤停止指示 `Job` 以 `BatchStatus`   `STOPPED` 停止 . 停止 `Job` 可以在处理过程中提供临时中断，以便操作员可以在重新启动_13796之前采取一些操作 . 

使用XML配置时，'stop'元素需要'restart'属性，该属性指定在“重新启动作业”时应执行的步骤 . 

使用java配置时， `stopAndRestart` 方法需要一个'restart'属性，该属性指定"Job is restarted"时执行应该执行的步骤 . 

在以下方案中，如果 `step1` 完成 `COMPLETE` ，则作业将停止 . 重新启动后，将在 `step2` 上开始执行 . 


```xml
<step id="step1" parent="s1">
    <stop on="COMPLETED" restart="step2"/>
</step>

<step id="step2" parent="s2"/>
```



```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                        .start(step1()).on("COMPLETED").stopAndRestart(step2())
                        .end()
                        .build();
}
```



#### 1.3.4.程序化流程决策

在某些情况下，可能需要比_13805更多的信息来决定接下来要执行的步骤 . 在这种情况下，可以使用 `JobExecutionDecider` 来帮助做出决定，如以下示例所示：


```java
public class MyDecider implements JobExecutionDecider {
    public FlowExecutionStatus decide(JobExecution jobExecution, StepExecution stepExecution) {
        String status;
        if (someCondition()) {
            status = "FAILED";
        }
        else {
            status = "COMPLETED";
        }
        return new FlowExecutionStatus(status);
    }
}
```


在以下示例作业配置中， `decision` 指定要使用的决策程序以及所有过渡：

XML配置


```xml
<job id="job">
    <step id="step1" parent="s1" next="decision" />

    <decision id="decision" decider="decider">
        <next on="FAILED" to="step2" />
        <next on="COMPLETED" to="step3" />
    </decision>

    <step id="step2" parent="s2" next="step3"/>
    <step id="step3" parent="s3" />
</job>

<beans:bean id="decider" class="com.MyDecider"/>
```


在以下示例中，使用Java配置时，实现 `JobExecutionDecider` 的bean将直接传递给 `next` 调用 . 

Java配置


```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                        .start(step1())
                        .next(decider()).on("FAILED").to(step2())
                        .from(decider()).on("COMPLETED").to(step3())
                        .end()
                        .build();
}
```



#### 1.3.5.分流

到目前为止所描述的每个场景都涉及一个 `Job` ，它以线性方式一次执行一个步骤 . 除了这种典型的风格，Spring Batch还允许使用并行流配置作业 . 

XML命名空间允许您使用'split'元素 . 如下例所示，'split'元素包含一个或多个'flow'元素，其中可以定义整个单独的流 .  'split'元素还可以包含任何前面讨论的过渡元素，例如'next'属性或'next'，'end'或'fail'元素 . 


```xml
<split id="split1" next="step4">
    <flow>
        <step id="step1" parent="s1" next="step2"/>
        <step id="step2" parent="s2"/>
    </flow>
    <flow>
        <step id="step3" parent="s3"/>
    </flow>
</split>
<step id="step4" parent="s4"/>
```


基于Java的配置允许您通过提供的构建器配置拆分 . 如下例所示，'split'元素包含一个或多个'flow'元素，其中可以定义整个单独的流 .  'split'元素还可以包含任何前面讨论的过渡元素，例如'next'属性或'next'，'end'或'fail'元素 . 


```java
@Bean
public Job job() {
        Flow flow1 = new FlowBuilder<SimpleFlow>("flow1")
                        .start(step1())
                        .next(step2())
                        .build();
        Flow flow2 = new FlowBuilder<SimpleFlow>("flow2")
                        .start(step3())
                        .build();

        return this.jobBuilderFactory.get("job")
                                .start(flow1)
                                .split(new SimpleAsyncTaskExecutor())
                                .add(flow2)
                                .next(step4())
                                .end()
                                .build();
}
```



#### 1.3.6.外化流程工作之间的定义和依赖关系

作业中的部分流可以外部化为单独的bean定义，然后重新使用 . 有两种方法可以做到这一点 . 第一种是简单地将流声明为对其他地方定义的流的引用，如以下示例所示：

XML配置


```xml
<job id="job">
    <flow id="job1.flow1" parent="flow1" next="step3"/>
    <step id="step3" parent="s3"/>
</job>

<flow id="flow1">
    <step id="step1" parent="s1" next="step2"/>
    <step id="step2" parent="s2"/>
</flow>
```


Java配置


```java
@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                                .start(flow1())
                                .next(step3())
                                .end()
                                .build();
}

@Bean
public Flow flow1() {
        return new FlowBuilder<SimpleFlow>("flow1")
                        .start(step1())
                        .next(step2())
                        .build();
}
```


如前面的示例所示，定义外部流的效果是将外部流中的步骤插入到作业中，就好像它们已被内联声明一样 . 通过这种方式，许多作业可以引用相同的模板流并将这些模板组合成不同的逻辑流 . 这也是分离各个流程的集成测试的好方法 . 

另一种形式的外化流程是使用 `JobStep`  .   `JobStep` 类似于 `FlowStep` 但实际上为指定的流中的步骤创建并启动单独的作业执行 . 

以下XML代码段显示了 `JobStep` 的示例：

XML配置


```xml
<job id="jobStepJob" restartable="true">
   <step id="jobStepJob.step1">
      <job ref="job" job-launcher="jobLauncher"
          job-parameters-extractor="jobParametersExtractor"/>
   </step>
</job>

<job id="job" restartable="true">...</job>

<bean id="jobParametersExtractor" class="org.spr...DefaultJobParametersExtractor">
   <property name="keys" value="input.file"/>
</bean>
```


以下Java代码段显示了 `JobStep` 的示例：

Java配置


```java
@Bean
public Job jobStepJob() {
        return this.jobBuilderFactory.get("jobStepJob")
                                .start(jobStepJobStep1(null))
                                .build();
}

@Bean
public Step jobStepJobStep1(JobLauncher jobLauncher) {
        return this.stepBuilderFactory.get("jobStepJobStep1")
                                .job(job())
                                .launcher(jobLauncher)
                                .parametersExtractor(jobParametersExtractor())
                                .build();
}

@Bean
public Job job() {
        return this.jobBuilderFactory.get("job")
                                .start(step1())
                                .build();
}

@Bean
public DefaultJobParametersExtractor jobParametersExtractor() {
        DefaultJobParametersExtractor extractor = new DefaultJobParametersExtractor();

        extractor.setKeys(new String[]{"input.file"});

        return extractor;
}
```


作业参数提取器是一种策略，用于确定 `Step` 的 `ExecutionContext` 如何转换为运行的 `Job` 的 `JobParameters`  . 当您希望有一些更精细的选项来监视和报告作业和步骤时， `JobStep` 非常有用 . 使用 `JobStep` 通常也是这个问题的一个很好的答案："How do I create dependencies between jobs?"这是一个将大型系统分解成更小模块并控制作业流的好方法 . 


### 1.4.作业和步骤属性的后期绑定

前面显示的XML和平面文件示例都使用Spring  `Resource` 抽象来获取文件 . 这是有效的，因为 `Resource` 有一个 `getFile` 方法，它返回一个 `java.io.File`  . 可以使用标准的Spring结构配置XML和平面文件资源，如以下示例所示：

XML配置


```xml
<bean id="flatFileItemReader"
      class="org.springframework.batch.item.file.FlatFileItemReader">
    <property name="resource"
              value="file://outputs/file.txt" />
</bean>
```


Java配置


```java
@Bean
public FlatFileItemReader flatFileItemReader() {
        FlatFileItemReader<Foo> reader = new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource("file://outputs/file.txt"))
                        ...
}
```


前面的 `Resource` 从指定的文件系统位置加载文件 . 请注意，绝对位置必须以双斜杠（ `//` ）开头 . 在大多数Spring应用程序中，此解决方案已经足够好了，因为这些资源的名称在编译时是已知的 . 但是，在批处理方案中，可能需要在运行时确定文件名作为作业的参数 . 这可以使用'-D'参数来解决系统属性 . 

以下XML代码段显示了如何从属性中读取文件名：

XML配置


```xml
<bean id="flatFileItemReader"
      class="org.springframework.batch.item.file.FlatFileItemReader">
    <property name="resource" value="${input.file.name}" />
</bean>
```


以下Java代码段显示了如何从属性中读取文件名：

Java配置


```java
@Bean
public FlatFileItemReader flatFileItemReader(@Value("${input.file.name}") String name) {
        return new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource(name))
                        ...
}
```


此解决方案工作所需的全部内容都是系统参数（例如 `-Dinput.file.name="file://outputs/file.txt"` ） . 


> 虽然这里可以使用 `PropertyPlaceholderConfigurer` ，但是如果系统属性始终设置则没有必要，因为Spring中的 `ResourceEditor` 已经过滤并在系统属性上替换占位符 . 

通常，在批处理设置中，最好在作业的 `JobParameters` 中对文件名进行参数化，而不是通过系统属性，并以这种方式访问它们 . 为实现此目的，Spring Batch允许各种 `Job` 和 `Step` 属性的后期绑定，如以下代码段所示：

XML配置


```xml
<bean id="flatFileItemReader" scope="step"
      class="org.springframework.batch.item.file.FlatFileItemReader">
    <property name="resource" value="#{jobParameters['input.file.name']}" />
</bean>
```


Java配置


```java
@StepScope
@Bean
public FlatFileItemReader flatFileItemReader(@Value("#{jobParameters['input.file.name']}") String name) {
        return new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource(name))
                        ...
}
```


 `JobExecution` 和 `StepExecution` 级 `ExecutionContext` 都可以以相同的方式访问，如以下示例所示：

XML配置


```xml
<bean id="flatFileItemReader" scope="step"
      class="org.springframework.batch.item.file.FlatFileItemReader">
    <property name="resource" value="#{jobExecutionContext['input.file.name']}" />
</bean>
```


XML配置


```xml
<bean id="flatFileItemReader" scope="step"
      class="org.springframework.batch.item.file.FlatFileItemReader">
    <property name="resource" value="#{stepExecutionContext['input.file.name']}" />
</bean>
```


Java配置


```java
@StepScope
@Bean
public FlatFileItemReader flatFileItemReader(@Value("#{jobExecutionContext['input.file.name']}") String name) {
        return new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource(name))
                        ...
}
```


Java配置


```java
@StepScope
@Bean
public FlatFileItemReader flatFileItemReader(@Value("#{stepExecutionContext['input.file.name']}") String name) {
        return new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource(name))
                        ...
}
```



> 

任何使用后期绑定的bean都必须使用scope = "step"声明 . 有关更多信息，请参见[Step Scope](#step-scope) . 


#### 1.4.1.步骤范围

上面的所有后期绑定示例都在bean定义上声明了“step”范围，如以下示例所示：

XML配置


```xml
<bean id="flatFileItemReader" scope="step"
      class="org.springframework.batch.item.file.FlatFileItemReader">
    <property name="resource" value="#{jobParameters[input.file.name]}" />
</bean>
```


Java配置


```java
@StepScope
@Bean
public FlatFileItemReader flatFileItemReader(@Value("#{jobParameters[input.file.name]}") String name) {
        return new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource(name))
                        ...
}
```


为了使用后期绑定，需要使用 `Step` 的范围，因为在 `Step` 启动之前实际上不能实例化bean，以允许找到属性 . 因为默认情况下它不是Spring容器的一部分，所以必须通过使用 `batch` 命名空间或通过为 `StepScope` 显式包含bean定义或使用 `@EnableBatchProcessing` 注释显式添加范围 . 只使用其中一种方法 . 以下示例使用 `batch` 命名空间：


```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:batch="http://www.springframework.org/schema/batch"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="...">
<batch:job .../>
...
</beans>
```


以下示例显式包含bean定义：


```xml
<bean class="org.springframework.batch.core.scope.StepScope" />
```



#### 1.4.2.工作范围

Spring Batch 3.0中引入的 `Job`  scope与配置中的 `Step`  scope类似，但它是 `Job`  context的Scope，因此每个正在运行的作业只有一个这样的bean实例 . 此外，还提供了对使用 `#{..}` 占位符从 `JobContext` 访问的引用的后期绑定的支持 . 使用此功能，可以从作业或作业执行上下文和作业参数中提取Bean属性，如以下示例所示：

XML配置


```xml
<bean id="..." class="..." scope="job">
    <property name="name" value="#{jobParameters[input]}" />
</bean>
```


XML配置


```xml
<bean id="..." class="..." scope="job">
    <property name="name" value="#{jobExecutionContext['input.name']}.txt" />
</bean>
```


Java配置


```java
@JobScope
@Bean
public FlatFileItemReader flatFileItemReader(@Value("#{jobParameters[input]}") String name) {
        return new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource(name))
                        ...
}
```


Java配置


```java
@JobScope
@Bean
public FlatFileItemReader flatFileItemReader(@Value("#{jobExecutionContext['input.name']}") String name) {
        return new FlatFileItemReaderBuilder<Foo>()
                        .name("flatFileItemReader")
                        .resource(new FileSystemResource(name))
                        ...
}
```
因为默认情况下它不是Spring容器的一部分，所以必须通过使用 `batch` 命名空间，通过为JobScope显式包含bean定义，或使用 `@EnableBatchProcessing` 注释（但不是全部）来显式添加范围 . 以下示例使用 `batch` 命名空间：


```xml
<beans xmlns="http://www.springframework.org/schema/beans"
                  xmlns:batch="http://www.springframework.org/schema/batch"
                  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
                  xsi:schemaLocation="...">

<batch:job .../>
...
</beans>
```


以下示例包含一个显式定义 `JobScope` 的bean：


```xml
<bean class="org.springframework.batch.core.scope.JobScope" />
```
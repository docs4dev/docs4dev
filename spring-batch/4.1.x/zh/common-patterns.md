## 1.常见批处理模式

XML Java

一些批处理作业可以纯粹由Spring Batch中的现成组件组装 . 例如， `ItemReader` 和 `ItemWriter` 实现可以配置为涵盖各种场景 . 但是，对于大多数情况，必须编写自定义代码 . 应用程序开发人员的主要API入口点是 `Tasklet` ， `ItemReader` ， `ItemWriter` 和各种侦听器接口 . 大多数简单的批处理作业可以使用Spring Batch  `ItemReader` 的现成输入，但通常情况是处理和编写中存在需要开发人员实现 `ItemWriter` 或 `ItemProcessor` 的自定义问题 . 

在本章中，我们提供了一些自定义业务逻辑中常见模式的示例 . 这些示例主要以侦听器接口为特征 . 应该注意，如果合适， `ItemReader` 或 `ItemWriter` 也可以实现监听器接口 . 


### 1.1.记录项目处理和失败

一个常见的用例是需要逐步，逐项地对错误进行特殊处理，可能需要记录到特殊通道或将记录插入数据库 . 面向块的 `Step` （从步骤工厂bean创建）允许用户使用 `read` 上的错误 `ItemReadListener` 和 `write` 上的错误 `ItemWriteListener` 来实现此用例 . 以下代码段说明了一个记录读写失败的侦听器：


```java
public class ItemFailureLoggerListener extends ItemListenerSupport {

    private static Log logger = LogFactory.getLog("item.error");

    public void onReadError(Exception ex) {
        logger.error("Encountered error on read", e);
    }

    public void onWriteError(Exception ex, List<? extends Object> items) {
        logger.error("Encountered error on write", ex);
    }
}
```


实现此侦听器后，必须使用步骤注册，如以下示例所示：

XML配置


```xml
<step id="simpleStep">
...
<listeners>
    <listener>
        <bean class="org.example...ItemFailureLoggerListener"/>
    </listener>
</listeners>
</step>
```


Java配置


```java
@Bean
public Step simpleStep() {
        return this.stepBuilderFactory.get("simpleStep")
                                ...
                                .listener(new ItemFailureLoggerListener())
                                .build();
}
```



> 如果您的侦听器在 `onError()` 方法中执行任何操作，则它必须位于将要回滚的事务中 . 如果需要在 `onError()` 方法中使用事务资源（如数据库），请考虑向该方法添加声明性事务（有关详细信息，请参阅Spring Core Reference Guide），并为其传播属性赋予 `REQUIRES_NEW` 的值 . 


### 1.2.因业务原因手动停止工作

Spring Batch通过 `JobLauncher` 接口提供 `stop()` 方法，但这实际上是供操作员而不是应用程序员使用 . 有时，从业务逻辑中停止作业执行更方便或更有意义 . 

最简单的方法是抛出一个 `RuntimeException` （一个既不会无限期重试也不会被跳过） . 例如，可以使用自定义异常类型，如以下示例所示：


```java
public class PoisonPillItemProcessor<T> implements ItemProcessor<T, T> {

    @Override
    public T process(T item) throws Exception {
        if (isPoisonPill(item)) {
            throw new PoisonPillException("Poison pill detected: " + item);
        }
        return item;
    }
}
```


停止执行步骤的另一种简单方法是从 `ItemReader` 返回 `null` ，如以下示例所示：


```java
public class EarlyCompletionItemReader implements ItemReader<T> {

    private ItemReader<T> delegate;

    public void setDelegate(ItemReader<T> delegate) { ... }

    public T read() throws Exception {
        T item = delegate.read();
        if (isEndItem(item)) {
            return null; // end the step here
        }
        return item;
    }

}
```


前面的示例实际上依赖于以下事实： `CompletionPolicy` 策略的默认实现在要处理的项目为 `null` 时表示完整批次 . 可以实现更复杂的完成策略，并通过 `SimpleStepFactoryBean` 注入 `Step` ，如以下示例所示：

XML配置


```xml
<step id="simpleStep">
    <tasklet>
        <chunk reader="reader" writer="writer" commit-interval="10"
               chunk-completion-policy="completionPolicy"/>
    </tasklet>
</step>

<bean id="completionPolicy" class="org.example...SpecialCompletionPolicy"/>
```


Java配置


```java
@Bean
public Step simpleStep() {
        return this.stepBuilderFactory.get("simpleStep")
                                .<String, String>chunk(new SpecialCompletionPolicy())
                                .reader(reader())
                                .writer(writer())
                                .build();
}
```


另一种方法是在 `StepExecution` 中设置一个标志，该标志由项目处理之间的框架中的 `Step` 实现检查 . 要实现此替代方案，我们需要访问当前的 `StepExecution` ，这可以通过实现 `StepListener` 并将其注册到 `Step` 来实现 . 以下示例显示了设置标志的侦听器：


```java
public class CustomItemWriter extends ItemListenerSupport implements StepListener {

    private StepExecution stepExecution;

    public void beforeStep(StepExecution stepExecution) {
        this.stepExecution = stepExecution;
    }

    public void afterRead(Object item) {
        if (isPoisonPill(item)) {
            stepExecution.setTerminateOnly(true);
       }
    }

}
```


设置标志后，默认行为是针对抛出 `JobInterruptedException` 的步骤 . 可以通过 `StepInterruptionPolicy` 控制此行为 . 但是，唯一的选择是抛出或不抛出异常，因此这始终是作业的异常结束 . 


### 1.3.添加页脚记录

通常，在写入平面文件时，必须在完成所有处理之后将"footer"记录附加到文件的末尾 . 这可以使用Spring Batch提供的 `FlatFileFooterCallback` 接口来实现 .   `FlatFileFooterCallback` （及其对应的 `FlatFileHeaderCallback` ）是 `FlatFileItemWriter` 的可选属性，可以添加到项目编写器中，如以下示例所示：

XML配置


```xml
<bean id="itemWriter" class="org.spr...FlatFileItemWriter">
    <property name="resource" ref="outputResource" />
    <property name="lineAggregator" ref="lineAggregator"/>
    <property name="headerCallback" ref="headerCallback" />
    <property name="footerCallback" ref="footerCallback" />
</bean>
```


Java配置


```java
@Bean
public FlatFileItemWriter<String> itemWriter(Resource outputResource) {
        return new FlatFileItemWriterBuilder<String>()
                        .name("itemWriter")
                        .resource(outputResource)
                        .lineAggregator(lineAggregator())
                        .headerCallback(headerCallback())
                        .footerCallback(footerCallback())
                        .build();
}
```


页脚回调接口只有一个在必须写入页脚时调用的方法，如以下接口定义所示：


```java
public interface FlatFileFooterCallback {

    void writeFooter(Writer writer) throws IOException;

}
```



#### 1.3.1.编写摘要页脚

涉及页脚记录的常见要求是在输出过程中聚合信息并将此信息附加到文件末尾 . 此页脚通常用作文件的摘要或提供校验和 . 

例如，如果批处理作业正在将 `Trade` 记录写入平面文件，并且要求将所有 `Trades` 的总金额放在页脚中，则可以使用以下 `ItemWriter` 实现：


```java
public class TradeItemWriter implements ItemWriter<Trade>,
                                        FlatFileFooterCallback {

    private ItemWriter<Trade> delegate;

    private BigDecimal totalAmount = BigDecimal.ZERO;

    public void write(List<? extends Trade> items) throws Exception {
        BigDecimal chunkTotal = BigDecimal.ZERO;
        for (Trade trade : items) {
            chunkTotal = chunkTotal.add(trade.getAmount());
        }

        delegate.write(items);

        // After successfully writing all items
        totalAmount = totalAmount.add(chunkTotal);
    }

    public void writeFooter(Writer writer) throws IOException {
        writer.write("Total Amount Processed: " + totalAmount);
    }

    public void setDelegate(ItemWriter delegate) {...}
}
```


此 `TradeItemWriter` 存储 `totalAmount` 值，该值随着写入的每个 `Trade` 项目的 `amount` 而增加 . 在处理完最后一个 `Trade` 之后，框架调用 `writeFooter` ，它将 `totalAmount` 放入文件中 . 请注意 `write` 方法使用a临时变量 `chunkTotal` ，用于存储块中 `Trade` 金额的总和 . 这样做是为了确保如果在 `write` 方法中发生跳过，则 `totalAmount` 保持不变 . 它只是在 `write` 方法的末尾，一旦我们确保没有抛出异常，我们就更新 `totalAmount`  . 

为了调用 `writeFooter` 方法，必须将 `TradeItemWriter` （实现 `FlatFileFooterCallback` ）连接到 `FlatFileItemWriter` 作为 `footerCallback`  . 以下示例显示了如何执行此操作：

XML配置


```xml
<bean id="tradeItemWriter" class="..TradeItemWriter">
    <property name="delegate" ref="flatFileItemWriter" />
</bean>

<bean id="flatFileItemWriter" class="org.spr...FlatFileItemWriter">
   <property name="resource" ref="outputResource" />
   <property name="lineAggregator" ref="lineAggregator"/>
   <property name="footerCallback" ref="tradeItemWriter" />
</bean>
```


Java配置


```java
@Bean
public TradeItemWriter tradeItemWriter() {
        TradeItemWriter itemWriter = new TradeItemWriter();

        itemWriter.setDelegate(flatFileItemWriter(null));

        return itemWriter;
}

@Bean
public FlatFileItemWriter<String> flatFileItemWriter(Resource outputResource) {
        return new FlatFileItemWriterBuilder<String>()
                        .name("itemWriter")
                        .resource(outputResource)
                        .lineAggregator(lineAggregator())
                        .footerCallback(tradeItemWriter())
                        .build();
}
```


到目前为止， `TradeItemWriter` 的编写方式只有在 `Step` 不可重启时才能正常工作 . 这是因为该类是有状态的（因为它存储 `totalAmount` ），但 `totalAmount` 不会持久保存到数据库 . 因此，在重新启动时无法检索它 . 为了使此类可重新启动， `ItemStream` 接口应与方法 `open` 和 `update` 一起实现，如以下示例所示：


```java
public void open(ExecutionContext executionContext) {
    if (executionContext.containsKey("total.amount") {
        totalAmount = (BigDecimal) executionContext.get("total.amount");
    }
}

public void update(ExecutionContext executionContext) {
    executionContext.put("total.amount", totalAmount);
}
```


在将该对象持久保存到数据库之前，update方法将 `totalAmount` 的最新版本存储到 `ExecutionContext`  .  open方法从 `ExecutionContext` 中检索任何现有的 `totalAmount` 并将其用作处理的起始点，允许 `TradeItemWriter` 在上次运行 `Step` 时停止的地方重新启动 . 


### 1.4.基于驱动查询的ItemReaders

在[chapter on readers and writers](readersAndWriters.html)中，讨论了使用分页的数据库输入 . 许多数据库供应商（例如DB2）具有非常悲观的锁定策略，如果正在读取的表也需要由在线应用程序的其他部分使用，则可能导致问题 . 此外，在非常大的数据集上打开游标可能会导致某些供应商的数据库出现问题 . 因此，许多项目更喜欢使用“驾驶查询”方法来读取数据 . 这种方法通过迭代键而不是需要返回的整个对象来工作，如下图所示：


![Driving Query Job](https://www.docs4dev.com/images/1d48690f-25bb-457c-a2dd-79db3f3493df.png)


图1.驱动查询作业

如您所见，上图中显示的示例使用与基于游标的示例中使用的相同的“FOO”表 . 但是，不是选择整行，而是在SQL语句中仅选择了ID . 因此，不是从 `read` 返回 `FOO` 对象，而是返回 `Integer`  . 然后可以使用此数字查询“详细信息”，这是一个完整的 `Foo` 对象，如下图所示：


![Driving Query Example](https://www.docs4dev.com/images/d6e106d4-63ac-4171-9b3e-6d3db10c6935.png)


图2.驱动查询示例

应该使用 `ItemProcessor` 将从驱动查询获得的密钥转换为完整的'Foo'对象 . 现有的DAO可用于根据密钥查询完整对象 . 


### 1.5.多行记录

虽然平面文件通常是每个记录仅限于一行的情况，但通常文件可能包含跨多行多行的记录 . 以下文件摘录显示了这种安排的一个例子：


```java
HEA;0013100345;2007-02-15
NCU;Smith;Peter;;T;20014539;F
BAD;;Oak Street 31/A;;Small Town;00235;IL;US
FOT;2;2;267.34
```


以'HEA'开头的行和以'FOT'开头的行之间的所有内容都被视为一条记录 . 为了正确处理这种情况，必须考虑几个因素：


- 
 `ItemReader` 不必一次读取一条记录，而必须将多行记录的每一行作为一组读取，以便它可以完整地传递给 `ItemWriter`  . 


- 
每种线型可能需要以不同方式进行标记 . 

因为单个记录跨越多行并且因为我们可能不知道有多少行，所以_15288必须小心地始终读取整个记录 . 为此，应将自定义 `ItemReader` 实现为 `FlatFileItemReader` 的包装器，如以下示例所示：

XML配置


```xml
<bean id="itemReader" class="org.spr...MultiLineTradeItemReader">
    <property name="delegate">
        <bean class="org.springframework.batch.item.file.FlatFileItemReader">
            <property name="resource" value="data/iosample/input/multiLine.txt" />
            <property name="lineMapper">
                <bean class="org.spr...DefaultLineMapper">
                    <property name="lineTokenizer" ref="orderFileTokenizer"/>
                    <property name="fieldSetMapper" ref="orderFieldSetMapper"/>
                </bean>
            </property>
        </bean>
    </property>
</bean>
```


Java配置


```java
@Bean
public MultiLineTradeItemReader itemReader() {
        MultiLineTradeItemReader itemReader = new MultiLineTradeItemReader();

        itemReader.setDelegate(flatFileItemReader());

        return itemReader;
}

@Bean
public FlatFileItemReader flatFileItemReader() {
        FlatFileItemReader<Trade> reader = new FlatFileItemReaderBuilder<Trade>()
                        .name("flatFileItemReader")
                        .resource(new ClassPathResource("data/iosample/input/multiLine.txt"))
                        .lineTokenizer(orderFileTokenizer())
                        .fieldSetMapper(orderFieldSetMapper())
                        .build();
        return reader;
}
```


为了确保正确地标记每一行，这对于固定长度输入尤为重要， `PatternMatchingCompositeLineTokenizer` 可以在委托 `FlatFileItemReader` 上使用 . 有关详细信息，请参阅[FlatFileItemReader in the Readers and Writers chapter](readersAndWriters.html#flatFileItemReader) . 然后，委托阅读器使用 `PassThroughFieldSetMapper` 将每行返回 `FieldSet` 返回到包装 `ItemReader` ，如以下示例所示：

XML内容


```xml
<bean id="orderFileTokenizer" class="org.spr...PatternMatchingCompositeLineTokenizer">
    <property name="tokenizers">
        <map>
            <entry key="HEA*" value-ref="headerRecordTokenizer" />
            <entry key="FOT*" value-ref="footerRecordTokenizer" />
            <entry key="NCU*" value-ref="customerLineTokenizer" />
            <entry key="BAD*" value-ref="billingAddressLineTokenizer" />
        </map>
    </property>
</bean>
```


Java内容


```java
@Bean
public PatternMatchingCompositeLineTokenizer orderFileTokenizer() {
        PatternMatchingCompositeLineTokenizer tokenizer =
                        new PatternMatchingCompositeLineTokenizer();

        Map<String, LineTokenizer> tokenizers = new HashMap<>(4);

        tokenizers.put("HEA*", headerRecordTokenizer());
        tokenizers.put("FOT*", footerRecordTokenizer());
        tokenizers.put("NCU*", customerLineTokenizer());
        tokenizers.put("BAD*", billingAddressLineTokenizer());

        tokenizer.setTokenizers(tokenizers);

        return tokenizer;
}
```


此包装器必须能够识别记录的结尾，以便它可以在其委托上不断调用 `read()` ，直到到达结尾 . 对于每个读取的行，包装器应该构建要返回的项 . 到达页脚后，可以返回该项目以传递到 `ItemProcessor` 和 `ItemWriter` ，如以下示例所示：


```java
private FlatFileItemReader<FieldSet> delegate;

public Trade read() throws Exception {
    Trade t = null;

    for (FieldSet line = null; (line = this.delegate.read()) != null;) {
        String prefix = line.readString(0);
        if (prefix.equals("HEA")) {
            t = new Trade(); // Record must start with header
        }
        else if (prefix.equals("NCU")) {
            Assert.notNull(t, "No header was found.");
            t.setLast(line.readString(1));
            t.setFirst(line.readString(2));
            ...
        }
        else if (prefix.equals("BAD")) {
            Assert.notNull(t, "No header was found.");
            t.setCity(line.readString(4));
            t.setState(line.readString(6));
          ...
        }
        else if (prefix.equals("FOT")) {
            return t; // Record must end with footer
        }
    }
    Assert.isNull(t, "No 'END' was found.");
    return null;
}
```



### 1.6.执行系统命令

许多批处理作业都要求从批处理作业中调用外部命令 . 这样的过程可以由调度程序单独启动，但是关于运行的公共元数据的优点将会丢失 . 此外，多步骤工作也需要分成多个工作 . 

由于需求如此常见，Spring Batch为调用系统命令提供了 `Tasklet` 实现，如以下示例所示：

XML组态


```xml
<bean class="org.springframework.batch.core.step.tasklet.SystemCommandTasklet">
    <property name="command" value="echo hello" />
    <!-- 5 second timeout for the command to complete -->
    <property name="timeout" value="5000" />
</bean>
```


Java配置


```java
@Bean
public SystemCommandTasklet tasklet() {
        SystemCommandTasklet tasklet = new SystemCommandTasklet();

        tasklet.setCommand("echo hello");
        tasklet.setTimeout(5000);

        return tasklet;
}
```



### 1.7.未找到输入时处理步骤完成

在许多批处理方案中，在数据库或文件中找不到要处理的行并不例外 .   `Step` 被简单地认为没有找到工作并且完成了0项读取 .  Spring Batch中提供的所有 `ItemReader` 实现都默认使用此方法 . 如果即使存在输入也没有写出任何内容（如果文件名称错误或出现类似问题，通常会发生这种情况），这可能会导致一些混淆 . 出于这个原因，应该检查元数据本身以确定框架发现处理的工作量 . 但是，如果没有输入被视为特殊情况怎么办？在这种情况下，以编程方式检查元数据中没有处理的项目并导致失败是最佳解决方案 . 因为这是一个常见的用例，所以Spring Batch为监听器提供了这个功能，如 `NoWorkFoundStepExecutionListener` 的类定义所示：


```java
public class NoWorkFoundStepExecutionListener extends StepExecutionListenerSupport {

    public ExitStatus afterStep(StepExecution stepExecution) {
        if (stepExecution.getReadCount() == 0) {
            return ExitStatus.FAILED;
        }
        return null;
    }

}
```


前面的 `StepExecutionListener` 在'afterStep'阶段检查 `StepExecution` 的 `readCount` 属性，以确定是否没有读取任何项目 . 如果是这种情况，则返回退出代码FAILED，表示 `Step` 应该失败 . 否则，返回 `null` ，这不会影响 `Step` 的状态 . 


### 1.8.将数据传递给未来步骤

将信息从一个步骤传递到另一个步骤通常很有用 . 这可以通过 `ExecutionContext` 来完成 . 问题是有两个 `ExecutionContexts` ：一个在 `Step` 级别，一个在 `Job` 级别 .   `Step`   `ExecutionContext` 只保留与步长一样长，而 `Job`   `ExecutionContext` 仍然通过整个 `Job`  . 另一方面， `Step`   `ExecutionContext` 在每次 `Step` 提交一个块时更新，而 `Job`   `ExecutionContext` 仅在每个 `Step` 结束时更新 . 

这种分离的结果是，当 `Step` 正在执行时，所有数据必须放在 `Step`   `ExecutionContext` 中 . 这样做可确保在 `Step` 运行时正确存储数据 . 如果数据存储到 `Job`   `ExecutionContext` ，则在 `Step` 执行期间不会保留数据 . 如果 `Step` 失败，则该数据将丢失 . 


```java
public class SavingItemWriter implements ItemWriter<Object> {
    private StepExecution stepExecution;

    public void write(List<? extends Object> items) throws Exception {
        // ...

        ExecutionContext stepContext = this.stepExecution.getExecutionContext();
        stepContext.put("someKey", someObject);
    }

    @BeforeStep
    public void saveStepExecution(StepExecution stepExecution) {
        this.stepExecution = stepExecution;
    }
}
```


要使数据可用于将来 `Steps` ，在步骤完成后，它必须是"promoted"到 `Job`   `ExecutionContext`  .  Spring Batch为此提供了 `ExecutionContextPromotionListener`  . 必须使用与必须提升的 `ExecutionContext` 中的数据相关的键配置侦听器 . 也可以选择配置退出代码模式列表，以便进行促销（ `COMPLETED` 是默认值） . 与所有侦听器一样，它必须在 `Step` 上注册，如以下示例所示：

XML配置


```xml
<job id="job1">
    <step id="step1">
        <tasklet>
            <chunk reader="reader" writer="savingWriter" commit-interval="10"/>
        </tasklet>
        <listeners>
            <listener ref="promotionListener"/>
        </listeners>
    </step>

    <step id="step2">
       ...
    </step>
</job>

<beans:bean id="promotionListener" class="org.spr....ExecutionContextPromotionListener">
    <beans:property name="keys">
        <list>
            <value>someKey</value>
        </list>
    </beans:property>
</beans:bean>
```


Java配置


```java
@Bean
public Job job1() {
        return this.jobBuilderFactory.get("job1")
                                .start(step1())
                                .next(step1())
                                .build();
}

@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(10)
                                .reader(reader())
                                .writer(savingWriter())
                                .listener(promotionListener())
                                .build();
}

@Bean
public ExecutionContextPromotionListener promotionListener() {
        ExecutionContextPromotionListener listener = new ExecutionContextPromotionListener();

        listener.setKeys(new String[] {"someKey" });

        return listener;
}
```


最后，必须从 `Job`   `ExecutionContext` 检索保存的值，如以下示例所示：


```java
public class RetrievingItemWriter implements ItemWriter<Object> {
    private Object someObject;

    public void write(List<? extends Object> items) throws Exception {
        // ...
    }

    @BeforeStep
    public void retrieveInterstepData(StepExecution stepExecution) {
        JobExecution jobExecution = stepExecution.getJobExecution();
        ExecutionContext jobContext = jobExecution.getExecutionContext();
        this.someObject = jobContext.get("someKey");
    }
}
```
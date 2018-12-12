## 1. ItemReaders和ItemWriters

XML Java

所有批处理都可以用最简单的形式描述为读取大量数据，执行某种类型的计算或转换，以及将结果写出来 .  Spring Batch提供了三个关键接口来帮助执行批量读取和写入： `ItemReader` ， `ItemProcessor` 和 `ItemWriter`  . 


### 1.1. ItemReader

虽然是一个简单的概念， `ItemReader` 是从许多不同类型的输入提供数据的手段 . 最一般的例子包括：


- 
平面文件：平面文件读取器读取平面文件中的数据行，该文件通常描述具有由文件中的固定位置定义的数据字段或由某些特殊字符（例如逗号）分隔的数据字段的记录 . 


- 
XML：XML  `ItemReaders` 进程XML独立于用于解析，映射和验证对象的技术 . 输入数据允许针对XSD架构验证XML文件 . 


- 
数据库：访问数据库资源以返回可以映射到对象进行处理的结果集 . 默认的SQL  `ItemReader` 实现调用 `RowMapper` 来返回对象，如果需要重新启动则跟踪当前行，存储基本统计信息，并提供稍后解释的一些事务增强功能 . 

还有更多的可能性，但我们将重点放在本章的基本内容上 . 可在[Appendix A](appendix.html#listOfReadersAndWriters)中找到所有可用 `ItemReader` 实现的完整列表 . 

 `ItemReader` 是通用输入操作的基本接口，如以下接口定义所示：


```java
public interface ItemReader<T> {

    T read() throws Exception, UnexpectedInputException, ParseException, NonTransientResourceException;

}
```


 `read` 方法定义了 `ItemReader` 最重要的 Contract  . 如果没有剩下的项目，则调用它返回一个项目或 `null`  . 项可能表示文件中的行，数据库中的行或XML文件中的元素 . 通常期望这些映射到可用的域对象（例如 `Trade` ， `Foo` 或其他），但 Contract 中没有要求这样做 . 

预计 `ItemReader` 接口的实现仅是前向的 . 但是，如果底层资源是事务性的（例如JMS队列），则调用 `read` 可能会在回滚方案中的后续调用中返回相同的逻辑项 . 值得注意的是， `ItemReader` 缺少要处理的项目不会导致异常被抛出 . 例如，配置了返回0结果的查询的数据库 `ItemReader` 在第一次调用read时返回 `null`  . 


### 1.2. ItemWriter

 `ItemWriter` 在功能上类似于 `ItemReader` 但具有反向操作 . 资源仍然需要定位，打开和关闭，但它们的不同之处在于 `ItemWriter` 写出来而不是读入 . 在数据库或队列的情况下，这些操作可能是插入，更新或发送 . 输出序列化的格式特定于每个批处理作业 . 

与 `ItemReader` 一样， `ItemWriter` 是一个相当通用的接口，如以下接口定义所示：


```java
public interface ItemWriter<T> {

    void write(List<? extends T> items) throws Exception;

}
```


与 `ItemReader` 上的 `read` 一样， `write` 提供 `ItemWriter` 的基本 Contract  . 它会尝试写出传入的项目列表，只要它是打开的 . 因为通常期望将项目“批处理”到一个块然后输出，所以接口接受项目列表而不是项目本身 . 在写出列表之后，可以在从write方法返回之前执行任何可能需要的刷新 . 例如，如果写入Hibernate DAO，则可以进行多次写入调用，每个项目一次 . 然后，编写器可以在返回之前在hibernate会话上调用 `flush`  . 


### 1.3. ItemProcessor

 `ItemReader` 和 `ItemWriter` 接口对于它们的特定任务都非常有用，但是如果要在写入之前插入业务逻辑怎么办？读取和写入的一个选项是使用复合模式：创建包含另一个 `ItemWriter` 的 `ItemWriter` 或包含另一个 `ItemReader` 的 `ItemReader`  . 以下代码显示了一个示例：


```java
public class CompositeItemWriter<T> implements ItemWriter<T> {

    ItemWriter<T> itemWriter;

    public CompositeItemWriter(ItemWriter<T> itemWriter) {
        this.itemWriter = itemWriter;
    }

    public void write(List<? extends T> items) throws Exception {
        //Add business logic here
       itemWriter.write(items);
    }

    public void setDelegate(ItemWriter<T> itemWriter){
        this.itemWriter = itemWriter;
    }
}
```


前面的类包含另一个 `ItemWriter` ，它在提供了一些业务逻辑后委托给它 . 这种模式也可以很容易地用于 `ItemReader` ，也许可以根据主 `ItemReader` 提供的输入获得更多的参考数据 . 如果您需要自己控制对 `write` 的调用，这也很有用 . 但是，如果你只想在实际写入之前“转换”传入的项目进行写入，那么你自己就不需要 `write`  . 您只需修改该项目即可 . 对于此场景，Spring Batch提供 `ItemProcessor` 接口，如以下界面所示定义：


```java
public interface ItemProcessor<I, O> {

    O process(I item) throws Exception;
}
```


 `ItemProcessor` 很简单 . 给定一个对象，转换它并返回另一个对象 . 提供的对象可以是也可以不是同一类型 . 关键是业务逻辑可以在流程中应用，完全由开发人员来创建逻辑 .   `ItemProcessor` 可以直接连接到一个步骤 . 例如，假设 `ItemReader` 提供了一个类型为 `Foo` 的类，并且在写出之前需要将其转换为类型 `Bar`  . 以下示例显示执行转换的 `ItemProcessor` ：


```java
public class Foo {}

public class Bar {
    public Bar(Foo foo) {}
}

public class FooProcessor implements ItemProcessor<Foo,Bar>{
    public Bar process(Foo foo) throws Exception {
        //Perform simple transformation, convert a Foo to a Bar
        return new Bar(foo);
    }
}

public class BarWriter implements ItemWriter<Bar>{
    public void write(List<? extends Bar> bars) throws Exception {
        //write bars
    }
}
```


在前面的示例中，有一个类 `Foo` ，一个类 `Bar` 和一个遵循 `ItemProcessor` 接口的类 `FooProcessor`  . 转换很简单，但任何类型的转换都可以在这里完成 .   `BarWriter` 写入 `Bar` 对象，如果提供任何其他类型则抛出异常 . 类似地，如果提供了除 `Foo` 之外的任何内容，则 `FooProcessor` 会抛出异常 . 然后可以将 `FooProcessor` 注入 `Step` ，如以下示例所示：

XML配置


```xml
<job id="ioSampleJob">
    <step name="step1">
        <tasklet>
            <chunk reader="fooReader" processor="fooProcessor" writer="barWriter"
                   commit-interval="2"/>
        </tasklet>
    </step>
</job>
```


Java配置


```java
@Bean
public Job ioSampleJob() {
        return this.jobBuilderFactory.get("ioSampleJOb")
                                .start(step1())
                                .end()
                                .build();
}

@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(fooReader())
                                .processor(fooProcessor())
                                .writer(barWriter())
                                .build();
}
```



#### 1.3.1.链接ItemProcessors

在许多场景中执行单个转换很有用，但是如果要将多个 `ItemProcessor` 实现“链接”在一起会怎么样？这可以使用前面提到的复合图案来完成 . 要更新先前的单个转换，例如， `Foo` 将转换为 `Bar` ，它将转换为 `Foobar` 并写出，如以下示例所示：


```java
public class Foo {}

public class Bar {
    public Bar(Foo foo) {}
}

public class Foobar {
    public Foobar(Bar bar) {}
}

public class FooProcessor implements ItemProcessor<Foo,Bar>{
    public Bar process(Foo foo) throws Exception {
        //Perform simple transformation, convert a Foo to a Bar
        return new Bar(foo);
    }
}

public class BarProcessor implements ItemProcessor<Bar,Foobar>{
    public Foobar process(Bar bar) throws Exception {
        return new Foobar(bar);
    }
}

public class FoobarWriter implements ItemWriter<Foobar>{
    public void write(List<? extends Foobar> items) throws Exception {
        //write items
    }
}
```


可以将 `FooProcessor` 和 `BarProcessor` “链接”在一起以得到 `Foobar` ，如以下示例所示：


```java
CompositeItemProcessor<Foo,Foobar> compositeProcessor =
                                      new CompositeItemProcessor<Foo,Foobar>();
List itemProcessors = new ArrayList();
itemProcessors.add(new FooTransformer());
itemProcessors.add(new BarTransformer());
compositeProcessor.setDelegates(itemProcessors);
```


与前面的示例一样，复合处理器可以配置为 `Step` ：

XML配置


```xml
<job id="ioSampleJob">
    <step name="step1">
        <tasklet>
            <chunk reader="fooReader" processor="compositeItemProcessor" writer="foobarWriter"
                   commit-interval="2"/>
        </tasklet>
    </step>
</job>

<bean id="compositeItemProcessor"
      class="org.springframework.batch.item.support.CompositeItemProcessor">
    <property name="delegates">
        <list>
            <bean class="..FooProcessor" />
            <bean class="..BarProcessor" />
        </list>
    </property>
</bean>
```


Java配置


```java
@Bean
public Job ioSampleJob() {
        return this.jobBuilderFactory.get("ioSampleJob")
                                .start(step1())
                                .end()
                                .build();
}

@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(fooReader())
                                .processor(compositeProcessor())
                                .writer(foobarWriter())
                                .build();
}

@Bean
public CompositeItemProcessor compositeProcessor() {
        List<ItemProcessor> delegates = new ArrayList<>(2);
        delegates.add(new FooProcessor());
        delegates.add(new BarProcessor());

        CompositeItemProcessor processor = new CompositeItemProcessor();

        processor.setDelegates(delegates);

        return processor;
}
```



#### 1.3.2.过滤记录

项目处理器的一个典型用途是在将记录传递给_13972之前过滤掉记录 . 过滤是一种与跳过不同的操作 . 跳过表示记录无效，而过滤只表示不应写入记录 . 

例如，考虑一个批处理作业，它读取包含三种不同类型记录的文件：要插入的记录，要更新的记录和要删除的记录 . 如果系统不支持记录删除，那么我们不希望将任何"delete"记录发送到 `ItemWriter`  . 但是，由于这些记录实际上并不是坏记录，我们希望将它们过滤掉而不是跳过它们 . 因此， `ItemWriter` 将仅收到"insert"和"update"条记录 . 

要过滤记录，可以从 `ItemProcessor` 返回 `null`  . 框架检测到结果是 `null` 并避免将该项添加到传递给 `ItemWriter` 的记录列表中 . 像往常一样，从 `ItemProcessor` 抛出的异常会导致跳过 . 


#### 1.3.3.容错

回滚块时，可以重新处理在读取期间缓存的项目 . 如果将步骤配置为容错（通常通过使用跳过或重试处理），则应使用幂等方式实现任何 `ItemProcessor`  . 通常，这将包括不对 `ItemProcessor` 的输入项执行任何更改，并且仅更新作为结果的实例 . 


### 1.4. ItemStream

 `ItemReaders` 和 `ItemWriters` 都很好地服务于他们的个人目的，但是他们两个人都有一个共同的担忧，那就是需要另一个界面 . 通常，作为批处理作业范围的一部分，需要打开，关闭读者和编写者，并且需要一种持久状态的机制 .   `ItemStream` 接口用于此目的，如以下示例所示：


```java
public interface ItemStream {

    void open(ExecutionContext executionContext) throws ItemStreamException;

    void update(ExecutionContext executionContext) throws ItemStreamException;

    void close() throws ItemStreamException;
}
```


在描述每种方法之前，我们应该提到 `ExecutionContext`  . 同时实现 `ItemStream` 的 `ItemReader` 的客户端应在调用 `read` 之前调用 `open` ，以便打开任何资源（如文件）或获取连接 . 类似的限制适用于实现 `ItemStream` 的 `ItemWriter`  . 如第2章所述，如果在 `ExecutionContext` 中找到预期数据，则可以使用它在初始状态以外的位置启动 `ItemReader` 或 `ItemWriter`  . 相反，调用 `close` 以确保在打开期间分配的任何资源都安全释放 .   `update` 主要被调用以确保当前被保持的任何状态被加载到提供的 `ExecutionContext` 中 . 在提交之前调用此方法，以确保在提交之前当前状态在数据库中持久存在 . 

在 `ItemStream` 的客户端是 `Step` （来自Spring Batch Core）的特殊情况下，为每个StepExecution创建一个 `ExecutionContext` ，以允许用户存储特定执行的状态，并期望在相同的情况下返回它 .   `JobInstance` 再次启动 . 对于那些熟悉Quartz的人来说，语义与Quartz  `JobDataMap` 非常相似 . 


### 1.5.委托模式并注册步骤

请注意， `CompositeItemWriter` 是委派模式的示例，这在Spring Batch中很常见 . 代表们自己也可以实施回调接口，例如 `StepListener`  . 如果他们这样做，并且如果他们作为 `Step` 中的 `Step` 的一部分与Spring Batch Core一起使用，那么他们几乎肯定需要使用 `Step` 手动注册 . 直接连接到 `Step` 的读取器，写入器或处理器如果实现 `ItemStream` 或 `StepListener` 接口，则会自动注册 . 但是，由于 `Step` 不知道委托，因此需要将它们作为侦听器或流（或两者都适当）注入，如以下示例所示：

XML配置


```xml
<job id="ioSampleJob">
    <step name="step1">
        <tasklet>
            <chunk reader="fooReader" processor="fooProcessor" writer="compositeItemWriter"
                   commit-interval="2">
                <streams>
                    <stream ref="barWriter" />
                </streams>
            </chunk>
        </tasklet>
    </step>
</job>

<bean id="compositeItemWriter" class="...CustomCompositeItemWriter">
    <property name="delegate" ref="barWriter" />
</bean>

<bean id="barWriter" class="...BarWriter" />
```


Java配置


```java
@Bean
public Job ioSampleJob() {
        return this.jobBuilderFactory.get("ioSampleJob")
                                .start(step1())
                                .end()
                                .build();
}

@Bean
public Step step1() {
        return this.stepBuilderFactory.get("step1")
                                .<String, String>chunk(2)
                                .reader(fooReader())
                                .processor(fooProcessor())
                                .writer(compositeItemWriter())
                                .stream(barWriter())
                                .build();
}

@Bean
public CustomCompositeItemWriter compositeItemWriter() {

        CustomCompositeItemWriter writer = new CustomCompositeItemWriter();

        writer.setDelegate(barWriter());

        return writer;
}

@Bean
public BarWriter barWriter() {
        return new BarWriter();
}
```



### 1.6.平面文件

交换批量数据的最常见机制之一一直是平面文件 . 与XML（用于定义其结构（XSD）的商定标准）不同，阅读平面文件的任何人都必须提前了解文件的结构 . 通常，所有平面文件分为两种类型：分隔和固定长度 . 分隔文件是字段由分隔符分隔的文件，例如逗号 . 固定长度文件具有设定长度的字段 . 


#### 1.6.1. FieldSet

在Spring Batch中处理平面文件时，无论是输入还是输出，最重要的类之一是 `FieldSet`  . 许多体系结构和库包含帮助您从文件读入的抽象，但它们通常返回 `String` 或 `String` 对象的数组 . 这真的只能让你到达那里 .   `FieldSet` 是Spring Batch的抽象，用于启用文件资源中字段的绑定 . 它允许开发人员使用文件输入，就像使用数据库输入一样 .   `FieldSet` 在概念上类似于JDBC  `ResultSet`  .   `FieldSet` 只需要一个参数：一个 `String` 标记数组 . 或者，您也可以配置字段的名称，以便在 `ResultSet` 之后可以按索引或名称访问字段，如以下示例所示：


```java
String[] tokens = new String[]{"foo", "1", "true"};
FieldSet fs = new DefaultFieldSet(tokens);
String name = fs.readString(0);
int value = fs.readInt(1);
boolean booleanValue = fs.readBoolean(2);
```


 `FieldSet` 界面上还有更多选项，例如 `Date` ，long， `BigDecimal` 等 .   `FieldSet` 的最大优点是它提供了对平面文件输入的一致解析 . 而不是每个批处理作业以可能意外的方式进行不同的解析，它在处理由格式异常引起的错误或进行简单的数据转换时都是一致的 . 


#### 1.6.2. FlatFileItemReader

平面文件是包含最多二维（表格）数据的任何类型的文件 . 在名为 `FlatFileItemReader` 的类中，可以在Spring Batch框架中读取平面文件，该类提供了读取和解析平面文件的基本功能 .   `FlatFileItemReader` 的两个最重要的必需依赖项是 `Resource` 和 `LineMapper`  .   `LineMapper` 接口将在下一节中详细介绍 .  resource属性表示Spring Core  `Resource`  . 可以在[Spring Framework, Chapter 5. Resources](https://docs.spring.io/spring/docs/current/spring-framework-reference/core.html#resources)中找到解释如何创建此类bean的文档 . 因此，除了显示以下简单示例之外，本指南不会详细介绍创建 `Resource` 对象：


```java
Resource resource = new FileSystemResource("resources/trades.csv");
```


在复杂的批处理环境中，目录结构通常由EAI基础结构管理，其中 Build 外部接口的删除区域，用于将文件从FTP位置移动到批处理位置，反之亦然 . 文件移动实用程序超出了Spring Batch体系结构的范围，但批处理作业流将文件移动实用程序作为作业流中的步骤包含在其中并不罕见 . 批处理体系结构只需要知道如何定位要处理的文件 .  Spring Batch开始从这个起点将数据输入管道的过程 . 但是，[Spring Integration](https://projects.spring.io/spring-integration/)提供了许多这类服务 . 

 `FlatFileItemReader` 中的其他属性允许您进一步指定数据的解释方式，如下表所述：

||
|地产|类型|说明|
| ---- | ---- | ---- |
| comments | String [] |指定表示注释行的行前缀 .  |
| encoding | String |指定要使用的文本编码 . 默认值为 `Charset.defaultCharset()`  .  |
| lineMapper |  `LineMapper`  |将 `String` 转换为表示该项目的 `Object`  .  |
| linesToSkip | int |文件顶部要忽略的行数 .  |
| recordSeparatorPolicy | RecordSeparatorPolicy |用于确定行结尾的位置，并执行诸如在带引号的字符串内的行结束之类的操作 .  |
| resource |  `Resource`  |要读取的资源 .  |
| skippedLinesCallback | LineCallbackHandler |传递要跳过的文件中行的原始行内容的接口 . 如果 `linesToSkip` 设置为2，则调用此接口两次 .  |
| strict | boolean |在严格模式下，如果输入资源不存在，则读取器会在 `ExecutionContext` 上抛出异常 . 否则，它会记录问题并继续 .  |


##### LineMapper

与 `RowMapper` 一样，它采用低级构造（如 `ResultSet` ）并返回 `Object` ，平面文件处理需要相同的构造来转换 `String` 行成 `Object` ，如下面的接口定义所示：


```java
public interface LineMapper<T> {

    T mapLine(String line, int lineNumber) throws Exception;

}
```


基本 Contract 是，给定当前行和与之关联的行号，映射器应返回结果域对象 . 这类似于 `RowMapper` ，因为每一行都与其行号相关联，就像 `ResultSet` 中的每一行都与其行号相关联一样 . 这允许将行号绑定到生成的域对象以进行身份比较或进行更详细的日志记录 . 然而，与 `RowMapper` 不同， `LineMapper` 被给予一条原始线，如上所述，它只会让你到达中途 . 必须将该行标记为 `FieldSet` ，然后可以将其映射到对象，如本文档后面所述 . 


##### LineTokenizer

将输入行转换为 `FieldSet` 的抽象是必要的，因为可能有许多格式的平面文件数据需要转换为 `FieldSet`  . 在Spring Batch中，此接口是 `LineTokenizer` ：


```java
public interface LineTokenizer {

    FieldSet tokenize(String line);

}
```


 `LineTokenizer` 的 Contract 是这样的，给定一行输入（理论上 `String` 可能包含多行），返回表示该行的 `FieldSet`  . 然后可以将 `FieldSet` 传递给 `FieldSetMapper`  .  Spring Batch包含以下 `LineTokenizer` 实现：


- 
 `DelimitedLineTokenizer` ：用于记录中的字段由分隔符分隔的文件 . 最常见的分隔符是逗号，但也经常使用管道或分号 . 


- 
 `FixedLengthTokenizer` ：用于记录中的字段均为"fixed width"的文件 . 必须为每种记录类型定义每个字段的宽度 . 


- 
 `PatternMatchingCompositeLineTokenizer` ：通过检查模式，确定应在特定行上使用哪个 `LineTokenizer`  . 


##### FieldSetMapper

 `FieldSetMapper` 接口定义了一个方法 `mapFieldSet` ，它接受一个 `FieldSet` 对象并将其内容映射到一个对象 . 此对象可以是自定义DTO，域对象或数组，具体取决于作业的需要 .   `FieldSetMapper` 与 `LineTokenizer` 结合使用，将一行数据从资源转换为所需类型的对象，如以下接口定义所示：


```java
public interface FieldSetMapper<T> {

    T mapFieldSet(FieldSet fieldSet) throws BindException;

}
```


使用的模式与 `JdbcTemplate` 使用的 `RowMapper` 相同 . 


##### DefaultLineMapper

现在已经定义了用于读取平面文件的基本接口，很明显需要三个基本步骤：

从文件中读取一行 . 将String行传递给LineTokenizer＃tokenize（）方法以检索FieldSet . 将从标记化返回的FieldSet传递给FieldSetMapper，从ItemReader＃read（）方法返回结果 . 

上述两个接口代表两个独立的任务：将一行转换为 `FieldSet` 并将 `FieldSet` 映射到域对象 . 由于 `LineTokenizer` 的输入与 `LineMapper` （一行）的输入匹配，并且 `FieldSetMapper` 的输出与 `LineMapper` 的输出匹配，因此提供了同时使用 `LineTokenizer` 和 `FieldSetMapper` 的默认实现 .   `DefaultLineMapper` ，如以下类定义所示，表示大多数用户需要的行为：


```java
public class DefaultLineMapper<T> implements LineMapper<>, InitializingBean {

    private LineTokenizer tokenizer;

    private FieldSetMapper<T> fieldSetMapper;

    public T mapLine(String line, int lineNumber) throws Exception {
        return fieldSetMapper.mapFieldSet(tokenizer.tokenize(line));
    }

    public void setLineTokenizer(LineTokenizer tokenizer) {
        this.tokenizer = tokenizer;
    }

    public void setFieldSetMapper(FieldSetMapper<T> fieldSetMapper) {
        this.fieldSetMapper = fieldSetMapper;
    }
}
```


上述功能在默认实现中提供，而不是内置到读取器本身（如在框架的先前版本中所做的那样），以允许用户更灵活地控制解析过程，尤其是在需要访问原始行时 . 


##### Simple分隔文件读取示例

以下示例说明如何使用实际域方案读取平面文件 . 此特定批处理作业从以下文件中读取足球运动员：


```java
ID,lastName,firstName,position,birthYear,debutYear
"AbduKa00,Abdul-Jabbar,Karim,rb,1974,1996",
"AbduRa00,Abdullah,Rabih,rb,1975,1999",
"AberWa00,Abercrombie,Walter,rb,1959,1982",
"AbraDa00,Abramowicz,Danny,wr,1945,1967",
"AdamBo00,Adams,Bob,te,1946,1969",
"AdamCh00,Adams,Charlie,wr,1979,2003"
```


此文件的内容映射到以下 `Player` 域对象：


```java
public class Player implements Serializable {

    private String ID;
    private String lastName;
    private String firstName;
    private String position;
    private int birthYear;
    private int debutYear;

    public String toString() {
        return "PLAYER:ID=" + ID + ",Last Name=" + lastName +
            ",First Name=" + firstName + ",Position=" + position +
            ",Birth Year=" + birthYear + ",DebutYear=" +
            debutYear;
    }

    // setters and getters...
}
```


要将 `FieldSet` 映射到 `Player` 对象，需要定义返回玩家的 `FieldSetMapper` ，如以下示例所示：


```java
protected static class PlayerFieldSetMapper implements FieldSetMapper<Player> {
    public Player mapFieldSet(FieldSet fieldSet) {
        Player player = new Player();

        player.setID(fieldSet.readString(0));
        player.setLastName(fieldSet.readString(1));
        player.setFirstName(fieldSet.readString(2));
        player.setPosition(fieldSet.readString(3));
        player.setBirthYear(fieldSet.readInt(4));
        player.setDebutYear(fieldSet.readInt(5));

        return player;
    }
}
```


然后可以通过正确构造 `FlatFileItemReader` 并调用 `read` 来读取该文件，如以下示例所示：


```java
FlatFileItemReader<Player> itemReader = new FlatFileItemReader<Player>();
itemReader.setResource(new FileSystemResource("resources/players.csv"));
//DelimitedLineTokenizer defaults to comma as its delimiter
DefaultLineMapper<Player> lineMapper = new DefaultLineMapper<Player>();
lineMapper.setLineTokenizer(new DelimitedLineTokenizer());
lineMapper.setFieldSetMapper(new PlayerFieldSetMapper());
itemReader.setLineMapper(lineMapper);
itemReader.open(new ExecutionContext());
Player player = itemReader.read();
```


每次调用 `read` 都会从文件中的每一行返回一个新的 `Player` 对象 . 到达文件末尾时，返回 `null`  . 

按名称
##### Mapping字段

 `DelimitedLineTokenizer` 和 `FixedLengthTokenizer` 允许另外一项功能，它在功能上类似于JDBC  `ResultSet`  . 可以将这些字段的名称注入到这些 `LineTokenizer` 实现中的任何一个中，以增加映射函数的可读性 . 首先，将平面文件中所有字段的列名注入到tokenizer中，如以下示例所示：


```java
tokenizer.setNames(new String[] {"ID", "lastName","firstName","position","birthYear","debutYear"});
```


 `FieldSetMapper` 可以使用以下信息：


```java
public class PlayerMapper implements FieldSetMapper<Player> {
    public Player mapFieldSet(FieldSet fs) {

       if(fs == null){
           return null;
       }

       Player player = new Player();
       player.setID(fs.readString("ID"));
       player.setLastName(fs.readString("lastName"));
       player.setFirstName(fs.readString("firstName"));
       player.setPosition(fs.readString("position"));
       player.setDebutYear(fs.readInt("debutYear"));
       player.setBirthYear(fs.readInt("birthYear"));

       return player;
   }
}
```



##### Automapping FieldSets到域对象

对于许多人来说，必须编写特定的 `FieldSetMapper` 与为 `JdbcTemplate` 编写特定的 `RowMapper` 一样繁琐 .  Spring Batch通过提供 `FieldSetMapper` 来使这更容易，它通过使用JavaBean规范将字段名称与对象上的setter匹配来自动映射字段 . 再次使用足球示例， `BeanWrapperFieldSetMapper` 配置看起来像以下代码段：

XML组态


```xml
<bean id="fieldSetMapper"
      class="org.springframework.batch.item.file.mapping.BeanWrapperFieldSetMapper">
    <property name="prototypeBeanName" value="player" />
</bean>

<bean id="player"
      class="org.springframework.batch.sample.domain.Player"
      scope="prototype" />
```


Java配置


```java
@Bean
public FieldSetMapper fieldSetMapper() {
        BeanWrapperFieldSetMapper fieldSetMapper = new BeanWrapperFieldSetMapper();

        fieldSetMapper.setPrototypeBeanName("player");

        return fieldSetMapper;
}

@Bean
@Scope("prototype")
public Player player() {
        return new Player();
}
```


对于 `FieldSet` 中的每个条目，映射器在 `Player` 对象的新实例上查找相应的setter（因此，需要原型范围），就像Spring容器查找与属性名称匹配的setter一样 . 映射 `FieldSet` 中的每个可用字段，并返回结果 `Player` 对象，不需要代码 . 


##### Fixed Length File Formats

到目前为止，只详细讨论了分隔文件 . 但是，它们只占文件阅读图片的一半 . 许多使用平面文件的组织使用固定长度格式 . 下面是一个示例固定长度文件：


```java
UK21341EAH4121131.11customer1
UK21341EAH4221232.11customer2
UK21341EAH4321333.11customer3
UK21341EAH4421434.11customer4
UK21341EAH4521535.11customer5
```


虽然这看起来像一个大字段，但它实际上代表了4个不同的字段：

ISIN：订购商品的唯一标识符 - 长度为12个字符 . 数量：订购商品的数量 - 长度为3个字符 . 价格：该商品的价格 -  5个字符长 . 客户：订购商品的客户的ID  - 长度为9个字符 . 

配置 `FixedLengthLineTokenizer` 时，必须以范围的形式提供每个长度，如以下示例所示：

XML配置


```xml
<bean id="fixedLengthLineTokenizer"
      class="org.springframework.batch.io.file.transform.FixedLengthTokenizer">
    <property name="names" value="ISIN,Quantity,Price,Customer" />
    <property name="columns" value="1-12, 13-15, 16-20, 21-29" />
</bean>
```


因为 `FixedLengthLineTokenizer` 使用与上面讨论的相同的 `LineTokenizer` 接口，所以它返回相同的 `FieldSet` ，就好像使用了分隔符一样 . 这允许在处理其输出时使用相同的方法，例如使用 `BeanWrapperFieldSetMapper`  . 


> 

支持范围的上述语法要求在 `ApplicationContext` 中配置专用属性编辑器 `RangeArrayPropertyEditor`  . 但是，此bean在 `ApplicationContext` 中自动声明，其中使用批处理命名空间 . 

Java配置


```java
@Bean
public FixedLengthTokenizer fixedLengthTokenizer() {
        FixedLengthTokenizer tokenizer = new FixedLengthTokenizer();

        tokenizer.setNames("ISIN", "Quantity", "Price", "Customer");
        tokenizer.setColumns(new Range(1-12),
                                                new Range(13-15),
                                                new Range(16-20),
                                                new Range(21-29));

        return tokenizer;
}
```


因为 `FixedLengthLineTokenizer` 使用与上面讨论的相同的 `LineTokenizer` 接口，所以它返回相同的 `FieldSet` ，就好像使用了分隔符一样 . 这允许在处理其输出时使用相同的方法，例如使用 `BeanWrapperFieldSetMapper`  . 


##### 单个文件中的多个记录类型

到目前为止，所有文件阅读示例都是为了简单起见而做出的关键假设：文件中的所有记录都具有相同的格式 . 但是，情况可能并非总是如此 . 一个文件可能具有不同格式的记录是非常常见的，这些记录需要以不同的方式进行标记并映射到不同的对象 . 以下文件摘录说明了这一点：


```java
USER;Smith;Peter;;T;20014539;F
LINEA;1044391041ABC037.49G201XX1383.12H
LINEB;2134776319DEF422.99M005LI
```


在此文件中，我们有三种类型的记录，"USER"，"LINEA"和"LINEB" .  "USER"行对应于 `User` 对象 .  "LINEA"和"LINEB"都对应于 `Line` 对象，但"LINEA"的信息多于"LINEB" . 

 `ItemReader` 单独读取每一行，但我们必须指定不同的 `LineTokenizer` 和 `FieldSetMapper` 对象，以便 `ItemWriter` 接收正确的项目 .   `PatternMatchingCompositeLineMapper` 通过允许配置模式到 `LineTokenizer` 实例和模式到 `FieldSetMapper` 实例的映射使这变得容易，如以下示例所示：

XML配置


```xml
<bean id="orderFileLineMapper"
      class="org.spr...PatternMatchingCompositeLineMapper">
    <property name="tokenizers">
        <map>
            <entry key="USER*" value-ref="userTokenizer" />
            <entry key="LINEA*" value-ref="lineATokenizer" />
            <entry key="LINEB*" value-ref="lineBTokenizer" />
        </map>
    </property>
    <property name="fieldSetMappers">
        <map>
            <entry key="USER*" value-ref="userFieldSetMapper" />
            <entry key="LINE*" value-ref="lineFieldSetMapper" />
        </map>
    </property>
</bean>
```


Java配置


```java
@Bean
public PatternMatchingCompositeLineMapper orderFileLineMapper() {
        PatternMatchingCompositeLineMapper lineMapper =
                new PatternMatchingCompositeLineMapper();

        Map<String, LineTokenizer> tokenizers = new HashMap<>(3);
        tokenizers.put("USER*", userTokenizer());
        tokenizers.put("LINEA*", lineATokenizer());
        tokenizers.put("LINEB*", lineBTokenizer());

        lineMapper.setTokenizers(tokenizers);

        Map<String, FieldSetMapper> mappers = new HashMap<>(2);
        mappers.put("USER*", userFieldSetMapper());
        mappers.put("LINE*", lineFieldSetMapper());

        lineMapper.setFieldSetMappers(mappers);

        return lineMapper;
}
```


在此示例中，"LINEA"和"LINEB"具有单独的 `LineTokenizer` 实例，但它们都使用相同的 `FieldSetMapper`  . 

 `PatternMatchingCompositeLineMapper` 使用 `PatternMatcher#match` 方法为每行选择正确的委托 .  `PatternMatcher` 允许两个具有特殊含义的通配符：问号（"?"）恰好匹配一个字符，而星号（"*"）匹配零个或多个字符 . 请注意，在前面的配置中，所有模式都以星号结尾，使它们成为行的有效前缀 . 无论配置中的顺序如何， `PatternMatcher` 始终匹配可能的最具体模式 . 因此，如果"LINE*"和"LINEA*"都列为模式，"LINEA"将匹配模式"LINEA*"，而"LINEB"将匹配模式"LINE*" . 此外，单个星号（"*"）可以通过匹配任何其他模式不匹配的行来作为默认值，如以下示例所示 . 

XML配置


```xml
<entry key="*" value-ref="defaultLineTokenizer" />
```


Java配置


```java
...
tokenizers.put("*", defaultLineTokenizer());
...
```


还有一个 `PatternMatchingCompositeLineTokenizer` 可以单独用于标记化 . 

平面文件通常包含每个跨越多行的记录 . 要处理这种情况，需要采用更复杂的策略 . 可以在 `multiLineRecords` 样本中找到这种常见模式的演示 . 

平面文件中的
##### Exception处理

标记行可能会导致抛出异常，有许多情况 . 许多平面文件不完美，包含格式不正确的记录 . 许多用户在记录问题，原始行和行号时选择跳过这些错误行 . 稍后可以手动或通过其他批处理作业检查这些日志 . 因此，Spring Batch提供了处理解析异常的异常层次结构： `FlatFileParseException` 和 `FlatFileFormatException`  . 当尝试读取文件时遇到任何错误时， `FlatFileItemReader` 会抛出 `FlatFileParseException`  .   `LineTokenizer` 接口的实现抛出 `FlatFileFormatException` ，表示在标记化时遇到更具体的错误 . 


###### IncorrectTokenCountException

 `DelimitedLineTokenizer` 和 `FixedLengthLineTokenizer` 都有能够指定可用于创建 `FieldSet` 的列名 . 但是，如果列名称的数量与标记行时找到的列数不匹配，则无法创建 `FieldSet` ，并抛出 `IncorrectTokenCountException` ，其中包含遇到的标记数和预期的数量，如以下示例：


```java
tokenizer.setNames(new String[] {"A", "B", "C", "D"});

try {
    tokenizer.tokenize("a,b,c");
}
catch(IncorrectTokenCountException e){
    assertEquals(4, e.getExpectedCount());
    assertEquals(3, e.getActualCount());
}
```


由于tokenizer配置了4个列名，但在文件中只找到3个令牌，因此抛出了 `IncorrectTokenCountException`  . 


###### IncorrectLineLengthException

在解析时，以固定长度格式格式化的文件具有其他要求，因为与分隔格式不同，每列必须严格遵守其预定义的宽度 . 如果总行长度不等于此列的最大值，则抛出异常，如以下示例所示：


```java
tokenizer.setColumns(new Range[] { new Range(1, 5),
                                   new Range(6, 10),
                                   new Range(11, 15) });
try {
    tokenizer.tokenize("12345");
    fail("Expected IncorrectLineLengthException");
}
catch (IncorrectLineLengthException ex) {
    assertEquals(15, ex.getExpectedLength());
    assertEquals(5, ex.getActualLength());
}
```


上面的tokenizer的配置范围是：1-5,6-10和11-15 . 因此，该行的总长度为15.但是，在前面的示例中，传入了长度为5的行，导致 `IncorrectLineLengthException` 被抛出 . 在此处抛出异常而不是仅映射第一列允许线路的处理更早失败，并且在尝试读取 `FieldSetMapper` 中的第2列时，如果失败则包含更多信息 . 但是，有些情况下，线的长度并不总是恒定的 . 因此，可以通过'strict'属性关闭行长度验证，如以下示例所示：


```java
tokenizer.setColumns(new Range[] { new Range(1, 5), new Range(6, 10) });
tokenizer.setStrict(false);
FieldSet tokens = tokenizer.tokenize("12345");
assertEquals("12345", tokens.readString(0));
assertEquals("", tokens.readString(1));
```


前面的示例几乎与之前的示例相同，只是调用了 `tokenizer.setStrict(false)`  . 此设置告诉标记生成器在标记行时不强制执行行长度 . 现在可以正确创建并返回 `FieldSet`  . 但是，它仅包含剩余值的空标记 . 


#### 1.6.3. FlatFileItemWriter

写出平面文件有同样的问题和问题，从文件读入必须克服 . 步骤必须能够以事务方式编写分隔或固定长度格式 . 


##### LineAggregator

正如 `LineTokenizer` 接口是获取项目并将其转换为 `String` 所必需的一样，文件编写必须能够将多个字段聚合为单个字符串以写入文件 . 在Spring Batch中，这是 `LineAggregator` ，如以下接口定义所示：


```java
public interface LineAggregator<T> {

    public String aggregate(T item);

}
```


 `LineAggregator` 与 `LineTokenizer` 的逻辑相反 .   `LineTokenizer` 采用 `String` 并返回 `FieldSet` ，而 `LineAggregator` 采用 `item` 并返回 `String`  . 


###### PassThroughLineAggregator

 `LineAggregator` 接口的最基本实现是 `PassThroughLineAggregator` ，它假定该对象已经是一个字符串或者其字符串表示形式可以写入，如下面的代码所示：


```java
public class PassThroughLineAggregator<T> implements LineAggregator<T> {

    public String aggregate(T item) {
        return item.toString();
    }
}
```


如果需要直接控制创建字符串但是 `FlatFileItemWriter` 的优点（例如事务和重启支持）是必要的，则前面的实现很有用 . 


##### Simplified文件编写示例

现在已经定义了 `LineAggregator` 接口及其最基本的实现 `PassThroughLineAggregator` ，可以解释基本的写入流程：

要写入的对象将传递给LineAggregator以获取String . 返回的String将写入配置的文件 . 

以下摘录自 `FlatFileItemWriter` 在代码中表达了这一点：


```java
public void write(T item) throws Exception {
    write(lineAggregator.aggregate(item) + LINE_SEPARATOR);
}
```


简单配置可能如下所示：

XML配置


```xml
<bean id="itemWriter" class="org.spr...FlatFileItemWriter">
    <property name="resource" value="file:target/test-outputs/output.txt" />
    <property name="lineAggregator">
        <bean class="org.spr...PassThroughLineAggregator"/>
    </property>
</bean>
```


Java配置


```java
@Bean
public FlatFileItemWriter itemWriter() {
        return  new FlatFileItemWriterBuilder<Foo>()
                                   .name("itemWriter")
                                   .resource(new FileSystemResource("target/test-outputs/output.txt"))
                                   .lineAggregator(new PassThroughLineAggregator<>())
                                   .build();
}
```



##### FieldExtractor

前面的示例可能对写入文件的最基本用法很有用 . 但是， `FlatFileItemWriter` 的大多数用户都有一个需要写出的域对象，因此必须转换为一行 . 在文件阅读中，需要以下内容：

从文件中读取一行 . 将该行传递给LineTokenizer＃tokenize（）方法，以便检索FieldSet . 将从标记化返回的FieldSet传递给FieldSetMapper，从ItemReader＃read（）方法返回结果 . 

文件写入有类似但相反的步骤：

将要写入的项目传递给作者 . 将项目上的字段转换为数组 . 将结果数组聚合成一行 . 

因为框架无法知道需要写出对象中的哪些字段，所以必须编写 `FieldExtractor` 来完成将项目转换为数组的任务，如以下接口定义所示：


```java
public interface FieldExtractor<T> {

    Object[] extract(T item);

}
```


 `FieldExtractor` 接口的实现应该从提供的对象的字段创建一个数组，然后可以使用元素之间的分隔符或作为固定宽度线的一部分写出 . 


###### PassThroughFieldExtractor

在许多情况下，需要写出一个集合，例如数组， `Collection` 或 `FieldSet`  .  "Extracting"来自其中一种集合类型的数组非常简单 . 为此，请将集合转换为数组 . 因此，应在此方案中使用 `PassThroughFieldExtractor`  . 应该注意的是，如果传入的对象不是一种集合，那么 `PassThroughFieldExtractor` 返回仅包含要提取的项的数组 . 


###### BeanWrapperFieldExtractor

与文件读取部分中描述的 `BeanWrapperFieldSetMapper` 一样，通常最好配置如何将域对象转换为对象数组，而不是自己编写转换 .   `BeanWrapperFieldExtractor` 提供此功能，如以下示例所示：


```java
BeanWrapperFieldExtractor<Name> extractor = new BeanWrapperFieldExtractor<Name>();
extractor.setNames(new String[] { "first", "last", "born" });

String first = "Alan";
String last = "Turing";
int born = 1912;

Name n = new Name(first, last, born);
Object[] values = extractor.extract(n);

assertEquals(first, values[0]);
assertEquals(last, values[1]);
assertEquals(born, values[2]);
```


此提取器实现只有一个必需属性：要映射的字段的名称 . 就像 `BeanWrapperFieldSetMapper` 需要字段名称来将 `FieldSet` 上的字段映射到提供的对象上的setter一样， `BeanWrapperFieldExtractor` 需要将名称映射到用于创建对象数组的getter . 值得注意的是，名称的顺序决定了数组中字段的顺序 . 


##### Delimited文件编写示例

最基本的平面文件格式是所有字段由分隔符分隔的格式 . 这可以使用 `DelimitedLineAggregator` 来完成 . 以下示例写出一个简单的域对象，该对象表示对客户帐户的信用：


```java
public class CustomerCredit {

    private int id;
    private String name;
    private BigDecimal credit;

    //getters and setters removed for clarity
}
```


由于正在使用域对象，因此必须提供 `FieldExtractor` 接口的实现以及要使用的分隔符，如以下示例所示：

XML配置


```xml
<bean id="itemWriter" class="org.springframework.batch.item.file.FlatFileItemWriter">
    <property name="resource" ref="outputResource" />
    <property name="lineAggregator">
        <bean class="org.spr...DelimitedLineAggregator">
            <property name="delimiter" value=","/>
            <property name="fieldExtractor">
                <bean class="org.spr...BeanWrapperFieldExtractor">
                    <property name="names" value="name,credit"/>
                </bean>
            </property>
        </bean>
    </property>
</bean>
```


Java配置


```java
@Bean
public FlatFileItemWriter<CustomerCredit> itemWriter(Resource outputResource) throws Exception {
        BeanWrapperFieldExtractor<CustomerCredit> fieldExtractor = new BeanWrapperFieldExtractor<>();
        fieldExtractor.setNames(new String[] {"name", "credit"});
        fieldExtractor.afterPropertiesSet();

        DelimitedLineAggregator<CustomerCredit> lineAggregator = new DelimitedLineAggregator<>();
        lineAggregator.setDelimiter(",");
        lineAggregator.setFieldExtractor(fieldExtractor);

        return new FlatFileItemWriterBuilder<CustomerCredit>()
                                .name("customerCreditWriter")
                                .resource(outputResource)
                                .lineAggregator(lineAggregator)
                                .build();
}
```


在前面的示例中，本章前面介绍的 `BeanWrapperFieldExtractor` 用于将 `CustomerCredit` 中的名称和信用字段转换为对象数组，然后在每个字段之间用逗号写出 . 

也可以使用 `FlatFileItemWriterBuilder.DelimitedBuilder` 自动创建 `BeanWrapperFieldExtractor` 和 `DelimitedLineAggregator` ，如以下示例所示：

Java配置


```java
@Bean
public FlatFileItemWriter<CustomerCredit> itemWriter(Resource outputResource) throws Exception {
        return new FlatFileItemWriterBuilder<CustomerCredit>()
                                .name("customerCreditWriter")
                                .resource(outputResource)
                                .delimited()
                                .delimiter("|")
                                .names(new String[] {"name", "credit"})
                                .build();
}
```



##### Fixed宽度文件编写示例

分隔不是唯一的平面文件格式 . 许多人更喜欢使用每列的设定宽度来描绘字段之间，这通常被称为“固定宽度” .  Spring Batch在使用 `FormatterLineAggregator` 进行文件写入时支持此功能 . 使用上述相同的 `CustomerCredit` 域对象，可以按如下方式配置：

XML配置


```xml
<bean id="itemWriter" class="org.springframework.batch.item.file.FlatFileItemWriter">
    <property name="resource" ref="outputResource" />
    <property name="lineAggregator">
        <bean class="org.spr...FormatterLineAggregator">
            <property name="fieldExtractor">
                <bean class="org.spr...BeanWrapperFieldExtractor">
                    <property name="names" value="name,credit" />
                </bean>
            </property>
            <property name="format" value="%-9s%-2.0f" />
        </bean>
    </property>
</bean>
```


Java配置


```java
@Bean
public FlatFileItemWriter<CustomerCredit> itemWriter(Resource outputResource) throws Exception {
        BeanWrapperFieldExtractor<CustomerCredit> fieldExtractor = new BeanWrapperFieldExtractor<>();
        fieldExtractor.setNames(new String[] {"name", "credit"});
        fieldExtractor.afterPropertiesSet();

        FormatterLineAggregator<CustomerCredit> lineAggregator = new FormatterLineAggregator<>();
        lineAggregator.setFormat("%-9s%-2.0f");
        lineAggregator.setFieldExtractor(fieldExtractor);

        return new FlatFileItemWriterBuilder<CustomerCredit>()
                                .name("customerCreditWriter")
                                .resource(outputResource)
                                .lineAggregator(lineAggregator)
                                .build();
}
```


前面的大部分示例应该看起来很熟悉 . 但是，format属性的值是new，并显示在以下元素中：


```xml
<property name="format" value="%-9s%-2.0f" />
```



```java
...
FormatterLineAggregator<CustomerCredit> lineAggregator = new FormatterLineAggregator<>();
lineAggregator.setFormat("%-9s%-2.0f");
...
```


底层实现是使用作为Java 5的一部分添加的相同 `Formatter` 构建的.Java  `Formatter` 基于C编程语言的 `printf` 功能 . 有关如何配置格式化程序的大多数详细信息都可以在[Formatter](https://docs.oracle.com/javase/8/docs/api/java/util/Formatter.html)的Javadoc中找到 . 

也可以使用 `FlatFileItemWriterBuilder.FormattedBuilder` 自动创建 `BeanWrapperFieldExtractor` 和 `FormatterLineAggregator` ，如下例所示：

Java配置


```java
@Bean
public FlatFileItemWriter<CustomerCredit> itemWriter(Resource outputResource) throws Exception {
        return new FlatFileItemWriterBuilder<CustomerCredit>()
                                .name("customerCreditWriter")
                                .resource(outputResource)
                                .formatted()
                                .format("%-9s%-2.0f")
                                .names(new String[] {"name", "credit"})
                                .build();
}
```



##### Handling文件创建

 `FlatFileItemReader` 与文件资源的关系非常简单 . 初始化阅读器时，它会打开文件（如果存在），如果不存在则抛出异常 . 文件写作并不那么简单 . 乍一看，对于 `FlatFileItemWriter` ，似乎应该存在类似的简单 Contract ：如果文件已经存在，则抛出异常，如果不存在，则创建它并开始编写 . 但是，可能会重新启动 `Job` 会导致问题 . 在正常重启方案中， Contract 是相反的：如果文件存在，则从最后一个已知的正确位置开始写入，如果不存在，则抛出异常 . 但是，如果此作业的文件名始终相同，会发生什么？在这种情况下，除非重新启动，否则您希望删除该文件（如果存在） . 由于这种可能性， `FlatFileItemWriter` 包含属性 `shouldDeleteIfExists`  . 将此属性设置为true会导致在打开writer时删除具有相同名称的现有文件 . 


### 1.7. XML项目读者和作家

Spring Batch为读取XML记录并将它们映射到Java对象以及将Java对象编写为XML记录提供了事务性基础结构 . 


> 
流XML的约束

StAX API用于I / O，因为其他标准XML解析API不适合批处理要求（DOM一次将整个输入加载到内存中，SAX通过允许用户仅提供回调来控制解析过程） . 

我们需要考虑Spring Batch中XML输入和输出的工作原理 . 首先，有一些概念因文件读写而异，但在Spring Batch XML处理中很常见 . 使用XML处理，而不是需要标记化的记录行（ `FieldSet` 实例），假设XML资源是与各个记录对应的“片段”的集合，如下图所示：


![XML Input](https://www.docs4dev.com/images/51c979ee-823d-4f9e-99fb-57e86b0e4cd8.png)


图1. XML输入

'trade'标签被定义为上述场景中的'根元素' .  “<trade>”和“</ trade>”之间的所有内容都被视为一个“片段” .  Spring Batch使用对象/ XML映射（OXM）将片段绑定到对象 . 但是，Spring Batch与任何特定的XML绑定技术无关 . 典型用途是委托给[Spring OXM](https://docs.spring.io/spring/docs/current/spring-framework-reference/data-access.html#oxm)，它为最流行的OXM技术提供统一的抽象 . 对Spring OXM的依赖是可选的如果需要，您可以选择实现Spring Batch特定接口 . 与OXM支持的技术的关系如下图所示：


![OXM Binding](https://www.docs4dev.com/images/8652f094-b5dd-4dbf-bd00-69b1e963580d.png)


图2. OXM绑定

通过介绍OXM以及如何使用XML片段来表示记录，我们现在可以更仔细地检查读者和作者 . 


#### 1.7.1. StaxEventItemReader

 `StaxEventItemReader` 配置提供了从XML输入流处理记录的典型设置 . 首先，考虑 `StaxEventItemReader` 可以处理的以下XML记录集：


```xml
<?xml version="1.0" encoding="UTF-8"?>
<records>
    <trade xmlns="http://springframework.org/batch/sample/io/oxm/domain">
        <isin>XYZ0001</isin>
        <quantity>5</quantity>
        <price>11.39</price>
        <customer>Customer1</customer>
    </trade>
    <trade xmlns="http://springframework.org/batch/sample/io/oxm/domain">
        <isin>XYZ0002</isin>
        <quantity>2</quantity>
        <price>72.99</price>
        <customer>Customer2c</customer>
    </trade>
    <trade xmlns="http://springframework.org/batch/sample/io/oxm/domain">
        <isin>XYZ0003</isin>
        <quantity>9</quantity>
        <price>99.99</price>
        <customer>Customer3</customer>
    </trade>
</records>
```


为了能够处理XML记录，需要以下内容：


- 
根元素名称：构成要映射的对象的片段的根元素的名称 . 示例配置使用trade的值来证明这一点 . 


- 
资源：表示要读取的文件的Spring资源 . 


- 
 `Unmarshaller` ：Spring OXM提供的用于将XML片段映射到对象的解组工具 . 

以下示例显示如何定义 `StaxEventItemReader` ，该_14284使用名为 `trade` 的根元素， `org/springframework/batch/item/xml/domain/trades.xml` 的资源和名为 `tradeMarshaller` 的unmarshaller . 

XML配置


```xml
<bean id="itemReader" class="org.springframework.batch.item.xml.StaxEventItemReader">
    <property name="fragmentRootElementName" value="trade" />
    <property name="resource" value="org/springframework/batch/item/xml/domain/trades.xml" />
    <property name="unmarshaller" ref="tradeMarshaller" />
</bean>
```


Java配置


```java
@Bean
public StaxEventItemReader itemReader() {
        return new StaxEventItemReaderBuilder<Trade>()
                        .name("itemReader")
                        .resource(new FileSystemResource("org/springframework/batch/item/xml/domain/trades.xml"))
                        .addFragmentRootElements("trade")
                        .unmarshaller(tradeMarshaller())
                        .build();

}
```


请注意，在此示例中，我们选择使用 `XStreamMarshaller` ，它接受作为映射传入的别名，其中第一个键和值是片段的名称（即根元素）和要绑定的对象类型 . 然后，类似于 `FieldSet` ，映射到对象类型中的字段的其他元素的名称在 Map 中被描述为键/值对 . 在配置文件中，我们可以使用Spring配置实用程序来描述所需的别名，如下所示：

XML配置


```xml
<bean id="tradeMarshaller"
      class="org.springframework.oxm.xstream.XStreamMarshaller">
    <property name="aliases">
        <util:map id="aliases">
            <entry key="trade"
                   value="org.springframework.batch.sample.domain.trade.Trade" />
            <entry key="price" value="java.math.BigDecimal" />
            <entry key="isin" value="java.lang.String" />
            <entry key="customer" value="java.lang.String" />
            <entry key="quantity" value="java.lang.Long" />
        </util:map>
    </property>
</bean>
```


Java配置


```java
@Bean
public XStreamMarshaller tradeMarshaller() {
        Map<String, Class> aliases = new HashMap<>();
        aliases.put("trade", Trade.class);
        aliases.put("price", BigDecimal.class);
        aliases.put("isin", String.class);
        aliases.put("customer", String.class);
        aliases.put("quantity", Long.class);

        XStreamMarshaller marshaller = new XStreamMarshaller();

        marshaller.setAliases(aliases);

        return marshaller;
}
```


在输入时，读取器读取XML资源，直到它识别出新片段即将开始 . 默认情况下，阅读器匹配元素名称以识别新片段即将开始 . 阅读器从片段创建独立的XML文档，并将文档传递给反序列化器（通常是Spring OXM  `Unmarshaller` 的包装器），以将XML映射到Java对象 . 

总之，此过程类似于以下Java代码，它使用Spring配置提供的注入：


```java
StaxEventItemReader<Trade> xmlStaxEventItemReader = new StaxEventItemReader<>();
Resource resource = new ByteArrayResource(xmlResource.getBytes());

Map aliases = new HashMap();
aliases.put("trade","org.springframework.batch.sample.domain.trade.Trade");
aliases.put("price","java.math.BigDecimal");
aliases.put("customer","java.lang.String");
aliases.put("isin","java.lang.String");
aliases.put("quantity","java.lang.Long");
XStreamMarshaller unmarshaller = new XStreamMarshaller();
unmarshaller.setAliases(aliases);
xmlStaxEventItemReader.setUnmarshaller(unmarshaller);
xmlStaxEventItemReader.setResource(resource);
xmlStaxEventItemReader.setFragmentRootElementName("trade");
xmlStaxEventItemReader.open(new ExecutionContext());

boolean hasNext = true;

Trade trade = null;

while (hasNext) {
    trade = xmlStaxEventItemReader.read();
    if (trade == null) {
        hasNext = false;
    }
    else {
        System.out.println(trade);
    }
}
```



#### 1.7.2. StaxEventItemWriter

输出与输入对称 .   `StaxEventItemWriter` 需要一个 `Resource` ，一个marshaller和一个 `rootTagName`  .  Java对象被传递给marshaller（通常是标准的Spring OXM Marshaller），它通过使用自定义事件编写器写入 `Resource` ，该编写器过滤OXM工具为每个片段生成的 `StartDocument` 和 `EndDocument` 事件 . 以下示例使用 `StaxEventItemWriter` ：

XML配置


```xml
<bean id="itemWriter" class="org.springframework.batch.item.xml.StaxEventItemWriter">
    <property name="resource" ref="outputResource" />
    <property name="marshaller" ref="tradeMarshaller" />
    <property name="rootTagName" value="trade" />
    <property name="overwriteOutput" value="true" />
</bean>
```


Java配置


```java
@Bean
public StaxEventItemWriter itemWriter(Resource outputResource) {
        return new StaxEventItemWriterBuilder<Trade>()
                        .name("tradesWriter")
                        .marshaller(tradeMarshaller())
                        .resource(outputResource)
                        .rootTagName("trade")
                        .overwriteOutput(true)
                        .build();

}
```


上述配置设置了三个必需属性，并设置了本章前面提到的可选 `overwriteOutput=true` 属性，用于指定是否可以覆盖现有文件 . 应该注意的是，以下示例中用于编写器的编组器与本章前面的阅读示例中使用的编组器完全相同：

XML配置


```xml
<bean id="customerCreditMarshaller"
      class="org.springframework.oxm.xstream.XStreamMarshaller">
    <property name="aliases">
        <util:map id="aliases">
            <entry key="customer"
                   value="org.springframework.batch.sample.domain.trade.Trade" />
            <entry key="price" value="java.math.BigDecimal" />
            <entry key="isin" value="java.lang.String" />
            <entry key="customer" value="java.lang.String" />
            <entry key="quantity" value="java.lang.Long" />
        </util:map>
    </property>
</bean>
```


Java配置


```java
@Bean
public XStreamMarshaller customerCreditMarshaller() {
        XStreamMarshaller marshaller = new XStreamMarshaller();

        Map<String, Class> aliases = new HashMap<>();
        aliases.put("trade", Trade.class);
        aliases.put("price", BigDecimal.class);
        aliases.put("isin", String.class);
        aliases.put("customer", String.class);
        aliases.put("quantity", Long.class);

        marshaller.setAliases(aliases);

        return marshaller;
}
```


总结一下Java示例，以下代码说明了所讨论的所有要点，演示了所需属性的编程设置：


```java
FileSystemResource resource = new FileSystemResource("data/outputFile.xml")

Map aliases = new HashMap();
aliases.put("trade","org.springframework.batch.sample.domain.trade.Trade");
aliases.put("price","java.math.BigDecimal");
aliases.put("customer","java.lang.String");
aliases.put("isin","java.lang.String");
aliases.put("quantity","java.lang.Long");
Marshaller marshaller = new XStreamMarshaller();
marshaller.setAliases(aliases);

StaxEventItemWriter staxItemWriter =
        new StaxEventItemWriterBuilder<Trade>()
                                .name("tradesWriter")
                                .marshaller(marshaller)
                                .resource(resource)
                                .rootTagName("trade")
                                .overwriteOutput(true)
                                .build();

staxItemWriter.afterPropertiesSet();

ExecutionContext executionContext = new ExecutionContext();
staxItemWriter.open(executionContext);
Trade trade = new Trade();
trade.setPrice(11.39);
trade.setIsin("XYZ0001");
trade.setQuantity(5L);
trade.setCustomer("Customer1");
staxItemWriter.write(trade);
```



### 1.8. JSON项目读者和作家

Spring Batch以下列格式提供对读写JSON资源的支持：


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


假设JSON资源是与各个项对应的JSON对象数组 .  Spring Batch不依赖于任何特定的JSON库 . 


#### 1.8.1. JsonItemReader

 `JsonItemReader` 委托JSON解析并绑定到 `org.springframework.batch.item.json.JsonObjectReader` 接口的实现 . 此接口旨在通过使用流API以块的形式读取JSON对象来实现 . 目前提供两种实现方式：


- 
[Jackson](https://github.com/FasterXML/jackson)通过 `org.springframework.batch.item.json.JacksonJsonObjectReader` 


- 
[Gson](https://github.com/google/gson)通过 `org.springframework.batch.item.json.GsonJsonObjectReader` 

为了能够处理JSON记录，需要以下内容：


- 
 `Resource` ：表示要读取的JSON文件的Spring资源 . 


- 
 `JsonObjectReader` ：用于将JSON对象解析并绑定到项目的JSON对象读取器

以下示例显示如何定义适用于以前的JSON资源 `org/springframework/batch/item/json/trades.json` 的 `JsonItemReader` 和基于Jackson的 `JsonObjectReader` ：


```java
@Bean
public JsonItemReader<Trade> jsonItemReader() {
   return new JsonItemReaderBuilder<Trade>()
                 .jsonObjectReader(new JacksonJsonObjectReader<>(Trade.class))
                 .resource(new ClassPathResource("trades.json"))
                 .name("tradeJsonItemReader")
                 .build();
}
```



#### 1.8.2. JsonFileItemWriter

 `JsonFileItemWriter` 将项目编组委托给 `org.springframework.batch.item.json.JsonObjectMarshaller` 接口 . 此接口的 Contract 是获取一个对象并将其编组为JSON  `String`  . 目前提供两种实现方式：


- 
[Jackson](https://github.com/FasterXML/jackson)通过 `org.springframework.batch.item.json.JacksonJsonObjectMarshaller` 


- 
[Gson](https://github.com/google/gson)通过 `org.springframework.batch.item.json.GsonJsonObjectMarshaller` 

为了能够编写JSON记录，需要以下内容：


- 
 `Resource` ：Spring  `Resource` ，表示要写入的JSON文件


- 
 `JsonObjectMarshaller` ：JSON对象编组器，用于将对象编组为JSON格式

以下示例显示如何定义 `JsonFileItemWriter` ：


```java
@Bean
public JsonFileItemWriter<Trade> jsonFileItemWriter() {
   return new JsonFileItemWriterBuilder<Trade>()
                 .jsonObjectMarshaller(new JacksonJsonObjectMarshaller<>())
                 .resource(new ClassPathResource("trades.json"))
                 .name("tradeJsonFileItemWriter")
                 .build();
}
```



### 1.9.多文件输入

这很常见要求在单个 `Step` 内处理多个文件 . 假设文件都具有相同的格式， `MultiResourceItemReader` 支持XML和平面文件处理的这种类型的输入 . 考虑目录中的以下文件：


```java
file-1.txt  file-2.txt  ignored.txt
```


 `file-1.txt` 和 `file-2.txt` 的格式相同，并且出于商业原因，应该一起处理 .   `MultiResourceItemReader` 可用于通过使用通配符读取这两个文件，如以下示例所示：

XML配置


```xml
<bean id="multiResourceReader" class="org.spr...MultiResourceItemReader">
    <property name="resources" value="classpath:data/input/file-*.txt" />
    <property name="delegate" ref="flatFileItemReader" />
</bean>
```


Java配置


```java
@Bean
public MultiResourceItemReader multiResourceReader() {
        return new MultiResourceItemReaderBuilder<Foo>()
                                        .delegate(flatFileItemReader())
                                        .resources(resources())
                                        .build();
}
```


引用的委托是一个简单的 `FlatFileItemReader`  . 上述配置从两个文件读取输入，处理回滚和重新启动方案 . 应该注意的是，与任何 `ItemReader` 一样，添加额外输入（在本例中为文件）可能会在重新启动时导致潜在问题 . 建议批处理作业使用各自的目录，直到成功完成 . 


> 使用 `MultiResourceItemReader#setComparator(Comparator)` 对输入资源进行排序，以确保在重新启动方案中的作业运行之间保留资源排序 . 


### 1.10.数据库

与大多数企业应用程序样式一样，数据库是批处理的中央存储机制 . 但是，由于系统必须使用的数据集的大小，批处理与其他应用程序样式不同 . 如果SQL语句返回100万行，则结果集可能会将所有返回的结果保存在内存中，直到读取完所有行为止 .  Spring Batch为此问题提供了两种类型的解决方案：


- 
[Cursor-based ItemReader Implementations](#cursorBasedItemReaders)


- 
[Paging ItemReader Implementations](#pagingItemReaders)


#### 1.10.1.基于游标的ItemReader实现

使用数据库游标通常是大多数批处理开发人员的默认方法，因为它是数据库解决“流”关系数据问题的方法 .  Java  `ResultSet` 类本质上是一种用于操作游标的面向对象机制 .   `ResultSet` 将游标维护到当前数据行 . 在 `ResultSet` 上调用 `next` 会将此光标移动到下一行 .  Spring Batch基于游标的 `ItemReader` 实现在初始化时打开游标，并在每次调用 `read` 时向前移动光标一行，返回可用于处理的映射对象 . 然后调用 `close` 方法以确保释放所有资源 .  Spring核心 `JdbcTemplate` 通过使用回调模式完全映射 `ResultSet` 中的所有行并关闭然后将控制权返回给方法调用者来解决此问题 . 但是，在批处理中，必须等到步骤完成 . 下图显示了基于游标的 `ItemReader` 如何工作的通用图 . 请注意，虽然该示例使用SQL（因为SQL广为人知），但任何技术都可以实现基本方法 . 


![Cursor Example](https://www.docs4dev.com/images/0195eeb2-1fd6-4924-9412-102e15a5322e.png)


图3.光标示例

此示例说明了基本模式 . 给定一个'FOO'表，它有三列： `ID` ， `NAME` 和 `BAR` ，选择ID大于1但小于7的所有行 . 这将光标的开头（第1行）放在ID 2上 . 结果这一行应该是一个完全映射的 `Foo` 对象 . 再次调用 `read()` 将光标移动到下一行，即ID为3的 `Foo`  . 这些读取的结果在每个 `read` 之后写出，允许对象被垃圾收集（假设没有实例变量维护对它们的引用） ） . 


##### JdbcCursorItemReader

 `JdbcCursorItemReader` 是基于游标的技术的JDBC实现 . 它直接与 `ResultSet` 一起工作，并且需要一个SQL语句来对从 `DataSource` 获得的连接运行 . 以下数据库模式用作示例：


```java
CREATE TABLE CUSTOMER (
   ID BIGINT IDENTITY PRIMARY KEY,
   NAME VARCHAR(45),
   CREDIT FLOAT
);
```


许多人更喜欢为每一行使用域对象，因此以下示例使用 `RowMapper` 接口的实现来映射 `CustomerCredit` 对象：


```java
public class CustomerCreditRowMapper implements RowMapper<CustomerCredit> {

    public static final String ID_COLUMN = "id";
    public static final String NAME_COLUMN = "name";
    public static final String CREDIT_COLUMN = "credit";

    public CustomerCredit mapRow(ResultSet rs, int rowNum) throws SQLException {
        CustomerCredit customerCredit = new CustomerCredit();

        customerCredit.setId(rs.getInt(ID_COLUMN));
        customerCredit.setName(rs.getString(NAME_COLUMN));
        customerCredit.setCredit(rs.getBigDecimal(CREDIT_COLUMN));

        return customerCredit;
    }
}
```


因为 `JdbcCursorItemReader` 与 `JdbcTemplate` 共享关键接口，所以查看如何使用 `JdbcTemplate` 读取此数据的示例很有用，以便将其与 `ItemReader` 进行对比 . 出于此示例的目的，假设 `CUSTOMER` 数据库中有1,000行 . 第一个示例使用 `JdbcTemplate` ：


```java
//For simplicity sake, assume a dataSource has already been obtained
JdbcTemplate jdbcTemplate = new JdbcTemplate(dataSource);
List customerCredits = jdbcTemplate.query("SELECT ID, NAME, CREDIT from CUSTOMER",
                                          new CustomerCreditRowMapper());
```


运行上面的代码片段后， `customerCredits` 列表包含1,000个 `CustomerCredit` 对象 . 在查询方法中，从 `DataSource` 获取连接，对其运行提供的SQL，并为 `ResultSet` 中的每一行调用 `mapRow` 方法 . 将此与 `JdbcCursorItemReader` 的方法进行对比，如以下示例所示：


```java
JdbcCursorItemReader itemReader = new JdbcCursorItemReader();
itemReader.setDataSource(dataSource);
itemReader.setSql("SELECT ID, NAME, CREDIT from CUSTOMER");
itemReader.setRowMapper(new CustomerCreditRowMapper());
int counter = 0;
ExecutionContext executionContext = new ExecutionContext();
itemReader.open(executionContext);
Object customerCredit = new Object();
while(customerCredit != null){
    customerCredit = itemReader.read();
    counter++;
}
itemReader.close();
```


运行前面的代码片段后，计数器等于1,000 . 如果上面的代码已将返回的 `customerCredit` 放入列表中，则结果与 `JdbcTemplate` 示例完全相同 . 然而， `ItemReader` 的最大优点是它允许项目“流式传输” .   `read` 方法可以调用一次，项目可以由 `ItemWriter` 写出，然后可以使用 `read` 获取下一个项目 . 这允许项目读取和写入以“块”完成并定期提交，这是高性能批处理的本质 . 此外，它非常容易配置用于注入 spring 批 `Step` ，如以下示例所示：

XML配置


```xml
<bean id="itemReader" class="org.spr...JdbcCursorItemReader">
    <property name="dataSource" ref="dataSource"/>
    <property name="sql" value="select ID, NAME, CREDIT from CUSTOMER"/>
    <property name="rowMapper">
        <bean class="org.springframework.batch.sample.domain.CustomerCreditRowMapper"/>
    </property>
</bean>
```


Java配置


```java
@Bean
public JdbcCursorItemReader<CustomerCredit> itemReader() {
        return new JdbcCursorItemReaderBuilder<CustomerCredit>()
                        .dataSource(this.dataSource)
                        .name("creditReader")
                        .sql("select ID, NAME, CREDIT from CUSTOMER")
                        .rowMapper(new CustomerCreditRowMapper())
                        .build();

}
```



###### 其他属性

因为在Java中打开游标有很多不同的选项，所以 `JdbcCursorItemReader` 上有许多属性可以设置，如下表所述：

||
| ignoreWarnings |确定是否记录SQLWarning或导致异常 . 默认值为 `true` （表示记录了警告） .  |
| ---- | ---- |
| fetchSize |为JDBC驱动程序提供有关 `ItemReader` 使用的 `ResultSet` 对象需要更多行时应从数据库中提取的行数的提示 . 默认情况下，不提供任何提示 .  |
| maxRows |设置基础 `ResultSet` 在任何时候都可以容纳的最大行数限制 .  |
| queryTimeout |设置驱动程序等待 `Statement` 对象运行的秒数 . 如果超出限制，则抛出 `DataAccessException`  .  （有关详细信息，请参阅驱动程序供应商文档|
| verifyCursorPosition |因为 `ItemReader` 持有的 `ResultSet` 被传递给 `RowMapper` ，用户可以自己调用 `ResultSet.next()` ，这可能会导致读者内部计数出现问题 . 将此值设置为 `true` 会导致在 `RowMapper` 调用之后光标位置与之前不同时抛出异常 .  |
| saveState |指示读者的状态是否应保存在 `ItemStream#update(ExecutionContext)` 提供的 `ExecutionContext` 中 . 默认值为 `true`  . |
| driverSupportsAbsolute |指示JDBC驱动程序是否支持在 `ResultSet` 上设置绝对行 . 对于支持 `ResultSet.absolute()` 的JDBC驱动程序，建议将其设置为 `true` ，因为它可以提高性能，尤其是在处理大型数据集时步骤失败时 . 默认为 `false`  .  |
| setUseSharedExtendedConnection |指示是否应该由所有其他处理使用用于游标的连接，从而共享同一事务 . 如果将其设置为 `false` ，则使用自己的连接打开游标，并且不参与为其余步骤处理启动的任何事务 . 如果将此标志设置为 `true` ，则必须将DataSource包装在 `ExtendedConnectionDataSourceProxy` 中，以防止在每次提交后关闭和释放连接 . 将此选项设置为 `true` 时，将使用“READ_ONLY”和“HOLD_CURSORS_OVER_COMMIT”选项创建用于打开游标的语句 . 这允许在事务开始时保持游标打开，并在步骤处理中执行提交 . 要使用此功能，您需要一个支持此功能的数据库和一个支持JDBC 3.0或更高版本的JDBC驱动程序 . 默认为 `false`  .  |


##### HibernateCursorItemReader

正如普通的Spring用户做出关于是否使用ORM解决方案的重要决策，这些解决方案会影响他们是否使用 `JdbcTemplate` 或 `HibernateTemplate` ，Spring Batch用户具有相同的选项 .   `HibernateCursorItemReader` 是游标技术的Hibernate实现 .  Hibernate在批处理中的使用一直存在争议 . 这主要是因为Hibernate最初是为支持在线应用程序样式而开发的 . 但是，这并不意味着它不能用于批处理 . 解决此问题的最简单方法是使用 `StatelessSession` 而不是标准会话 . 这将删除Hibernate使用的所有缓存和脏检查，这可能会导致批处理方案中出现问题 . 有关无状态和普通hibernate会话之间差异的更多信息，请参阅特定hibernate发行版的文档 .   `HibernateCursorItemReader` 允许您声明一个HQL语句并传入 `SessionFactory` ，它将以与 `JdbcCursorItemReader` 相同的基本方式传回每个调用的一个项目 . 以下示例配置使用与JDBC阅读器相同的“客户信用”示例：


```java
HibernateCursorItemReader itemReader = new HibernateCursorItemReader();
itemReader.setQueryString("from CustomerCredit");
//For simplicity sake, assume sessionFactory already obtained.
itemReader.setSessionFactory(sessionFactory);
itemReader.setUseStatelessSession(true);
int counter = 0;
ExecutionContext executionContext = new ExecutionContext();
itemReader.open(executionContext);
Object customerCredit = new Object();
while(customerCredit != null){
    customerCredit = itemReader.read();
    counter++;
}
itemReader.close();
```


假设已为 `Customer` 表正确创建了hibernate映射文件，则此配置 `ItemReader` 以与 `JdbcCursorItemReader` 描述的完全相同的方式返回 `CustomerCredit` 对象 .  'useStatelessSession'属性默认为true，但已添加到此处以引起注意打开或关闭它的能力 . 还值得注意的是，可以通过 `setFetchSize` 属性设置底层游标的获取大小 . 与 `JdbcCursorItemReader` 一样，配置很简单，如以下示例所示：

XML配置


```xml
<bean id="itemReader"
      class="org.springframework.batch.item.database.HibernateCursorItemReader">
    <property name="sessionFactory" ref="sessionFactory" />
    <property name="queryString" value="from CustomerCredit" />
</bean>
```


Java配置


```java
@Bean
public HibernateCursorItemReader itemReader(SessionFactory sessionFactory) {
        return new HibernateCursorItemReaderBuilder<CustomerCredit>()
                        .name("creditReader")
                        .sessionFactory(sessionFactory)
                        .queryString("from CustomerCredit")
                        .build();
}
```



##### StoredProcedureItemReader

有时需要使用存储过程获取游标数据 .   `StoredProcedureItemReader` 的工作方式与 `JdbcCursorItemReader` 类似，不同之处在于，它不运行查询来获取游标，而是运行返回游标的存储过程 . 存储过程可以以三种不同的方式返回游标：


- 
作为返回的 `ResultSet` （由SQL Server，Sybase，DB2，Derby和MySQL使用） . 


- 
作为out参数返回的ref-cursor（由Oracle和PostgreSQL使用） . 


- 
作为存储函数调用的返回值 . 

以下示例配置使用相同的“客户信用”例子如前面的例子：

XML配置


```xml
<bean id="reader" class="o.s.batch.item.database.StoredProcedureItemReader">
    <property name="dataSource" ref="dataSource"/>
    <property name="procedureName" value="sp_customer_credit"/>
    <property name="rowMapper">
        <bean class="org.springframework.batch.sample.domain.CustomerCreditRowMapper"/>
    </property>
</bean>
```


Java配置


```java
@Bean
public StoredProcedureItemReader reader(DataSource dataSource) {
        StoredProcedureItemReader reader = new StoredProcedureItemReader();

        reader.setDataSource(dataSource);
        reader.setProcedureName("sp_customer_credit");
        reader.setRowMapper(new CustomerCreditRowMapper());

        return reader;
}
```


前面的示例依赖于存储过程来提供 `ResultSet` 作为返回结果（前面的选项1） . 

如果存储过程返回 `ref-cursor` （选项2），那么我们需要提供out参数的位置，即返回 `ref-cursor`  . 以下示例显示如何使用第一个参数作为引用游标：

XML配置


```xml
<bean id="reader" class="o.s.batch.item.database.StoredProcedureItemReader">
    <property name="dataSource" ref="dataSource"/>
    <property name="procedureName" value="sp_customer_credit"/>
    <property name="refCursorPosition" value="1"/>
    <property name="rowMapper">
        <bean class="org.springframework.batch.sample.domain.CustomerCreditRowMapper"/>
    </property>
</bean>
```


Java配置


```java
@Bean
public StoredProcedureItemReader reader(DataSource dataSource) {
        StoredProcedureItemReader reader = new StoredProcedureItemReader();

        reader.setDataSource(dataSource);
        reader.setProcedureName("sp_customer_credit");
        reader.setRowMapper(new CustomerCreditRowMapper());
        reader.setRefCursorPosition(1);

        return reader;
}
```


如果光标是从存储的函数返回的（选项3），我们需要将属性“function”设置为 `true`  . 它默认为 `false`  . 以下示例显示了它的外观：

XML配置


```xml
<bean id="reader" class="o.s.batch.item.database.StoredProcedureItemReader">
    <property name="dataSource" ref="dataSource"/>
    <property name="procedureName" value="sp_customer_credit"/>
    <property name="function" value="true"/>
    <property name="rowMapper">
        <bean class="org.springframework.batch.sample.domain.CustomerCreditRowMapper"/>
    </property>
</bean>
```


Java配置


```java
@Bean
public StoredProcedureItemReader reader(DataSource dataSource) {
        StoredProcedureItemReader reader = new StoredProcedureItemReader();

        reader.setDataSource(dataSource);
        reader.setProcedureName("sp_customer_credit");
        reader.setRowMapper(new CustomerCreditRowMapper());
        reader.setFunction(true);

        return reader;
}
```


在所有这些情况下，我们需要定义 `RowMapper` 以及 `DataSource` 和实际的过程名称 . 

如果存储过程或函数接受参数，则必须通过 `parameters` 属性声明和设置它们 . 以下示例（适用于Oracle）声明了三个参数 . 第一个是返回ref-cursor的out参数，第二个和第三个是带有 `INTEGER` 类型值的参数 . 

XML配置


```xml
<bean id="reader" class="o.s.batch.item.database.StoredProcedureItemReader">
    <property name="dataSource" ref="dataSource"/>
    <property name="procedureName" value="spring.cursor_func"/>
    <property name="parameters">
        <list>
            <bean class="org.springframework.jdbc.core.SqlOutParameter">
                <constructor-arg index="0" value="newid"/>
                <constructor-arg index="1">
                    <util:constant static-field="oracle.jdbc.OracleTypes.CURSOR"/>
                </constructor-arg>
            </bean>
            <bean class="org.springframework.jdbc.core.SqlParameter">
                <constructor-arg index="0" value="amount"/>
                <constructor-arg index="1">
                    <util:constant static-field="java.sql.Types.INTEGER"/>
                </constructor-arg>
            </bean>
            <bean class="org.springframework.jdbc.core.SqlParameter">
                <constructor-arg index="0" value="custid"/>
                <constructor-arg index="1">
                    <util:constant static-field="java.sql.Types.INTEGER"/>
                </constructor-arg>
            </bean>
        </list>
    </property>
    <property name="refCursorPosition" value="1"/>
    <property name="rowMapper" ref="rowMapper"/>
    <property name="preparedStatementSetter" ref="parameterSetter"/>
</bean>
```


Java配置


```java
@Bean
public StoredProcedureItemReader reader(DataSource dataSource) {
        List<SqlParameter> parameters = new ArrayList<>();
        parameters.add(new SqlOutParameter("newId", OracleTypes.CURSOR));
        parameters.add(new SqlParameter("amount", Types.INTEGER);
        parameters.add(new SqlParameter("custId", Types.INTEGER);

        StoredProcedureItemReader reader = new StoredProcedureItemReader();

        reader.setDataSource(dataSource);
        reader.setProcedureName("spring.cursor_func");
        reader.setParameters(parameters);
        reader.setRefCursorPosition(1);
        reader.setRowMapper(rowMapper());
        reader.setPreparedStatementSetter(parameterSetter());

        return reader;
}
```


除了参数声明之外，我们还需要指定一个 `PreparedStatementSetter` 实现来设置调用的参数值 . 这与上面的 `JdbcCursorItemReader` 相同 .  [Additional Properties](#JdbcCursorItemReaderProperties)中列出的所有其他属性也适用于 `StoredProcedureItemReader` . 


#### 1.10.2.寻呼ItemReader实现

使用数据库游标的替代方法是运行多个查询，其中每个查询都获取一部分结果 . 我们将此部分称为页面 . 每个查询都必须指定起始行号和我们想要在页面中返回的行数 . 


##### JdbcPagingItemReader

分页 `ItemReader` 的一个实现是 `JdbcPagingItemReader`  .   `JdbcPagingItemReader` 需要 `PagingQueryProvider` 负责提供用于检索构成页面的行的SQL查询 . 由于每个数据库都有自己的提供分页支持的策略，因此我们需要为每个受支持的数据库类型使用不同的 `PagingQueryProvider`  . 还有 `SqlPagingQueryProviderFactoryBean` 自动检测正在使用的数据库并确定适当的 `PagingQueryProvider` 实现 . 这简化了配置，是推荐的最佳实践 . 

 `SqlPagingQueryProviderFactoryBean` 要求您指定 `select` 子句和 `from` 子句 . 您还可以提供可选的 `where` 子句 . 这些子句和必需的 `sortKey` 用于构建SQL语句 . 


> 在 `sortKey` 上设置唯一键约束非常重要，以确保执行之间不会丢失任何数据 . 

打开阅读器后，它会以与任何其他 `ItemReader` 相同的基本方式将每个调用的一个项目传回 `read`  . 当需要额外的行时，分页发生在幕后 . 

以下示例配置使用与先前显示的基于游标的 `ItemReaders` 类似的“客户信用”示例：

XML配置


```xml
<bean id="itemReader" class="org.spr...JdbcPagingItemReader">
    <property name="dataSource" ref="dataSource"/>
    <property name="queryProvider">
        <bean class="org.spr...SqlPagingQueryProviderFactoryBean">
            <property name="selectClause" value="select id, name, credit"/>
            <property name="fromClause" value="from customer"/>
            <property name="whereClause" value="where status=:status"/>
            <property name="sortKey" value="id"/>
        </bean>
    </property>
    <property name="parameterValues">
        <map>
            <entry key="status" value="NEW"/>
        </map>
    </property>
    <property name="pageSize" value="1000"/>
    <property name="rowMapper" ref="customerMapper"/>
</bean>
```


Java配置


```java
@Bean
public JdbcPagingItemReader itemReader(DataSource dataSource, PagingQueryProvider queryProvider) {
        Map<String, Object> parameterValues = new HashMap<>();
        parameterValues.put("status", "NEW");

        return new JdbcPagingItemReaderBuilder<CustomerCredit>()
                                           .name("creditReader")
                                           .dataSource(dataSource)
                                           .queryProvider(queryProvider)
                                           .parameterValues(parameterValues)
                                           .rowMapper(customerCreditMapper())
                                           .pageSize(1000)
                                           .build();
}

@Bean
public SqlPagingQueryProviderFactoryBean queryProvider() {
        SqlPagingQueryProviderFactoryBean provider = new SqlPagingQueryProviderFactoryBean();

        provider.setSelectClause("select id, name, credit");
        provider.setFromClause("from customer");
        provider.setWhereClause("where status=:status");
        provider.setSortKey("id");

        return provider;
}
```


这个配置 `ItemReader` 使用 `RowMapper` 返回 `CustomerCredit` 对象，必须指定它 .  'pageSize'属性确定每次查询运行从数据库读取的实体数 . 

'parameterValues'属性可用于为查询指定 `Map` 参数值 . 如果在 `where` 子句中使用命名参数，则每个条目的键应与命名参数的名称匹配 . 如果你使用传统的'？'占位符，然后每个条目的键应该是占位符的编号，从1开始 . 


##### JpaPagingItemReader

分页 `ItemReader` 的另一个实现是 `JpaPagingItemReader`  .  JPA没有类似于Hibernate  `StatelessSession` 的概念，因此我们必须使用JPA规范提供的其他功能 . 由于JPA支持分页，因此在使用JPA进行批处理时这是一个很自然的选择 . 读取每个页面后，实体将分离并清除持久性上下文，以允许在处理页面后对实体进行垃圾回收 . 

 `JpaPagingItemReader` 允许您声明JPQL语句并传入 `EntityManagerFactory`  . 然后每次调用传回一个项目，以与任何其他 `ItemReader` 相同的基本方式读取 . 当需要其他实体时，分页发生在幕后 . 以下示例配置使用与先前显示的JDBC阅读器相同的“客户信用”示例：

XML配置


```xml
<bean id="itemReader" class="org.spr...JpaPagingItemReader">
    <property name="entityManagerFactory" ref="entityManagerFactory"/>
    <property name="queryString" value="select c from CustomerCredit c"/>
    <property name="pageSize" value="1000"/>
</bean>
```


Java配置


```java
@Bean
public JpaPagingItemReader itemReader() {
        return new JpaPagingItemReaderBuilder<CustomerCredit>()
                                           .name("creditReader")
                                           .entityManagerFactory(entityManagerFactory())
                                           .queryString("select c from CustomerCredit c")
                                           .pageSize(1000)
                                           .build();
}
```


假设 `CustomerCredit` 对象具有正确的JPA注释或ORM映射文件，此配置 `ItemReader` 以与上述 `JdbcPagingItemReader` 所述完全相同的方式返回 `CustomerCredit` 对象 .  'pageSize'属性确定每次查询执行时从数据库读取的实体数 . 


#### 1.10.3.数据库ItemWriters

虽然平面文件和XML文件都具有特定的 `ItemWriter` 实例，但在数据库世界中没有确切的等价物 . 这是因为交易提供所有需要的功能 .   `ItemWriter` 实现对于文件是必需的，因为它们必须表现为交易，跟踪书面项目并在适当的时间刷新或清除 . 数据库不需要此功能，因为写入已包含在事务中 . 用户可以创建自己的DAO来实现 `ItemWriter` 接口，也可以使用自定义 `ItemWriter` 中的一个来编写通用处理问题 . 无论哪种方式，他们应该没有任何问题 . 需要注意的一件事是通过批量输出提供的性能和错误处理功能 . 这在将hibernate用作 `ItemWriter` 时最常见，但在使用JDBC批处理模式时可能会遇到相同的问题 . 批处理数据库输出没有任何固有的缺陷，假设我们小心刷新并且数据中没有错误 . 但是，写入时的任何错误都可能导致混淆，因为无法知道哪个单独的项导致异常，或者即使任何单个项负责，如下图所示：


![Error On Flush](https://www.docs4dev.com/images/a4f31578-e529-4de9-a87d-f33eb78ce24f.png)


图4.刷新时的错误

如果在写入之前缓冲了项目，则在提交之前刷新缓冲区之前不会抛出任何错误 . 例如，假设每个块写入20个项目，第15个项目抛出 `DataIntegrityViolationException`  . 就_14511而言，所有20项都被成功写入，因为在实际写入之前无法知道错误发生 . 调用 `Session#flush()` 后，清空缓冲区并触发异常 . 在这一点上， `Step` 无能为力 . 必须回滚该事务 . 通常，此异常可能导致跳过项目（取决于跳过/重试策略），然后不再写入 . 但是，在批处理方案中，无法知道哪个项目导致了问题 . 当故障发生时，整个缓冲区被写入 . 解决此问题的唯一方法是在每个项目后刷新，如下图所示：


![Error On Write](https://www.docs4dev.com/images/318506e5-9548-49a5-b07d-3bc8fdb175b9.png)


图5.写入错误

这是一个常见的用例，特别是在使用Hibernate时， `ItemWriter` 实现的简单指南是在每次调用 `write()` 时刷新 . 这样做可以可靠地跳过项目，Spring Batch在内部处理错误后调用 `ItemWriter` 的粒度 . 


### 1.11.重用现有服务

批处理系统通常与其他应用程序样式结合使用 . 最常见的是在线系统，但它也可以通过移动每种应用程序样式使用的必要批量数据来支持集成甚至胖客户端应用程序 . 出于这个原因，许多用户通常希望在其批处理作业中重用现有DAO或其他服务 .  Spring容器本身通过允许注入任何必要的类来使这相当容易 . 但是，可能存在这样的情况：现有服务需要充当 `ItemReader` 或 `ItemWriter` ，以满足另一个Spring Batch类的依赖性，或者因为它确实是步骤的主要 `ItemReader`  . 为每个需要包装的服务编写适配器类是相当简单的，但由于它是如此常见的问题，Spring Batch提供了实现： `ItemReaderAdapter` 和 `ItemWriterAdapter`  . 这两个类通过调用委托模式实现标准的Spring方法，并且设置起来相当简单 . 以下示例使用 `ItemReaderAdapter` ：

XML配置


```xml
<bean id="itemReader" class="org.springframework.batch.item.adapter.ItemReaderAdapter">
    <property name="targetObject" ref="fooService" />
    <property name="targetMethod" value="generateFoo" />
</bean>

<bean id="fooService" class="org.springframework.batch.item.sample.FooService" />
```


Java配置


```java
@Bean
public ItemReaderAdapter itemReader() {
        ItemReaderAdapter reader = new ItemReaderAdapter();

        reader.setTargetObject(fooService());
        reader.setTargetMethod("generateFoo");

        return reader;
}

@Bean
public FooService fooService() {
        return new FooService();
}
```


需要注意的一点是， `targetMethod` 的 Contract 必须与 `read` 的 Contract 相同：当用尽时，它返回 `null`  . 否则，它返回 `Object`  . 其他任何事情都会阻止框架知道何时应该结束处理，导致无限循环或不正确的故障，这取决于 `ItemWriter` 的实现 . 以下示例使用 `ItemWriterAdapter` ：

XML配置


```xml
<bean id="itemWriter" class="org.springframework.batch.item.adapter.ItemWriterAdapter">
    <property name="targetObject" ref="fooService" />
    <property name="targetMethod" value="processFoo" />
</bean>

<bean id="fooService" class="org.springframework.batch.item.sample.FooService" />
```


Java配置


```java
@Bean
public ItemWriterAdapter itemWriter() {
        ItemWriterAdapter writer = new ItemWriterAdapter();

        writer.setTargetObject(fooService());
        writer.setTargetMethod("processFoo");

        return writer;
}

@Bean
public FooService fooService() {
        return new FooService();
}
```



### 1.12.验证输入

在本章的过程中，讨论了多种解析输入的方法 . 如果不是“格式良好”，每个主要实现都会引发异常 . 如果缺少一系列数据， `FixedLengthTokenizer` 会抛出异常 . 同样，尝试访问 `RowMapper` 或 `FieldSetMapper` 中不存在的索引或与预期格式不同的索引会导致抛出异常 . 在 `read` 返回之前抛出所有这些类型的异常 . 但是，它们没有解决返回项目是否有效的问题 . 例如，如果其中一个字段是年龄，则显然不能为负数 . 它可能正确解析，因为它存在并且是一个数字，但它不会导致异常 . 由于已经存在大量的验证框架，因此Spring Batch不会尝试提供另一种验证框架 . 相反，它提供了一个名为 `Validator` 的简单接口，可以通过任意数量的框架实现，如以下接口定义所示：


```java
public interface Validator<T> {

    void validate(T value) throws ValidationException;

}
```
 Contract 是 `validate` 方法如果对象无效则抛出异常，如果有效则返回正常 .  Spring Batch提供了一个开箱即用的 `ValidatingItemProcessor` ，如下面的bean定义所示：

XML配置


```xml
<bean class="org.springframework.batch.item.validator.ValidatingItemProcessor">
    <property name="validator" ref="validator" />
</bean>

<bean id="validator" class="org.springframework.batch.item.validator.SpringValidator">
        <property name="validator">
                <bean class="org.springframework.batch.sample.domain.trade.internal.validator.TradeValidator"/>
        </property>
</bean>
```


Java配置


```java
@Bean
public ValidatingItemProcessor itemProcessor() {
        ValidatingItemProcessor processor = new ValidatingItemProcessor();

        processor.setValidator(validator());

        return processor;
}

@Bean
public SpringValidator validator() {
        SpringValidator validator = new SpringValidator();

        validator.setValidator(new TradeValidator());

        return validator;
}
```


您还可以使用 `BeanValidatingItemProcessor` 来验证使用Bean Validation API（JSR-303）注释注释的项目 . 例如，给定以下类型 `Person` ：


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



### 1.13.防止国家持久化

默认情况下，所有 `ItemReader` 和 `ItemWriter` 实现在提交之前将其当前状态存储在 `ExecutionContext` 中 . 但是，这可能并不总是理想的行为 . 例如，许多开发人员选择使用流程指示器使其数据库读取器“可重新运行” . 将一个额外的列添加到输入数据以指示它是否已被处理 . 当正在读取（或写入）特定记录时，处理后的标志从 `false` 翻转到 `true`  . 然后，SQL语句可以在 `where` 子句中包含一个额外的语句，例如 `where PROCESSED_IND = false` ，从而确保在重新启动时仅返回未处理的记录 . 在这种情况下，最好不存储任何状态，例如当前行号，因为它在重新启动时无关紧要 . 因此，所有读者和编写者都包含“saveState”属性，如以下示例所示：

XML配置


```xml
<bean id="playerSummarizationSource" class="org.spr...JdbcCursorItemReader">
    <property name="dataSource" ref="dataSource" />
    <property name="rowMapper">
        <bean class="org.springframework.batch.sample.PlayerSummaryMapper" />
    </property>
    <property name="saveState" value="false" />
    <property name="sql">
        <value>
            SELECT games.player_id, games.year_no, SUM(COMPLETES),
            SUM(ATTEMPTS), SUM(PASSING_YARDS), SUM(PASSING_TD),
            SUM(INTERCEPTIONS), SUM(RUSHES), SUM(RUSH_YARDS),
            SUM(RECEPTIONS), SUM(RECEPTIONS_YARDS), SUM(TOTAL_TD)
            from games, players where players.player_id =
            games.player_id group by games.player_id, games.year_no
        </value>
    </property>
</bean>
```


Java配置


```java
@Bean
public JdbcCursorItemReader playerSummarizationSource(DataSource dataSource) {
        return new JdbcCursorItemReaderBuilder<PlayerSummary>()
                                .dataSource(dataSource)
                                .rowMapper(new PlayerSummaryMapper())
                                .saveState(false)
                                .sql("SELECT games.player_id, games.year_no, SUM(COMPLETES),"
                                  + "SUM(ATTEMPTS), SUM(PASSING_YARDS), SUM(PASSING_TD),"
                                  + "SUM(INTERCEPTIONS), SUM(RUSHES), SUM(RUSH_YARDS),"
                                  + "SUM(RECEPTIONS), SUM(RECEPTIONS_YARDS), SUM(TOTAL_TD)"
                                  + "from games, players where players.player_id ="
                                  + "games.player_id group by games.player_id, games.year_no")
                                .build();

}
```


上面配置的 `ItemReader` 不会在 `ExecutionContext` 中为其参与的任何执行中的任何条目 . 


### 1.14.创建自定义ItemReader和ItemWriters

到目前为止，本章已经讨论了Spring Batch中读写的基本 Contract 以及这样做的一些常见实现 . 但是，这些都是相当通用的，并且有许多潜在的场景可能无法通过开箱即用的实现来涵盖 . 本节通过一个简单的示例显示如何创建自定义 `ItemReader` 和 `ItemWriter` 实现并正确实现其 Contract  .   `ItemReader` 还实现 `ItemStream` ，以说明如何使读取器或写入器可重新启动 . 


#### 1.14.1.自定义ItemReader示例

出于此示例的目的，我们创建了一个简单的 `ItemReader` 实现，该实现从提供的列表中读取 . 我们首先实现 `ItemReader` 的最基本 Contract ，即 `read` 方法，如下面的代码所示：


```java
public class CustomItemReader<T> implements ItemReader<T>{

    List<T> items;

    public CustomItemReader(List<T> items) {
        this.items = items;
    }

    public T read() throws Exception, UnexpectedInputException,
       NonTransientResourceException, ParseException {

        if (!items.isEmpty()) {
            return items.remove(0);
        }
        return null;
    }
}
```


上面的类获取项目列表并一次返回一个项目，从列表中删除每个项目 . 当列表为空时，它返回 `null` ，从而满足 `ItemReader` 的最基本要求，如以下测试代码所示：


```java
List<String> items = new ArrayList<String>();
items.add("1");
items.add("2");
items.add("3");

ItemReader itemReader = new CustomItemReader<String>(items);
assertEquals("1", itemReader.read());
assertEquals("2", itemReader.read());
assertEquals("3", itemReader.read());
assertNull(itemReader.read());
```



##### 使ItemReader重新启动

最后的挑战是使 `ItemReader` 可重启 . 目前，如果处理中断并再次开始，则 `ItemReader` 必须从头开始 . 这在许多情况下实际上是有效的，但有时候批处理作业从中断的地方重新启动有时是可取的 . 关键的判别通常是读者是有状态的还是无国籍的 . 无状态读者不需要担心可重启性，但有状态的读者必须尝试在重启时重建其最后的已知状态 . 因此，我们建议您尽可能保持自定义读取器无状态，这样您就不必担心可重启性 . 

如果确实需要存储状态，则应使用 `ItemStream` 接口：


```java
public class CustomItemReader<T> implements ItemReader<T>, ItemStream {

    List<T> items;
    int currentIndex = 0;
    private static final String CURRENT_INDEX = "current.index";

    public CustomItemReader(List<T> items) {
        this.items = items;
    }

    public T read() throws Exception, UnexpectedInputException,
        ParseException, NonTransientResourceException {

        if (currentIndex < items.size()) {
            return items.get(currentIndex++);
        }

        return null;
    }

    public void open(ExecutionContext executionContext) throws ItemStreamException {
        if(executionContext.containsKey(CURRENT_INDEX)){
            currentIndex = new Long(executionContext.getLong(CURRENT_INDEX)).intValue();
        }
        else{
            currentIndex = 0;
        }
    }

    public void update(ExecutionContext executionContext) throws ItemStreamException {
        executionContext.putLong(CURRENT_INDEX, new Long(currentIndex).longValue());
    }

    public void close() throws ItemStreamException {}
}
```


在每次调用 `ItemStream`   `update` 方法时， `ItemReader` 的当前索引存储在提供的 `ExecutionContext` 中，其键为“current.index” . 调用 `ItemStream`   `open` 方法时，将检查 `ExecutionContext` 以查看它是否包含具有该键的条目 . 如果找到密钥，则将当前索引移动到该位置 . 这是一个相当简单的例子，但仍符合一般 Contract ：


```java
ExecutionContext executionContext = new ExecutionContext();
((ItemStream)itemReader).open(executionContext);
assertEquals("1", itemReader.read());
((ItemStream)itemReader).update(executionContext);

List<String> items = new ArrayList<String>();
items.add("1");
items.add("2");
items.add("3");
itemReader = new CustomItemReader<String>(items);

((ItemStream)itemReader).open(executionContext);
assertEquals("2", itemReader.read());
```


大多数 `ItemReaders` 具有更复杂的重启逻辑 . 例如， `JdbcCursorItemReader` 存储光标中最后处理的行的行ID . 

值得注意的是， `ExecutionContext` 中使用的密钥不应该是微不足道的 . 那是因为 `Step` 中的所有 `ItemStreams` 都使用相同的 `ExecutionContext`  . 在大多数情况下，只需在键名前加上键就足以保证唯一性 . 但是，在极少数情况下，在同一步骤中使用两个相同类型的 `ItemStream` （如果输出需要两个文件，则可能会发生这种情况），因此需要更独特的名称 . 出于这个原因，许多Spring Batch  `ItemReader` 和 `ItemWriter` 实现都有一个 `setName()` 属性，可以覆盖此键名 . 


#### 1.14.2.自定义ItemWriter示例

实现自定义 `ItemWriter` 在许多方面类似于上面的 `ItemReader` 示例，但在足够的方面有所不同以保证其自己的示例 . 但是，添加可重启性基本相同，因此本示例中未涉及 . 如使用 `ItemReader` 示例，使用 `List` 以使示例尽可能简单：


```java
public class CustomItemWriter<T> implements ItemWriter<T> {

    List<T> output = TransactionAwareProxyFactory.createTransactionalList();

    public void write(List<? extends T> items) throws Exception {
        output.addAll(items);
    }

    public List<T> getOutput() {
        return output;
    }
}
```



##### 使ItemWriter重新启动

为了使 `ItemWriter` 可重新启动，我们将遵循与 `ItemReader` 相同的过程，添加并实现 `ItemStream` 接口以同步执行上下文 . 在示例中，我们可能必须计算处理的项目数，并将其添加为页脚记录 . 如果我们需要这样做，我们可以在 `ItemWriter` 中实现 `ItemStream` ，以便在重新打开流时从计算器重构计数器 . 

在许多实际情况中，自定义 `ItemWriters` 也委托给另一个本身可以重新启动的编写器（例如，写入文件时），或者它写入事务资源，因此不需要重新启动，因为它是无状态的 . 当你有一个有状态的作家时，你应该确定实现 `ItemStream` 以及 `ItemWriter`  . 还要记住，编写器的客户端需要知道 `ItemStream` ，因此您可能需要将其注册为配置中的流 . 


### 1.15.项目读者和作者实现

在本节中，我们将向您介绍前面部分尚未讨论过的读者和作者 . 


#### 1.15.1.装饰者

在某些情况下，用户需要将特定行为附加到预先存在的 `ItemReader`  .  Spring Batch提供了一些开箱即用的装饰器，可以为 `ItemReader` 和 `ItemWriter` 实现添加额外的行为 . 

Spring Batch包含以下装饰器：


- 
[SynchronizedItemStreamReader](#synchronizedItemStreamReader)


- 
[SingleItemPeekableItemReader](#singleItemPeekableItemReader)


- 
[MultiResourceItemWriter](#multiResourceItemWriter)


- 
[ClassifierCompositeItemWriter](#classifierCompositeItemWriter)


- 
[ClassifierCompositeItemProcessor](#classifierCompositeItemProcessor)


##### SynchronizedItemStreamReader

当使用非线程安全的 `ItemReader` 时，Spring Batch提供 `SynchronizedItemStreamReader` 装饰器，可用于使 `ItemReader` 线程安全 .  Spring Batch提供 `SynchronizedItemStreamReaderBuilder` 来构造 `SynchronizedItemStreamReader` 的实例 . 


##### SingleItemPeekableItemReader

Spring Batch包含一个装饰器，它将一个peek方法添加到 `ItemReader`  . 这种偷看方法可以让用户提前查看一个项目 . 对peek的重复调用返回相同的项，这是从 `read` 方法返回的下一个项 .  Spring Batch提供 `SingleItemPeekableItemReaderBuilder` 来构造 `SingleItemPeekableItemReader` 的实例 . 


> SingleItemPeekableItemReader的peek方法不是线程安全的，因为无法在多个线程中实现窥视 . 只有一个偷看的线程会在下一次调用中获取该项 . 


##### MultiResourceItemWriter

 `MultiResourceItemWriter` 包装 `ResourceAwareItemWriterItemStream` 并在当前资源中写入的项数超过 `itemCountLimitPerResource` 时创建新的输出资源 .  Spring Batch提供 `MultiResourceItemWriterBuilder` 来构造 `MultiResourceItemWriter` 的实例 . 


##### ClassifierCompositeItemWriter

 `ClassifierCompositeItemWriter` 基于通过提供的 `Classifier` 实现的路由器模式，为每个项目调用 `ItemWriter` 实现的集合之一 . 如果所有委托都是线程安全的，那么实现是线程安全的 .  Spring Batch提供 `ClassifierCompositeItemWriterBuilder` 来构造 `ClassifierCompositeItemWriter` 的实例 . 


##### ClassifierCompositeItemProcessor

 `ClassifierCompositeItemProcessor` 是一个 `ItemProcessor` ，它根据通过提供的 `Classifier` 实现的路由器模式调用 `ItemProcessor` 实现的集合之一 .  Spring Batch提供 `ClassifierCompositeItemProcessorBuilder` 来构造 `ClassifierCompositeItemProcessor` 的实例 . 


#### 1.15.2.消息读者和作家

Spring Batch为常用的消息传递系统提供以下读者和编写器：


- 
[AmqpItemReader](#amqpItemReader)


- 
[AmqpItemWriter](#amqpItemWriter)


- 
[JmsItemReader](#jmsItemReader)


- 
[JmsItemWriter](#jmsItemWriter)


##### AmqpItemReader

 `AmqpItemReader` 是 `ItemReader` ，它使用 `AmqpTemplate` 来接收或转换来自交换的消息 .  Spring Batch提供 `AmqpItemReaderBuilder` 来构造 `AmqpItemReader` 的实例 . 


##### AmqpItemWriter

 `AmqpItemWriter` 是 `ItemWriter` ，它使用 `AmqpTemplate` 将消息发送到AMQP交换机 . 如果未在提供的 `AmqpTemplate` 中指定名称，则将消息发送到无名交换 .  Spring Batch提供 `AmqpItemWriterBuilder` 来构造 `AmqpItemWriter` 的实例 . 


##### JmsItemReader

对于使用 `JmsTemplate` 的JMS， `JmsItemReader` 是 `ItemReader`  . 模板应具有默认目标，该目标用于为 `read()` 方法提供项目 .  Spring Batch提供 `JmsItemReaderBuilder` 来构造 `JmsItemReader` 的实例 . 


##### JmsItemWriter

对于使用 `JmsTemplate` 的JMS， `JmsItemWriter` 是 `ItemWriter`  . 模板应具有默认目标，用于在 `write(List)` 中发送项目 .  Spring Batch提供 `JmsItemWriterBuilder` 来构造 `JmsItemWriter` 的实例 . 


#### 1.15.3.数据库读者

Spring Batch提供以下数据库读者：


- 
[Neo4jItemReader](#Neo4jItemReader)


- 
[MongoItemReader](#mongoItemReader)


- 
[HibernateCursorItemReader](#hibernateCursorItemReader)


- 
[HibernatePagingItemReader](#hibernatePagingItemReader)


- 
[RepositoryItemReader](#repositoryItemReader)


##### Neo4jItemReader

 `Neo4jItemReader` 是 `ItemReader` ，它使用分页技术从图形数据库Neo4j中读取对象 .  Spring Batch提供 `Neo4jItemReaderBuilder` 来构造 `Neo4jItemReader` 的实例 . 


##### MongoItemReader

 `MongoItemReader` 是一个 `ItemReader` ，它使用分页技术从MongoDB中读取文档 .  Spring Batch提供了 `MongoItemReaderBuilder` 来构造一个 `MongoItemReader` 的实例 . 


##### HibernateCursorItemReader

 `HibernateCursorItemReader` 是一个 `ItemStreamReader` ，用于读取基于Hibernate构建的数据库记录 . 它执行HQL查询，然后在初始化时，在调用 `read()` 方法时迭代结果集，连续返回与当前行对应的对象 .  Spring Batch提供 `HibernateCursorItemReaderBuilder` 来构造 `HibernateCursorItemReader` 的实例 . 


##### HibernatePagingItemReader

 `HibernatePagingItemReader` 是一个 `ItemReader` ，用于读取构建在Hibernate之上的数据库记录，并且一次只能读取固定数量的项目 .  Spring Batch提供 `HibernatePagingItemReaderBuilder` 来构造 `HibernatePagingItemReader` 的实例 . 


##### RepositoryItemReader

 `RepositoryItemReader` 是 `ItemReader` ，它使用 `PagingAndSortingRepository` 读取记录 .  Spring Batch提供 `RepositoryItemReaderBuilder` 来构造 `RepositoryItemReader` 的实例 . 


#### 1.15.4.数据库作者

Spring Batch提供以下数据库编写器：


- 
[Neo4jItemWriter](#neo4jItemWriter)


- 
[MongoItemWriter](#mongoItemWriter)


- 
[RepositoryItemWriter](#repositoryItemWriter)


- 
[HibernateItemWriter](#hibernateItemWriter)


- 
[JdbcBatchItemWriter](#jdbcBatchItemWriter)


- 
[JpaItemWriter](#jpaItemWriter)


- 
[GemfireItemWriter](#gemfireItemWriter)


##### Neo4jItemWriter

 `Neo4jItemWriter` 是一个写入Neo4j数据库的 `ItemWriter` 实现 .  Spring Batch提供 `Neo4jItemWriterBuilder` 来构造 `Neo4jItemWriter` 的实例 . 


##### MongoItemWriter

 `MongoItemWriter` 是一个 `ItemWriter` 实现，它使用Spring Data的 `MongoOperations` 实现写入MongoDB存储 .  Spring Batch提供 `MongoItemWriterBuilder` 来构造 `MongoItemWriter` 的实例 . 


##### RepositoryItemWriter

 `RepositoryItemWriter` 是来自Spring Data的 `CrudRepository` 的 `ItemWriter` 包装器 .  Spring Batch提供 `RepositoryItemWriterBuilder` 来构造 `RepositoryItemWriter` 的实例 . 


##### HibernateItemWriter

 `HibernateItemWriter` 是 `ItemWriter` ，它使用Hibernate会话来保存或更新不属于当前Hibernate会话的实体 .  Spring Batch提供 `HibernateItemWriterBuilder` 来构造 `HibernateItemWriter` 的实例 . 


##### JdbcBatchItemWriter

 `JdbcBatchItemWriter` 是 `ItemWriter` ，它使用 `NamedParameterJdbcTemplate` 中的批处理功能为所有提供的项目执行一批语句 .  Spring Batch提供 `JdbcBatchItemWriterBuilder` 来构造 `JdbcBatchItemWriter` 的实例 . 


##### JpaItemWriter

 `JpaItemWriter` 是一个 `ItemWriter` ，它使用JPA  `EntityManagerFactory` 来合并任何不属于持久化上下文的实体 .  Spring Batch提供 `JpaItemWriterBuilder` 来构造 `JpaItemWriter` 的实例 . 


##### GemfireItemWriter

 `GemfireItemWriter` 是 `ItemWriter` ，它使用 `GemfireTemplate` 将GemFire中的项目存储为键/值对 .  Spring Batch提供 `GemfireItemWriterBuilder` 来构造 `GemfireItemWriter` 的实例 . 


#### 1.15.5.专业读者

Spring Batch提供以下专业读者：


- 
[LdifReader](#ldifReader)


- 
[MappingLdifReader](#mappingLdifReader)


##### LdifReader

 `LdifReader` 从 `Resource` 读取LDIF（LDAP数据交换格式）记录，解析它们，并为每个执行的 `read` 返回 `LdapAttribute` 对象 .  Spring Batch提供 `LdifReaderBuilder` 来构造 `LdifReader` 的实例 . 


##### MappingLdifReader

 `MappingLdifReader` 从 `Resource` 读取LDIF（LDAP数据交换格式）记录，解析它们然后将每个LDIF记录映射到POJO（普通旧Java对象） . 每次读取都返回一个POJO .  Spring Batch提供 `MappingLdifReaderBuilder` 来构造 `MappingLdifReader` 的实例 . 


#### 1.15.6.专业作家

Spring Batch提供以下专业编写器：


- 
[SimpleMailMessageItemWriter](#simpleMailMessageItemWriter)


##### SimpleMailMessageItemWriter

 `SimpleMailMessageItemWriter` 是一个可以发送邮件的 `ItemWriter`  . 它将实际发送的消息委托给 `MailSender` 的实例 .  Spring Batch提供 `SimpleMailMessageItemWriterBuilder` 来构造 `SimpleMailMessageItemWriter` 的实例 . 


#### 1.15.7.专业处理器

Spring Batch提供以下专用处理器：


- 
[ScriptItemProcessor](#scriptItemProcessor)


##### ScriptItemProcessor

 `ScriptItemProcessor` 是 `ItemProcessor` ，它传递当前项以处理提供的脚本，并且处理器返回脚本的结果 .  Spring Batch提供 `ScriptItemProcessorBuilder` 来构造 `ScriptItemProcessor` 的实例 .
## 1.域名语言批处理

XML Java

对于任何有经验的批处理架构师，Spring Batch中使用的批处理的整体概念应该是熟悉和舒适的 . 有"Jobs"和"Steps"以及开发人员提供的处理单元，名为 `ItemReader` 和 `ItemWriter`  . 但是，由于Spring模式，操作，模板，回调和习语，有以下几种可能：


- 
坚持明确分离关注点的重大改进 . 


- 
清楚地描述了作为接口提供的架构层和服务 . 


- 
简单和默认的实现，允许快速采用和易于使用的开箱即用 . 


- 
显着增强的可扩展性 . 

下图是批量参考体系结构的简化版本，已使用了数十年 . 它概述了构成批处理领域语言的组件 . 这个体系结构框架是一个蓝图，已经在过去几代平台（COBOL / Mainframe，C / Unix，现在Java /任何地方）上实现了数十年的实现 .  JCL和COBOL开发人员可能对C，C＃和Java开发人员的概念感到满意 .  Spring Batch提供了健壮，可维护系统中常见的层，组件和技术服务的物理实现，这些系统用于解决简单到复杂批处理应用程序的创建问题，其基础结构和扩展可满足非常复杂的处理需求 . 


![Figure 2.1: Batch Stereotypes](https://www.docs4dev.com/images/9914b9e6-1cde-42ad-80da-14207b2725ba.png)


图1.批量刻板印象

上图突出显示了构成Spring Batch域语言的关键概念 .  Job有一个到多个步骤，每个步骤只有一个 `ItemReader` ，一个 `ItemProcessor` 和一个 `ItemWriter`  . 需要启动作业（使用 `JobLauncher` ），并且需要存储有关当前正在运行的进程的元数据（在 `JobRepository` 中） . 


### 1.1.工作

本节描述与批处理作业的概念相关的构造型 .   `Job` 是封装整个批处理过程的实体 . 与其他Spring项目一样， `Job` 与XML配置文件或基于Java的配置连接在一起 . 该配置可以称为"job configuration" . 但是， `Job` 只是整个层次结构的顶部，如下图所示：


![Job Hierarchy](https://www.docs4dev.com/images/ede70dcc-27a6-4691-896c-7e0d1570ed13.png)


图2.作业层次结构

在Spring Batch中， `Job` 只是 `Step` 实例的容器 . 它结合了逻辑上属于流的多个步骤，并允许为所有步骤配置全局属性，例如可重启性 . 作业配置包含：


- 
工作的简单名称 . 


- 
 `Step` 实例的定义和排序 . 


- 
作业是否可重新启动 . 

Spring Batch以 `SimpleJob` 类的形式提供了Job接口的默认简单实现，它在 `Job` 之上创建了一些标准功能 . 使用基于java的配置时，可以使用一组构建器来实例化 `Job` ，如以下示例所示：


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


Spring Batch以 `SimpleJob` 类的形式提供了 `Job` 接口的默认简单实现，它在 `Job` 之上创建了一些标准功能 . 但是，批处理命名空间抽象了直接实例化它的需要 . 相反，可以使用 `<job>` 标记，如以下示例所示：


```xml
<job id="footballJob">
    <step id="playerload" next="gameLoad"/>
    <step id="gameLoad" next="playerSummarization"/>
    <step id="playerSummarization"/>
</job>
```



#### 1.1.1. JobInstance

 `JobInstance` 指的是逻辑作业运行的概念 . 考虑应该在一天结束时运行一次的批处理作业，例如上图中的“EndOfDay” `Job`  . 有一个'EndOfDay'作业，但必须单独跟踪 `Job` 的每个单独运行 . 在这项工作的情况下，每天有一个逻辑 `JobInstance`  . 例如，1月1日运行，1月2日运行，依此类推 . 如果1月1日运行第一次失败并在第二天再次运行，那么它仍然是1月1日运行 .  （通常，这与它正在处理的数据相对应，这意味着1月1日运行处理1月1日的数据） . 因此，每个 `JobInstance` 可以有多个执行（ `JobExecution` 将在本章后面详细讨论），并且只有一个 `JobInstance` 对应于特定的 `Job` 并且识别 `JobParameters` 可以在给定时间运行 . 

 `JobInstance` 的定义与要加载的数据完全无关 . 完全由 `ItemReader` 实现决定数据的加载方式 . 例如，在EndOfDay方案中，数据上可能有一列指示数据所属的“生效日期”或“计划日期” . 因此，1月1日运行将仅加载来自第1个的数据，而1月2日运行将仅使用来自第2个的数据 . 由于这一决定很可能是一项商业决策，因此需要由 `ItemReader` 来决定 . 但是，使用相同的 `JobInstance` 确定是否“状态”（即 `ExecutionContext` ，这是讨论的本章后面的内容）来自之前的执行 . 使用新的 `JobInstance` 意味着“从头开始”，并且使用现有实例通常意味着“从您离开的地方开始” . 


#### 1.1.2. JobParameters

在讨论了 `JobInstance` 以及它与Job之间的区别之后，自然要问的问题是：“一个人如何与另一个人区别开来？”答案是： `JobParameters`  .   `JobParameters` 对象包含一组用于启动批处理作业的参数 . 它们可以在运行期间用于识别甚至作为参考数据，如下图所示：


![Job Parameters](https://www.docs4dev.com/images/2bdd00ab-6627-4ee6-b31b-01912a32fb9e.png)


图3.作业参数

在前面的示例中，有两个实例，一个用于1月1日，另一个用于1月2日，实际上只有一个 `Job` ，但它有两个 `JobParameter` 对象：一个以作业参数01-01-2017启动的对象另一个以参数01-02-2017开始 . 因此， Contract 可以定义为： `JobInstance`  =  `Job` 识别 `JobParameters`  . 这允许开发人员有效地控制 `JobInstance` 的定义方式，因为它们控制传入的参数 . 


> 并非所有作业参数都需要有助于识别 `JobInstance`  . 默认情况下，他们会这样做 . 但是，该框架还允许提交 `Job` ，其参数不会影响 `JobInstance` 的身份 . 


#### 1.1.3. JobExecution

 `JobExecution` 指的是单次尝试运行作业的技术概念 . 执行可能以失败或成功结束，但除非执行成功完成，否则对应于给定执行的 `JobInstance` 不被视为完成 . 使用前面描述的EndOfDay  `Job` 作为示例，考虑第一次运行时失败的01-01-2017的 `JobInstance`  . 如果使用与第一次运行（01-01-2017）相同的识别作业参数再次运行，则会创建新的 `JobExecution`  . 但是，仍然只有一个 `JobInstance`  . 

 `Job` 定义了作业是什么以及如何执行， `JobInstance` 是一个纯粹的组织对象，用于将执行组合在一起，主要是为了启用正确的重启语义 . 但是， `JobExecution` 是运行期间实际发生的主要存储机制，包含许多必须控制和保留的属性，如下表所示：

||
|属性|定义|
| ---- | ---- |
| Status |一个 `BatchStatus` 对象，指示执行的状态 . 在运行时，它是 `BatchStatus#STARTED`  . 如果失败，则为 `BatchStatus#FAILED`  . 如果成功完成，则为 `BatchStatus#COMPLETED` |
| startTime |  `java.util.Date` 表示执行启动时的当前系统时间 . 如果作业尚未启动，则此字段为空 .  |
| endTime |  `java.util.Date` 表示执行完成时的当前系统时间，无论它是否成功 . 如果作业尚未完成，则该字段为空 .  |
| exitStatus |  `ExitStatus` ，表示运行的结果 . 这是最重要的，因为它包含一个返回给调用者的退出代码 . 有关详细信息，请参阅第5章 . 如果作业尚未完成，则该字段为空 .  |
| createTime |  `java.util.Date` 表示首次保留 `JobExecution` 时的当前系统时间 . 作业可能尚未启动（因此没有开始时间），但它始终具有createTime，这是框架管理作业级别 `ExecutionContexts` 所需的 .  |
| lastUpdated |  `java.util.Date` 表示上次保留 `JobExecution` 的时间 . 如果作业尚未启动，则此字段为空 .  |
| executionContext |“属性包”，包含需要在执行之间保留的任何用户数据 .  |
| failureExceptions |执行 `Job` 期间遇到的异常列表 . 如果在 `Job` 失败期间遇到多个异常，这些可能很有用 .  |

这些属性很重要，因为它们是持久的，可用于完全确定执行的状态 . 例如，如果01-01的EndOfDay作业在晚上9:00执行而在9:30失败，则在批处理元数据表中进行以下输入：

||
| JOB_INST_ID | JOB_NAME |
| ---- | ---- |
| 1 | EndOfDayJob |

||
| JOB_EXECUTION_ID | TYPE_CD | KEY_NAME | DATE_VAL | IDENTIFYING |
| ---- | ---- | ---- | ---- | ---- |
| 1 | DATE | schedule.Date | 2017-01-01 | TRUE |

||
| JOB_EXEC_ID | JOB_INST_ID | START_TIME | END_TIME |状态|
| ---- | ---- | ---- | ---- | ---- |
| 1 | 1 | 2017-01-01 21:00 | 2017-01-01 21:30 | FAILED |


> 为了清晰和格式化，列名可能已缩写或删除 . 

现在作业已经失败，假设确定了问题的整个晚上，以便“批处理窗口”现在关闭 . 进一步假设窗口在晚上9点开始，该作业再次从01-01开始，从它停止并从9:30成功完成 . 因为它现在是第二天，所以01-02工作也必须运行，然后在9:31之后开始，并在10:30正常的一小时时间内完成 . 没有要求在之后启动一个 `JobInstance` 另外，除非两个作业有可能尝试访问相同的数据，导致数据库级别的锁定问题 . 完全由调度程序决定何时应运行 `Job`  . 由于它们是分开的 `JobInstances` ，因此Spring Batch不会尝试阻止它们同时运行 .  （尝试运行相同的 `JobInstance` 而另一个已经运行会导致 `JobExecutionAlreadyRunningException` 被抛出） . 现在， `JobInstance` 和 `JobParameters` 表中应该有一个额外的条目， `JobExecution` 表中有两个额外的条目，如下表所示：

||
| JOB_INST_ID | JOB_NAME |
| ---- | ---- |
| 1 | EndOfDayJob |
| 2 | EndOfDayJob |

||
| JOB_EXECUTION_ID | TYPE_CD | KEY_NAME | DATE_VAL | IDENTIFYING |
| ---- | ---- | ---- | ---- | ---- |
| 1 | DATE | schedule.Date | 2017-01-01 00:00:00 | TRUE |
| 2 | DATE | schedule.Date | 2017-01-01 00:00:00 | TRUE |
| 3 |日期| schedule.Date | 2017-01-02 00:00:00 | TRUE |

||
| JOB_EXEC_ID | JOB_INST_ID | START_TIME | END_TIME |状态|
| ---- | ---- | ---- | ---- | ---- |
| 1 | 1 | 2017-01-01 21:00 | 2017-01-01 21:30 | FAILED |
| 2 | 1 | 2017-01-02 21:00 | 2017-01-02 21:30 |已完成|
| 3 | 2 | 2017-01-02 21:31 | 2017-01-02 22:29 |已完成|


> 为了清晰和格式化，列名可能已缩写或删除 . 


### 1.2.步骤

 `Step` 是一个域对象，它封装了批处理作业的独立顺序阶段 . 因此，每个Job完全由一个或多个步骤组成 .   `Step` 包含定义和控制实际批处理所需的所有信息 . 这是一个必然模糊的描述，因为任何给定 `Step` 的内容由开发人员自行决定编写 `Job`  .   `Step` 可以像开发者所希望的那样简单或复杂 . 一个简单的 `Step` 可能会将数据从文件加载到数据库中，几乎不需要代码（取决于所使用的实现） . 更复杂的 `Step` 可能具有复杂的业务规则，这些规则作为处理的一部分应用 . 与 `Job` 一样， `Step` 具有与唯一 `JobExecution` 相关联的个体 `StepExecution` ，如下图所示：


![Figure 2.1: Job Hierarchy With Steps](https://www.docs4dev.com/images/95e0e4b6-c866-4f71-89ed-bb83ef99dbbd.png)


图4.带有步骤的作业层次结构


#### 1.2.1. StepExecution

 `StepExecution` 表示执行 `Step` 的单次尝试 . 每次运行 `Step` 时都会创建一个新的 `StepExecution` ，类似于 `JobExecution`  . 但是，如果某个步骤由于其失败之前的步骤而无法执行，则不会继续执行该步骤 . 仅当 `StepExecution` 实际启动时才会创建 `StepExecution`  . 

 `Step` 执行由 `StepExecution` 类的对象表示 . 每个执行都包含对其相应步骤和 `JobExecution` 以及与事务相关的数据的引用，例如提交和回滚计数以及开始和结束时间 . 此外，每个步骤执行都包含 `ExecutionContext` ，其中包含开发人员需要在批处理运行中保留的任何数据，例如重新启动所需的统计信息或状态信息 . 下表列出了 `StepExecution` 的属性：

||
|属性|定义|
| ---- | ---- |
| Status |一个 `BatchStatus` 对象，指示执行的状态 . 运行时，状态为 `BatchStatus.STARTED`  . 如果失败，则状态为 `BatchStatus.FAILED`  . 如果成功完成，状态为 `BatchStatus.COMPLETED`  .  |
| startTime |  `java.util.Date` 表示执行开始时的当前系统时间 . 如果该步骤尚未开始，则此字段为空 .  |
| endTime |  `java.util.Date` 表示执行完成时的当前系统时间，无论它是否成功 . 如果步骤尚未退出，则此字段为空 .  |
| exitStatus |  `ExitStatus` 表示执行结果 . 这是最重要的，因为它包含一个返回给调用者的退出代码 . 有关详细信息，请参阅第5章 . 如果作业尚未退出，则此字段为空 .  |
| executionContext |“属性包”，包含需要在执行之间保留的任何用户数据 .  |
| readCount |已成功读取的项目数 .  |
| writeCount |已成功写入的项目数 .  |
| commitCount |已为此执行提交的事务数 .  |
| rollbackCount |回滚 `Step` 控制的业务事务的次数 .  |
| readSkipCount |  `read` 失败的次数，导致跳过项目 .  |
| processSkipCount |  `process` 失败的次数，导致跳过项目 .  |
| filterCount |  `ItemProcessor` 已“过滤”的项目数 .  |
| writeSkipCount |  `write` 失败的次数，导致跳过项目 .  |


### 1.3. ExecutionContext

 `ExecutionContext` 表示由框架持久化和控制的键/值对的集合，以便允许开发人员存储限定为 `StepExecution` 对象或 `JobExecution` 对象的持久状态 . 对于熟悉Quartz的人来说，它与JobDataMap非常相似 . 最好的用法示例是方便重启 . 以平面文件输入为例，同时处理个人在这些行中，框架会定期在提交点保持 `ExecutionContext`  . 这样做允许 `ItemReader` 存储其状态，以防在运行期间发生致命错误或即使断电 . 所需要的只是将当前读取的行数放入上下文中，如下例所示，框架将完成剩下的工作：


```java
executionContext.putLong(getKey(LINES_READ_COUNT), reader.getPosition());
```


使用 `Job`  Stereotypes部分中的EndOfDay示例作为示例，假设有一个步骤“loadData”将文件加载到数据库中 . 在第一次失败运行后，元数据表将类似于以下示例：

||
| JOB_INST_ID | JOB_NAME |
| ---- | ---- |
| 1 | EndOfDayJob |

||
| JOB_INST_ID | TYPE_CD | KEY_NAME | DATE_VAL |
| ---- | ---- | ---- | ---- |
| 1 | DATE | schedule.Date | 2017-01-01 |

||
| JOB_EXEC_ID | JOB_INST_ID | START_TIME | END_TIME |状态|
| ---- | ---- | ---- | ---- | ---- |
| 1 | 1 | 2017-01-01 21:00 | 2017-01-01 21:30 | FAILED |

||
| STEP_EXEC_ID | JOB_EXEC_ID | STEP_NAME | START_TIME | END_TIME |状态|
| ---- | ---- | ---- | ---- | ---- | ---- |
| 1 | 1 | loadData | 2017-01-01 21:00 | 2017-01-01 21:30 | FAILED |

||
| STEP_EXEC_ID | SHORT_CONTEXT |
| ---- | ---- |
| 1 | {piece.count = 40321} |

在前面的例子中， `Step` 运行了30分钟并处理了40,321个“件”，这将表示此场景中文件中的行 . 此值在框架每次提交之前更新，并且可以包含与 `ExecutionContext` 中的条目对应的多个行 . 在提交之前得到通知需要各种 `StepListener` 实现之一（或 `ItemStream` ），本指南后面将对此进行更详细的讨论 . 与前面的示例一样，假设 `Job` 在第二天重新启动 . 重新启动时，将从数据库重新构建上次运行的 `ExecutionContext` 中的值 . 当 `ItemReader` 打开时，它可以检查上下文中是否有任何存储状态并从那里初始化自身，如下例所示：


```java
if (executionContext.containsKey(getKey(LINES_READ_COUNT))) {
    log.debug("Initializing for restart. Restart data is: " + executionContext);

    long lineCount = executionContext.getLong(getKey(LINES_READ_COUNT));

    LineReader reader = getReader();

    Object record = "";
    while (reader.getPosition() < lineCount && record != null) {
        record = readLine();
    }
}
```


在这种情况下，在上面的代码运行后，当前行为40,322，允许 `Step` 从它停止的位置再次启动 .   `ExecutionContext` 也可用于需要针对运行本身持久化的统计信息 . 例如，如果平面文件包含跨多行存在的处理订单，则可能需要存储已处理的订单数量（这与读取的行数有很大不同），以便可以在以下位置发送电子邮件 `Step` 的结尾与正文中处理的订单总数 . 框架处理为开发人员存储它，以便正确地将其与单个 `JobInstance` 进行范围化 . 要知道是否应该使用现有的 `ExecutionContext` 可能非常困难 . 例如，使用上面的'EndOfDay'示例，当01-01运行第二次再次启动时，框架识别出它是相同的 `JobInstance` ，并且在单个 `Step` 的基础上，将 `ExecutionContext` 拉出数据库，并且将它（作为 `StepExecution` 的一部分）交给 `Step` 本身 . 相反，对于01-02运行，框架识别出它是一个不同的实例，因此必须将空上下文传递给 `Step`  . 框架为开发人员提供了许多类型的确定，以确保在正确的时间给予它们状态 . 同样重要的是要注意在任何给定时间每 `StepExecution` 只存在一个 `ExecutionContext`  .   `ExecutionContext` 的客户端应该小心，因为这会创建一个共享密钥空间 . 因此，在放入值时应小心，以确保不会覆盖任何数据 . 但是， `Step` 在上下文中绝对没有数据存储，所以没有办法对框架产生负面影响 . 

同样重要的是要注意每 `JobExecution` 至少有一个 `ExecutionContext` ，每个 `StepExecution` 至少有一个 `ExecutionContext`  . 例如，请考虑以下代码段：


```java
ExecutionContext ecStep = stepExecution.getExecutionContext();
ExecutionContext ecJob = jobExecution.getExecutionContext();
//ecStep does not equal ecJob
```


如评论中所述， `ecStep` 不等于 `ecJob`  . 他们是两个不同的 `ExecutionContexts`  . 作用于 `Step` 的作用域保存在 `Step` 中的每个提交点，而作用域作用的作用保存在每个 `Step` 执行之间 . 


### 1.4. JobRepository

 `JobRepository` 是上述所有刻板印象的持久性机制 . 它为 `JobLauncher` ， `Job` 和 `Step` 实现提供CRUD操作 . 首次启动 `Job` 时，将从存储库中获取 `JobExecution` ，并且在执行过程中，通过将它们传递到存储库来保持 `StepExecution` 和 `JobExecution` 实现 . 

批处理命名空间支持使用 `<job-repository>` 标记配置 `JobRepository` 实例，如以下示例所示：


```xml
<job-repository id="jobRepository"/>
```


使用java配置时， `@EnableBatchProcessing`  annotation提供 `JobRepository` 作为自动配置的组件之一 . 


### 1.5. JobLauncher

 `JobLauncher` 表示用于使用给定的 `JobParameters` 集启动 `Job` 的简单接口，如以下示例所示：


```java
public interface JobLauncher {

public JobExecution run(Job job, JobParameters jobParameters)
            throws JobExecutionAlreadyRunningException, JobRestartException,
                   JobInstanceAlreadyCompleteException, JobParametersInvalidException;
}
```


预期实现从 `JobRepository` 获取有效的 `JobExecution` 并执行 `Job`  . 


### 1.6.项目阅读器

 `ItemReader` 是一个抽象，表示一次检索 `Step` 的输入 . 当 `ItemReader` 已经耗尽它可以提供的项目时，它通过返回 `null` 来指示这一点 . 有关 `ItemReader` 接口及其各种实现的更多详细信息，请参见[Readers And Writers](readersAndWriters.html#readersAndWriters) . 


### 1.7.项目作者

 `ItemWriter` 是一个抽象，表示 `Step` 的输出，一次一批或一批项目 . 通常， `ItemWriter` 不知道它接下来应该接收的输入，并且只知道在其当前调用中传递的项 . 有关 `ItemWriter` 接口及其各种实现的更多详细信息，请参见[Readers And Writers](readersAndWriters.html#readersAndWriters) . 


### 1.8.物品处理器

 `ItemProcessor` 是表示项目业务处理的抽象 . 当 `ItemReader` 读取一个项目并且 `ItemWriter` 写入它们时， `ItemProcessor` 提供了一个转换或应用其他业务处理的访问点 . 如果在处理项目时确定该项目无效，则返回 `null` 表示不应该写出该项目 . 有关 `ItemProcessor` 接口的更多详细信息，请参见[Readers And Writers](readersAndWriters.html#readersAndWriters) . 


### 1.9.批处理命名空间

以前列出的许多域概念需要在Spring  `ApplicationContext` 中进行配置 . 虽然可以在标准bean定义中使用上述接口的实现，但是为了便于配置，已经提供了命名空间，如以下示例所示：


```xml
<beans:beans xmlns="http://www.springframework.org/schema/batch"
xmlns:beans="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
xsi:schemaLocation="
   http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/batch
   http://www.springframework.org/schema/batch/spring-batch.xsd">

<job id="ioSampleJob">
    <step id="step1">
        <tasklet>
            <chunk reader="itemReader" writer="itemWriter" commit-interval="2"/>
        </tasklet>
    </step>
</job>

</beans:beans>
```


只要声明了批处理命名空间，就可以使用其任何元素 . 有关配置作业的更多信息，请参见[Configuring and Running a Job](job.html#configureJob) . 有关配置 `Step` 的更多信息，请参见[Configuring a Step](step.html#configureStep) .
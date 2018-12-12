## 1.单元测试

XML Java

与其他应用程序样式一样，对作为批处理作业的一部分编写的任何代码进行单元测试非常重要 .  Spring核心文档详细介绍了如何对Spring进行单元测试和集成测试，因此这里不再重复 . 但是，重要的是要考虑如何“端到端”测试批处理作业，这正是本章所涉及的内容 .  spring 批量测试项目包括促进这种端到端测试方法的类 . 


### 1.1.创建单元测试类

为了使单元测试运行批处理作业，框架必须加载作业的 `ApplicationContext`  . 两个注释用于触发此行为：


- 
 `@RunWith(SpringRunner.class)` ：表示该类应该使用Spring的JUnit工具


- 
 `@ContextConfiguration(…)` ：表示配置 `ApplicationContext` 的资源 . 

从v4.1开始，还可以使用 `@SpringBatchTest` 注释在测试上下文中注入Spring批处理测试实用程序，如 `JobLauncherTestUtils` 和 `JobRepositoryTestUtils`  . 

以下示例显示了正在使用的注释：

使用Java配置


```java
@SpringBatchTest
@RunWith(SpringRunner.class)
@ContextConfiguration(classes=SkipSampleConfiguration.class)
public class SkipSampleFunctionalTests { ... }
```


使用XML配置


```java
@SpringBatchTest
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = { "/simple-job-launcher-context.xml",
                                    "/jobs/skipSampleJob.xml" })
public class SkipSampleFunctionalTests { ... }
```



### 1.2.批量作业的端到端测试

“端到端”测试可以定义为从头到尾测试批处理作业的完整运行 . 这允许进行测试以设置测试条件，执行作业并验证最终结果 . 

在以下示例中，批处理作业从数据库读取并写入平面文件 . 测试方法首先使用测试数据设置数据库 . 它清除CUSTOMER表，然后插入10个新记录 . 然后，测试使用 `launchJob()` 方法启动 `Job`  .   `launchJob()` 方法由 `JobLauncherTestUtils` 类提供 .   `JobLauncherTestUtils` 类还提供 `launchJob(JobParameters)` 方法，该方法允许测试提供特定参数 .   `launchJob()` 方法返回 `JobExecution` 对象，这对于断言有关 `Job`  run的特定信息很有用 . 在以下情况中，测试将验证 `Job` 以状态"COMPLETED"结束：

基于XML的配置


```java
@SpringBatchTest
@RunWith(SpringRunner.class)
@ContextConfiguration(locations = { "/simple-job-launcher-context.xml",
                                    "/jobs/skipSampleJob.xml" })
public class SkipSampleFunctionalTests {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    private SimpleJdbcTemplate simpleJdbcTemplate;

    @Autowired
    public void setDataSource(DataSource dataSource) {
        this.simpleJdbcTemplate = new SimpleJdbcTemplate(dataSource);
    }

    @Test
    public void testJob() throws Exception {
        simpleJdbcTemplate.update("delete from CUSTOMER");
        for (int i = 1; i <= 10; i++) {
            simpleJdbcTemplate.update("insert into CUSTOMER values (?, 0, ?, 100000)",
                                      i, "customer" + i);
        }

        JobExecution jobExecution = jobLauncherTestUtils.launchJob();


        Assert.assertEquals("COMPLETED", jobExecution.getExitStatus().getExitCode());
    }
}
```


基于Java的配置


```java
@SpringBatchTest
@RunWith(SpringRunner.class)
@ContextConfiguration(classes=SkipSampleConfiguration.class)
public class SkipSampleFunctionalTests {

    @Autowired
    private JobLauncherTestUtils jobLauncherTestUtils;

    private SimpleJdbcTemplate simpleJdbcTemplate;

    @Autowired
    public void setDataSource(DataSource dataSource) {
        this.simpleJdbcTemplate = new SimpleJdbcTemplate(dataSource);
    }

    @Test
    public void testJob() throws Exception {
        simpleJdbcTemplate.update("delete from CUSTOMER");
        for (int i = 1; i <= 10; i++) {
            simpleJdbcTemplate.update("insert into CUSTOMER values (?, 0, ?, 100000)",
                                      i, "customer" + i);
        }

        JobExecution jobExecution = jobLauncherTestUtils.launchJob();


        Assert.assertEquals("COMPLETED", jobExecution.getExitStatus().getExitCode());
    }
}
```



### 1.3.测试个别步骤

对于复杂的批处理作业，端到端测试方法中的测试用例可能变得难以管理 . 在这些情况下，让测试用例自行测试各个步骤可能更有用 .   `JobLauncherTestUtils` 类包含一个名为 `launchStep` 的方法，该方法采用步骤名称并仅运行该特定的 `Step`  . 这种方法允许更有针对性的测试，让测试只为该步骤设置数据并直接验证其结果 . 以下示例显示如何使用 `launchStep` 方法按名称加载 `Step` ：


```java
JobExecution jobExecution = jobLauncherTestUtils.launchStep("loadFileStep");
```



### 1.4.测试Step-Scoped组件

通常，在运行时为您的步骤配置的组件使用步骤范围和后期绑定来从步骤或作业执行中注入上下文 . 除非您有办法将上下文设置为执行步骤，否则将这些组件作为独立组件进行测试非常棘手 . 这是Spring Batch中两个组件的目标： `StepScopeTestExecutionListener` 和 `StepScopeTestUtils`  . 

侦听器在类级别声明，其作用是为每个测试方法创建步骤执行上下文，如以下示例所示：


```java
@ContextConfiguration
@TestExecutionListeners( { DependencyInjectionTestExecutionListener.class,
    StepScopeTestExecutionListener.class })
@RunWith(SpringRunner.class)
public class StepScopeTestExecutionListenerIntegrationTests {

    // This component is defined step-scoped, so it cannot be injected unless
    // a step is active...
    @Autowired
    private ItemReader<String> reader;

    public StepExecution getStepExecution() {
        StepExecution execution = MetaDataInstanceFactory.createStepExecution();
        execution.getExecutionContext().putString("input.data", "foo,bar,spam");
        return execution;
    }

    @Test
    public void testReader() {
        // The reader is initialized and bound to the input data
        assertNotNull(reader.read());
    }

}
```


有两个 `TestExecutionListeners`  . 一个是常规的Spring Test框架，它处理来自配置的应用程序上下文的依赖注入以注入读取器 . 另一个是Spring Batch  `StepScopeTestExecutionListener`  . 它的工作原理是在测试用例中查找 `StepExecution` 的工厂方法，将其用作测试方法的上下文，就好像该执行在运行时的 `Step` 中是活动的一样 . 工厂方法由其签名检测（必须返回 `StepExecution` ） . 如果未提供工厂方法，则会创建默认 `StepExecution`  . 

从v4.1开始，如果测试类使用 `@SpringBatchTest` 注释，则 `StepScopeTestExecutionListener` 和 `JobScopeTestExecutionListener` 将作为测试执行侦听器导入 . 前面的测试示例可以配置如下：


```java
@SpringBatchTest
@RunWith(SpringRunner.class)
@ContextConfiguration
public class StepScopeTestExecutionListenerIntegrationTests {

    // This component is defined step-scoped, so it cannot be injected unless
    // a step is active...
    @Autowired
    private ItemReader<String> reader;

    public StepExecution getStepExecution() {
        StepExecution execution = MetaDataInstanceFactory.createStepExecution();
        execution.getExecutionContext().putString("input.data", "foo,bar,spam");
        return execution;
    }

    @Test
    public void testReader() {
        // The reader is initialized and bound to the input data
        assertNotNull(reader.read());
    }

}
```


如果希望步骤范围的持续时间是测试方法的执行，则监听方法很方便 . 对于更灵活但更具侵入性的方法，您可以使用 `StepScopeTestUtils`  . 以下示例计算上一示例中显示的阅读器中可用的项目数：


```java
int count = StepScopeTestUtils.doInStepScope(stepExecution,
    new Callable<Integer>() {
      public Integer call() throws Exception {

        int count = 0;

        while (reader.read() != null) {
           count++;
        }
        return count;
    }
});
```



### 1.5.验证输出文件

当批处理作业写入数据库时，很容易查询数据库以验证输出是否符合预期 . 但是，如果批处理作业写入文件，则验证输出同样重要 .  Spring Batch提供了一个名为 `AssertFile` 的类，以便于验证输出文件 . 名为 `assertFileEquals` 的方法需要两个 `File` 对象（或两个 `Resource` 对象），并逐行断言这两个文件具有相同的内容 . 因此，可以创建具有预期输出的文件，并将其与实际结果进行比较，如以下示例所示：


```java
private static final String EXPECTED_FILE = "src/main/resources/data/input.txt";
private static final String OUTPUT_FILE = "target/test-outputs/output.txt";

AssertFile.assertFileEquals(new FileSystemResource(EXPECTED_FILE),
                            new FileSystemResource(OUTPUT_FILE));
```



### 1.6.模拟域对象

编写Spring Batch组件的单元和集成测试时遇到的另一个常见问题是如何模拟域对象 . 一个很好的例子是 `StepExecutionListener` ，如下面的代码片段所示：


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


前面的侦听器示例由框架提供，并检查 `StepExecution` 是否为空读取计数，从而表示没有完成任何工作 . 虽然这个例子非常简单，但它用于说明在尝试单元测试实现需要Spring Batch域对象的接口的类时可能遇到的问题类型 . 在前面的示例中，请考虑以下针对侦听器的单元测试：


```java
private NoWorkFoundStepExecutionListener tested = new NoWorkFoundStepExecutionListener();

@Test
public void noWork() {
    StepExecution stepExecution = new StepExecution("NoProcessingStep",
                new JobExecution(new JobInstance(1L, new JobParameters(),
                                 "NoProcessingJob")));

    stepExecution.setExitStatus(ExitStatus.COMPLETED);
    stepExecution.setReadCount(0);

    ExitStatus exitStatus = tested.afterStep(stepExecution);
    assertEquals(ExitStatus.FAILED.getExitCode(), exitStatus.getExitCode());
}
```


因为Spring Batch域模型遵循良好的面向对象原则， `StepExecution` 需要 `JobExecution` ，这需要 `JobInstance` 和 `JobParameters` 来创建有效的 `StepExecution`  . 虽然这在实体域模型中很好，但它确实可以创建用于单元测试详细信息的存根对象 . 为解决此问题，Spring Batch测试模块包含一个用于创建域对象的工厂： `MetaDataInstanceFactory`  . 鉴于此工厂，可以更新单元测试以使其更简洁，如以下示例所示：


```java
private NoWorkFoundStepExecutionListener tested = new NoWorkFoundStepExecutionListener();

@Test
public void testAfterStep() {
    StepExecution stepExecution = MetaDataInstanceFactory.createStepExecution();

    stepExecution.setExitStatus(ExitStatus.COMPLETED);
    stepExecution.setReadCount(0);

    ExitStatus exitStatus = tested.afterStep(stepExecution);
    assertEquals(ExitStatus.FAILED.getExitCode(), exitStatus.getExitCode());
}
```


上述创建简单 `StepExecution` 的方法只是工厂内可用的一种便捷方法 . 完整的方法列表可以在[Javadoc](https://docs.spring.io/spring-batch/apidocs/org/springframework/batch/test/MetaDataInstanceFactory.html)中找到 .
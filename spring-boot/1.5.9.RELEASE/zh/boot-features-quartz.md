## 40. Quartz Scheduler

Spring Boot提供了一些使用[Quartz scheduler](http://www.quartz-scheduler.org/)的便利，包括 `spring-boot-starter-quartz` “Starter”.如果Quartz可用，则自动配置 `Scheduler` （通过 `SchedulerFactoryBean` 抽象）.

自动拾取以下类型的Bean并将其与 `Scheduler` 关联：

-  `JobDetail` ：定义一个特定的工作.  `JobDetail` 实例可以使用 `JobBuilder`  API构建.

-  `Calendar` .

-  `Trigger` ：定义何时触发特定作业.

默认情况下，使用内存中的 `JobStore` .但是，如果应用程序中有 `DataSource`  bean，并且相应地配置了 `spring.quartz.job-store-type` 属性，则可以配置基于JDBC的存储，如以下示例所示：

```java
spring.quartz.job-store-type=jdbc
```

使用JDBC存储时，可以在启动时初始化架构，如以下示例所示：

```java
spring.quartz.jdbc.initialize-schema=always
```

> By默认情况下，使用Quartz库提供的标准脚本检测并初始化数据库.也可以通过设置 `spring.quartz.jdbc.schema` 属性来提供自定义脚本.

要让Quartz使用 `DataSource` 而不是应用程序的主 `DataSource` ，请声明一个 `DataSource`  bean，用 `@QuartzDataSource` 注释其 `@Bean` 方法.这样做可确保 `SchedulerFactoryBean` 和架构初始化都使用特定于Quartz的 `DataSource` .

默认情况下，配置创建的作业不会覆盖已从永久性作业存储区读取的已注册作业.要启用覆盖现有作业定义，请设置 `spring.quartz.overwrite-existing-jobs` 属性.

可以使用 `spring.quartz` 属性和 `SchedulerFactoryBeanCustomizer`  bean自定义Quartz Scheduler配置，这允许程序化 `SchedulerFactoryBean` 自定义.可以使用 `spring.quartz.properties.*` 自定义高级Quartz配置属性.

> 特别是， `Executor`  bean与调度程序无关，因为Quartz提供了一种通过 `spring.quartz.properties` 配置调度程序的方法.如果需要自定义任务执行程序，请考虑实现 `SchedulerFactoryBeanCustomizer` .

作业可以定义setter以注入数据映射属性.也可以以类似的方式注入常规bean，如以下示例所示：

```java
public class SampleJob extends QuartzJobBean {

	private MyService myService;

	private String name;

	// Inject "MyService" bean
	public void setMyService(MyService myService) { ... }

	// Inject the "name" job data property
	public void setName(String name) { ... }

	@Override
	protected void executeInternal(JobExecutionContext context)
			throws JobExecutionException {
		...
	}

}
```

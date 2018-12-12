## 41.任务执行和调度

在上下文中没有 `TaskExecutor`  bean的情况下，Spring Boot会自动配置 `ThreadPoolTaskExecutor` ，其中包含可以自动关联异步任务执行（ `@EnableAsync` ）和Spring MVC异步请求处理的合理默认值.

线程池使用8个核心线程，可根据负载增长和缩小.可以使用 `spring.task.execution` 命名空间对这些默认设置进行微调，如以下示例所示：

```java
spring.task.execution.pool.max-threads=16
spring.task.execution.pool.queue-capacity=100
spring.task.execution.pool.keep-alive=10s
```

这会将线程池更改为使用有界队列，以便在队列满（100个任务）时，线程池增加到最多16个线程.当线程在闲置10秒（而不是默认为60秒）时回收线程时，池的收缩会更加激进.

如果需要与计划任务执行相关联， `ThreadPoolTaskScheduler` 也可以自动配置（ `@EnableScheduling` ）.默认情况下，线程池使用一个线程，并且可以使用 `spring.task.scheduling` 命名空间对这些设置进行微调.

如果需要创建自定义执行程序或调度程序，则在上下文中可以使用 `TaskExecutorBuilder`  bean和 `TaskSchedulerBuilder`  bean.

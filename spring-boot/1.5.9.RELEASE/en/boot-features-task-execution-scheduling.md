## 41. Task Execution and Scheduling

In the absence of a  `TaskExecutor`  bean in the context, Spring Boot auto-configures a  `ThreadPoolTaskExecutor`  with sensible defaults that can be automatically associated to asynchronous task execution ( `@EnableAsync` ) and Spring MVC asynchronous request processing.

The thread pool uses 8 core threads that can grow and shrink according to the load. Those default settings can be fine-tuned using the  `spring.task.execution`  namespace as shown in the following example:

```java
spring.task.execution.pool.max-threads=16
spring.task.execution.pool.queue-capacity=100
spring.task.execution.pool.keep-alive=10s
```

This changes the thread pool to use a bounded queue so that when the queue is full (100 tasks), the thread pool increases to maximum 16 threads. Shrinking of the pool is more aggressive as threads are reclaimed when they are idle for 10 seconds (rather than 60 seconds by default).

A  `ThreadPoolTaskScheduler`  can also be auto-configured if need to be associated to scheduled task execution ( `@EnableScheduling` ). The thread pool uses one thread by default and those settings can be fine-tuned using the  `spring.task.scheduling`  namespace.

Both a  `TaskExecutorBuilder`  bean and a  `TaskSchedulerBuilder`  bean are made available in the context if a custom executor or scheduler needs to be created.

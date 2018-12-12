## 87.批量申请

本节回答使用Spring Batch with Spring Boot产生的问题.

> By默认情况下，批处理应用程序需要 `DataSource` 来存储作业详细信息.如果你想偏离它，你需要实现 `BatchConfigurer` .有关详细信息，请参阅[The Javadoc of @EnableBatchProcessing](https://docs.spring.io/spring-batch/apidocs/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html).

有关Spring Batch的更多信息，请参阅[Spring Batch project page](https://projects.spring.io/spring-batch/).

## 87.1在启动时执行Spring Batch作业

通过在上下文中的某处添加 `@EnableBatchProcessing` （来自Spring Batch）来启用Spring Batch自动配置.

默认情况下，它在启动时在应用程序上下文中执行 **all**   `Jobs` （有关详细信息，请参阅[JobLauncherCommandLineRunner](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/JobLauncherCommandLineRunner.java)）.您可以通过指定 `spring.batch.job.names` （以逗号分隔的作业名称模式列表）缩小到特定作业或作业的范围.

如果应用程序上下文包含 `JobRegistry` ，则 `spring.batch.job.names` 中的作业将在注册表中查找，而不是从上下文自动装配.这是一个具有更复杂系统的常见模式，其中多个作业在子上下文中定义并集中注册.

有关详细信息，请参阅[BatchAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java)和[@EnableBatchProcessing](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.java).


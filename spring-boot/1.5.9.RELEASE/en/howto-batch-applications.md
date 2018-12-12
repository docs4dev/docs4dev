## 87. Batch Applications

This section answers questions that arise from using Spring Batch with Spring Boot.

> By default, batch applications require a  `DataSource`  to store job details. If you want to deviate from that, you need to implement  `BatchConfigurer` . See [The Javadoc of @EnableBatchProcessing](https://docs.spring.io/spring-batch/apidocs/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.html) for more details.

For more about Spring Batch, see the [Spring Batch project page](https://projects.spring.io/spring-batch/).

## 87.1 Execute Spring Batch Jobs on Startup

Spring Batch auto-configuration is enabled by adding  `@EnableBatchProcessing`  (from Spring Batch) somewhere in your context.

By default, it executes  **all**   `Jobs`  in the application context on startup (see [JobLauncherCommandLineRunner](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/JobLauncherCommandLineRunner.java) for details). You can narrow down to a specific job or jobs by specifying  `spring.batch.job.names`  (which takes a comma-separated list of job name patterns).

If the application context includes a  `JobRegistry` , the jobs in  `spring.batch.job.names`  are looked up in the registry instead of being autowired from the context. This is a common pattern with more complex systems, where multiple jobs are defined in child contexts and registered centrally.

See [BatchAutoConfiguration](https://github.com/spring-projects/spring-boot/tree/v2.1.0.RELEASE/spring-boot-project/spring-boot-autoconfigure/src/main/java/org/springframework/boot/autoconfigure/batch/BatchAutoConfiguration.java) and [@EnableBatchProcessing](https://github.com/spring-projects/spring-batch/blob/master/spring-batch-core/src/main/java/org/springframework/batch/core/configuration/annotation/EnableBatchProcessing.java) for more details.


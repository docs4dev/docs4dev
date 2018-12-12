## Appendix A：元数据模式


### A.1 . 概观

Spring Batch Metadata表与Java中表示它们的Domain对象紧密匹配 . 例如， `JobInstance` ， `JobExecution` ， `JobParameters` 和 `StepExecution` 分别映射到 `BATCH_JOB_INSTANCE` ， `BATCH_JOB_EXECUTION` ， `BATCH_JOB_EXECUTION_PARAMS` 和 `BATCH_STEP_EXECUTION`  .   `ExecutionContext` 映射到 `BATCH_JOB_EXECUTION_CONTEXT` 和 `BATCH_STEP_EXECUTION_CONTEXT`  .   `JobRepository` 负责将每个Java对象保存并存储到正确的表中 . 本附录详细描述了元数据表，以及创建它们时做出的许多设计决策 . 在查看下面的各种表创建语句时，重要的是要认识到所使用的数据类型尽可能通用 .  Spring Batch提供了许多模式作为示例，由于各个数据库供应商处理数据类型的方式不同，所有模式都具有不同的数据类型 . 下图显示了所有6个表的ERD模型及其相互之间的关系：


![Spring Batch Meta-Data ERD](https://www.docs4dev.com/images/58a0ff31-cd97-44a1-a1ad-bbaf97dc1d8e.png)


图1. Spring Batch元数据ERD


#### A.1.1 . 示例DDL脚本

Spring Batch Core JAR文件包含用于为许多数据库平台创建关系表的示例脚本（这些平台又由作业存储库工厂bean或命名空间等效项自动检测） . 这些脚本可以按原样使用，也可以根据需要使用其他索引和约束进行修改 . 文件名的格式为 `schema-*.sql` ，其中"*"是目标数据库平台的简称 . 脚本位于包 `org.springframework.batch.core` 中 . 


#### A.1.2 . 迁移DDL脚本

Spring Batch提供了升级版本时需要执行的迁移DDL脚本 . 这些脚本可以在 `org/springframework/batch/core/migration` 下的Core Jar文件中找到 . 迁移脚本被组织到与引入它们的版本号对应的文件夹中：


- 
 `2.2` ：包含从 `2.2` 之前的版本迁移到版本 `2.2` 所需的脚本


- 
 `4.1` ：包含从 `4.1` 之前的版本迁移到版本 `4.1` 所需的脚本


#### A.1.3 . 版

本附录中讨论的许多数据库表都包含一个版本列 . 此列很重要，因为Spring Batch在处理数据库更新时采用了乐观锁定策略 . 这意味着每次“触摸”（更新）记录时，版本列中的值都会增加1 . 当存储库返回以保存该值时，如果版本号已更改，则会抛出 `OptimisticLockingFailureException` ，表示并发访问时出错 . 此检查是必要的，因为即使不同的批处理作业可能在不同的机器上运行，它们都使用相同的数据库表 . 


#### A.1.4 . 身分

 `BATCH_JOB_INSTANCE` ， `BATCH_JOB_EXECUTION` 和 `BATCH_STEP_EXECUTION` 每个都包含以 `_ID` 结尾的列 . 这些字段充当各自表的主键 . 但是，它们不是数据库生成的密钥 . 相反，它们是由单独的序列生成的 . 这是必要的，因为在将一个域对象插入数据库之后，需要在实际对象上设置它所给出的密钥，以便可以在Java中唯一地标识它们 . 较新的数据库驱动程序（JDBC 3.0及更高版本）通过数据库生成的密钥支持此功能 . 但是，不是要求该特征，而是使用序列 . 架构的每个变体都包含以下语句的某种形式：


```java
CREATE SEQUENCE BATCH_STEP_EXECUTION_SEQ;
CREATE SEQUENCE BATCH_JOB_EXECUTION_SEQ;
CREATE SEQUENCE BATCH_JOB_SEQ;
```


许多数据库供应商不支持序列 . 在这些情况下，使用解决方法，例如MySQL的以下语句：


```java
CREATE TABLE BATCH_STEP_EXECUTION_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_STEP_EXECUTION_SEQ values(0);
CREATE TABLE BATCH_JOB_EXECUTION_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_JOB_EXECUTION_SEQ values(0);
CREATE TABLE BATCH_JOB_SEQ (ID BIGINT NOT NULL) type=InnoDB;
INSERT INTO BATCH_JOB_SEQ values(0);
```


在前面的情况中，使用表来代替每个序列 .  Spring核心类 `MySQLMaxValueIncrementer` 然后按此顺序递增一列以提供类似的功能 . 


### A.2 . BATCH_JOB_INSTANCE

 `BATCH_JOB_INSTANCE` 表包含与 `JobInstance` 相关的所有信息，并作为整体层次结构的顶部 . 以下通用DDL语句用于创建它：


```java
CREATE TABLE BATCH_JOB_INSTANCE  (
  JOB_INSTANCE_ID BIGINT  PRIMARY KEY ,
  VERSION BIGINT,
  JOB_NAME VARCHAR(100) NOT NULL ,
  JOB_KEY VARCHAR(2500)
);
```


以下列表描述了表中的每一列：


- 
 `JOB_INSTANCE_ID` ：标识实例的唯一ID . 它也是主要关键 . 应通过调用 `JobInstance` 上的 `getId` 方法获取此列的值 . 


- 
 `VERSION` ：见[Version](#metaDataVersion) . 


- 
 `JOB_NAME` ：从 `Job` 对象获取的作业的名称 . 因为需要标识实例，所以它不能为null . 


- 
 `JOB_KEY` ： `JobParameters` 的序列化，它唯一地标识相同作业的不同实例 .  （ `JobInstances` 具有相同的作业名称必须具有不同的 `JobParameters` ，因此，不同的 `JOB_KEY` 值） . 


### A.3 . BATCH_JOB_EXECUTION_PARAMS

 `BATCH_JOB_EXECUTION_PARAMS` 表包含与 `JobParameters` 对象相关的所有信息 . 它包含传递给 `Job` 的0个或更多键/值对，并用作运行作业的参数的记录 . 对于有助于生成作业标识的每个参数， `IDENTIFYING` 标志设置为true . 请注意，该表已被非规范化 . 不是为每种类型创建单独的表，而是有一个表，其中有一列指示类型，如下面的清单所示：


```java
CREATE TABLE BATCH_JOB_EXECUTION_PARAMS  (
        JOB_EXECUTION_ID BIGINT NOT NULL ,
        TYPE_CD VARCHAR(6) NOT NULL ,
        KEY_NAME VARCHAR(100) NOT NULL ,
        STRING_VAL VARCHAR(250) ,
        DATE_VAL DATETIME DEFAULT NULL ,
        LONG_VAL BIGINT ,
        DOUBLE_VAL DOUBLE PRECISION ,
        IDENTIFYING CHAR(1) NOT NULL ,
        constraint JOB_EXEC_PARAMS_FK foreign key (JOB_EXECUTION_ID)
        references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
);
```


以下列表描述了每一列：


- 
 `JOB_EXECUTION_ID` ： `BATCH_JOB_EXECUTION` 表中的外键，指示参数条目所属的作业执行 . 请注意，每次执行可能存在多行（即键/值对） . 


- 
TYPE_CD：存储的值类型的字符串表示形式，可以是字符串，日期，长整型或双精度型 . 因为必须知道类型，所以它不能为null . 


- 
KEY_NAME：参数键 . 


- 
STRING_VAL：参数值，如果类型是字符串 . 


- 
DATE_VAL：参数值，如果类型是日期 . 


- 
LONG_VAL：参数值，如果类型为long . 


- 
DOUBLE_VAL：参数值，如果类型为double . 


- 
IDENTIFYING：标志，指示参数是否有助于相关 `JobInstance` 的标识 . 

请注意，此表没有主键 . 这是因为框架对一个框架没有用处，因此不需要它 . 如果需要，可以添加一个主键，可以使用数据库生成的键添加，而不会对框架本身造成任何问题 . 


### A.4 . BATCH_JOB_EXECUTION

 `BATCH_JOB_EXECUTION` 表包含与 `JobExecution` 对象相关的所有信息 . 每次运行 `Job` 时，总会有一个新的 `JobExecution` ，并且该表中有一个新行 . 以下清单显示了 `BATCH_JOB_EXECUTION` 表的定义：


```java
CREATE TABLE BATCH_JOB_EXECUTION  (
  JOB_EXECUTION_ID BIGINT  PRIMARY KEY ,
  VERSION BIGINT,
  JOB_INSTANCE_ID BIGINT NOT NULL,
  CREATE_TIME TIMESTAMP NOT NULL,
  START_TIME TIMESTAMP DEFAULT NULL,
  END_TIME TIMESTAMP DEFAULT NULL,
  STATUS VARCHAR(10),
  EXIT_CODE VARCHAR(20),
  EXIT_MESSAGE VARCHAR(2500),
  LAST_UPDATED TIMESTAMP,
  JOB_CONFIGURATION_LOCATION VARCHAR(2500) NULL,
  constraint JOB_INSTANCE_EXECUTION_FK foreign key (JOB_INSTANCE_ID)
  references BATCH_JOB_INSTANCE(JOB_INSTANCE_ID)
) ;
```


以下列表描述了每一列：


- 
 `JOB_EXECUTION_ID` ：唯一标识此执行的主键 . 可以通过调用 `JobExecution` 对象的 `getId` 方法获得此列的值 . 


- 
 `VERSION` ：见[Version](#metaDataVersion) . 


- 
 `JOB_INSTANCE_ID` ： `BATCH_JOB_INSTANCE` 表中的外键 . 它指示此执行所属的实例 . 每个实例可能有多个执行 . 


- 
 `CREATE_TIME` ：表示创建执行时间的时间戳 . 


- 
 `START_TIME` ：表示执行开始时间的时间戳 . 


- 
 `END_TIME` ：表示执行完成的时间的时间戳，无论成功或失败 . 当作业当前未运行时，此列中的空值表示存在某种类型的错误，并且框架在失败之前无法执行上次保存 . 


- 
 `STATUS` ：表示执行状态的字符串 . 这可能是 `COMPLETED` ， `STARTED` 等 . 此列的对象表示形式是 `BatchStatus` 枚举 . 


- 
 `EXIT_CODE` ：表示执行退出代码的字符串 . 在命令行作业的情况下，可以将其转换为数字 . 


- 
 `EXIT_MESSAGE` ：字符串表示作业如何退出的更详细描述 . 在失败的情况下，这可能包括尽可能多的堆栈跟踪 . 


- 
 `LAST_UPDATED` ：表示此执行最后一次持久化的时间戳 . 


### A.5 . BATCH_STEP_EXECUTION

BATCH_STEP_EXECUTION表包含与 `StepExecution` 对象相关的所有信息 . 此表在很多方面与 `BATCH_JOB_EXECUTION` 表类似，并且对于每个创建的 `JobExecution` ，每个 `Step` 始终至少有一个条目 . 以下清单显示了 `BATCH_STEP_EXECUTION` 表的定义：


```java
CREATE TABLE BATCH_STEP_EXECUTION  (
  STEP_EXECUTION_ID BIGINT  PRIMARY KEY ,
  VERSION BIGINT NOT NULL,
  STEP_NAME VARCHAR(100) NOT NULL,
  JOB_EXECUTION_ID BIGINT NOT NULL,
  START_TIME TIMESTAMP NOT NULL ,
  END_TIME TIMESTAMP DEFAULT NULL,
  STATUS VARCHAR(10),
  COMMIT_COUNT BIGINT ,
  READ_COUNT BIGINT ,
  FILTER_COUNT BIGINT ,
  WRITE_COUNT BIGINT ,
  READ_SKIP_COUNT BIGINT ,
  WRITE_SKIP_COUNT BIGINT ,
  PROCESS_SKIP_COUNT BIGINT ,
  ROLLBACK_COUNT BIGINT ,
  EXIT_CODE VARCHAR(20) ,
  EXIT_MESSAGE VARCHAR(2500) ,
  LAST_UPDATED TIMESTAMP,
  constraint JOB_EXECUTION_STEP_FK foreign key (JOB_EXECUTION_ID)
  references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ;
```


以下列表描述了每个列：


- 
 `STEP_EXECUTION_ID` ：唯一标识此执行的主键 . 应通过调用 `StepExecution` 对象的 `getId` 方法获取此列的值 . 


- 
 `VERSION` ：见[Version](#metaDataVersion) . 


- 
 `STEP_NAME` ：此执行所属步骤的名称 . 


- 
 `JOB_EXECUTION_ID` ： `BATCH_JOB_EXECUTION` 表中的外键 . 它表示 `StepExecution` 所属的 `JobExecution`  . 对于给定的 `Step` 名称，给定 `JobExecution` 可能只有一个 `StepExecution`  . 


- 
 `START_TIME` ：表示执行开始时间的时间戳 . 


- 
 `END_TIME` ：时间戳，表示执行完成的时间，无论成功与否 . 即使作业当前未运行，此列中的空值也表示存在某种类型的错误，并且框架在失败之前无法执行上次保存 . 


- 
 `STATUS` ：表示执行状态的字符串 . 这可能是 `COMPLETED` ， `STARTED` 等 . 此列的对象表示是 `BatchStatus` 枚举 . 


- 
 `COMMIT_COUNT` ：步骤在执行期间提交事务的次数 . 


- 
 `READ_COUNT` ：执行期间读取的项目数 . 


- 
 `FILTER_COUNT` ：从此执行中过滤掉的项目数 . 


- 
 `WRITE_COUNT` ：执行期间写入和提交的项目数 . 


- 
 `READ_SKIP_COUNT` ：执行期间跳过的项目数 . 


- 
 `WRITE_SKIP_COUNT` ：执行期间写入时跳过的项目数 . 


- 
 `PROCESS_SKIP_COUNT` ：执行期间处理期间跳过的项目数 . 


- 
 `ROLLBACK_COUNT` ：执行期间的回滚次数 . 请注意，此计数包括每次发生回滚时，包括重试的回滚和跳过恢复过程中的回滚 . 


- 
 `EXIT_CODE` ：表示执行退出代码的字符串 . 在命令行作业的情况下，可以将其转换为数字 . 


- 
 `EXIT_MESSAGE` ：字符串，表示作业如何退出的更详细说明 . 在失败的情况下，这可能包括尽可能多的堆栈跟踪 . 


- 
 `LAST_UPDATED` ：表示上次执行持久化的时间戳 . 


### A.6 . BATCH_JOB_EXECUTION_CONTEXT

 `BATCH_JOB_EXECUTION_CONTEXT` 表包含与 `Job` 的 `ExecutionContext` 相关的所有信息 . 每个 `JobExecution` 只有一个 `Job`   `ExecutionContext` ，它包含特定作业执行所需的所有作业级数据 . 此数据通常表示发生故障后必须检索的状态，以便 `JobInstance` 可以"start from where it left off" . 以下清单显示了 `BATCH_JOB_EXECUTION_CONTEXT` 表的定义：


```java
CREATE TABLE BATCH_JOB_EXECUTION_CONTEXT  (
  JOB_EXECUTION_ID BIGINT PRIMARY KEY,
  SHORT_CONTEXT VARCHAR(2500) NOT NULL,
  SERIALIZED_CONTEXT CLOB,
  constraint JOB_EXEC_CTX_FK foreign key (JOB_EXECUTION_ID)
  references BATCH_JOB_EXECUTION(JOB_EXECUTION_ID)
) ;
```


以下列表描述了每一列：


- 
 `JOB_EXECUTION_ID` ：表示上下文所属的 `JobExecution` 的外键 . 可能存在与给定执行相关联的多个行 . 


- 
 `SHORT_CONTEXT` ： `SERIALIZED_CONTEXT` 的字符串版本 . 


- 
 `SERIALIZED_CONTEXT` ：整个上下文，序列化 . 


### A.7 . BATCH_STEP_EXECUTION_CONTEXT

 `BATCH_STEP_EXECUTION_CONTEXT` 表包含与 `Step` 的 `ExecutionContext` 相关的所有信息 . 每个 `StepExecution` 只有一个 `ExecutionContext` ，它包含特定步骤执行需要持久化的所有数据 . 此数据通常表示失败后必须检索的状态，以便 `JobInstance` 可以“从停止的位置开始” . 以下清单显示了 `BATCH_STEP_EXECUTION_CONTEXT` 表的定义：


```java
CREATE TABLE BATCH_STEP_EXECUTION_CONTEXT  (
  STEP_EXECUTION_ID BIGINT PRIMARY KEY,
  SHORT_CONTEXT VARCHAR(2500) NOT NULL,
  SERIALIZED_CONTEXT CLOB,
  constraint STEP_EXEC_CTX_FK foreign key (STEP_EXECUTION_ID)
  references BATCH_STEP_EXECUTION(STEP_EXECUTION_ID)
) ;
```


以下列表描述了每一列：


- 
 `STEP_EXECUTION_ID` ：表示上下文所属的 `StepExecution` 的外键 . 可能存在与给定执行相关联的多个行 . 


- 
 `SHORT_CONTEXT` ： `SERIALIZED_CONTEXT` 的字符串版本 . 


- 
 `SERIALIZED_CONTEXT` ：整个上下文，序列化 . 


### A.8 . 存档

由于每次运行批处理作业时都有多个表中的条目，因此通常会为元数据表创建存档策略 . 这些表本身旨在显示过去发生的事件的记录，通常不会影响任何作业的运行，但有一些与重启有关的明显例外：


- 
框架使用元数据表来确定之前是否运行过特定的 `JobInstance`  . 如果已运行并且作业不可重新启动，则抛出异常 . 


- 
如果在没有成功完成的情况下删除 `JobInstance` 的条目，框架会认为这项工作是新的而不是重新启动 . 


- 
如果重新启动作业，框架将使用已持久保存到 `ExecutionContext` 的任何数据来恢复 `Job’s` 状态 . 因此，如果未成功完成的作业从此表中删除任何条目，则会阻止它们在正确的位置启动（如果再次运行） . 


### A.9 . 国际和多字节字符

如果在业务处理中使用多字节字符集（例如中文或西里尔字符集），则可能需要在Spring Batch模式中保留这些字符 . 许多用户发现只需将模式更改为 `VARCHAR` 列长度的两倍就足够了 . 其他人更喜欢将[JobRepository](job.html#configuringJobRepository)配置为 `VARCHAR` 列长度的一半 `max-varchar-length`  . 一些用户还报告说他们在模式定义中使用 `NVARCHAR` 代替 `VARCHAR` . 最佳结果取决于数据库平台以及本地配置数据库服务器的方式 . 


### A.10 . 索引元数据表的建议

Spring Batch为几个常见数据库平台的核心jar文件中的元数据表提供DDL示例 . 索引声明不包含在该DDL中，因为用户可能希望索引的变化太多，具体取决于其精确平台，本地约定以及作业运行方式的业务要求 . 下面提供了一些指示，指出Spring Batch提供的DAO实现将在 `WHERE` 子句中使用哪些列以及它们的使用频率，以便各个项目可以自己构建关于索引的思路：

||
|默认表名| Where子句|频率|
| ---- | ---- | ---- |
| BATCH_JOB_INSTANCE | JOB_NAME =？和JOB_KEY =？ |每次启动工作|
| BATCH_JOB_EXECUTION | JOB_INSTANCE_ID =？ |每次重新启动作业时|
| BATCH_EXECUTION_CONTEXT | EXECUTION_ID =？和KEY_NAME =？ |在提交间隔，a.k.a . 块|
| BATCH_STEP_EXECUTION | VERSION =？ |在提交间隔，a.k.a . 块（以及步骤的开始和结束）|
| BATCH_STEP_EXECUTION | STEP_NAME =？和JOB_EXECUTION_ID =？ |在每个步骤执行之前|
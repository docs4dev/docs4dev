## Appendix A：ItemReaders和ItemWriters的列表


### A.1 . 项目读者

||
|项目读者|描述|
| ---- | ---- |
| AbstractItemCountingItemStreamItemReader |抽象基类，通过计算从 `ItemReader` 返回的项数来提供基本的重新启动功能 .  |
| AggregateItemReader |  `ItemReader` 以列表作为项目，存储注入的 `ItemReader` 中的对象，直到它们准备好作为集合打包 . 此 `ItemReader` 应使用 `FieldSetMapper AggregateItemReader#BEGIN_RECORD` 和 `AggregateItemReader#END_RECORD` 中的常量值标记记录的开头和结尾 . |
| AmqpItemReader |给定一个Spring  `AmqpTemplate` ，它提供了同步接收方法 .   `receiveAndConvert()` 方法允许您接收POJO对象 .  |
| FlatFileItemReader |从平面文件中读取 . 包括 `ItemStream` 和 `Skippable` 功能 . 见[FlatFileItemReader](readersAndWriters.html#flatFileItemReader) . |
| HibernateCursorItemReader |基于HQL查询从游标读取 . 见[Cursor-based ItemReaders](readersAndWriters.html#cursorBasedItemReaders) . |
| HibernatePagingItemReader |从分页的HQL查询中读取|
| ItemReaderAdapter |将任何类调整为 `ItemReader` 接口 .  |
| JdbcCursorItemReader |通过JDBC从数据库游标读取 . 见[Cursor-based ItemReaders](readersAndWriters.html#cursorBasedItemReaders) . |
| JdbcPagingItemReader |给定一个SQL语句，遍历行的页面，以便可以在不耗尽内存的情况下读取大型数据集 .  |
| JmsItemReader |给定Spring  `JmsOperations` 对象和要向其发送错误的JMS目标或目标名称，提供通过注入的 `JmsOperations#receive()` 方法接收的项目 . |
| JpaPagingItemReader |给定JPQL语句，遍历行的页面，以便可以在不耗尽内存的情况下读取大型数据集 .  |
| ListItemReader |一次一个地提供列表中的项目 .  |
| MongoItemReader |给定 `MongoOperations` 对象和基于JSON的MongoDB查询，提供从 `MongoOperations#find()` 方法接收的项目 . |
| Neo4jItemReader |给定一个 `Neo4jOperations` 对象和Cyhper查询的组件，项目将作为Neo4jOperations.query方法的结果返回 .  |
| RepositoryItemReader |给定一个Spring Data  `PagingAndSortingRepository` 对象，一个 `Sort` ，以及要执行的方法的名称，返回Spring Data存储库实现提供的项目 .  |
| StoredProcedureItemReader |从执行数据库存储过程产生的数据库游标中读取 . 见[StoredProcedureItemReader](readersAndWriters.html#StoredProcedureItemReader) |
| StaxEventItemReader |通过StAX读取 . 见[StaxEventItemReader](readersAndWriters.html#StaxEventItemReader) . |
| JsonItemReader |从Json文档中读取项目 . 见[JsonItemReader](readersAndWriters.html#JsonItemReader) . |


### A.2 . 物品作家

||
|项目编写者|描述|
| ---- | ---- |
| AbstractItemStreamItemWriter |结合 `ItemStream` 和 . 的抽象基类 `ItemWriter` 接口 .  |
| AmqpItemWriter |给定一个Spring  `AmqpTemplate` ，它提供了一个同步 `send` 方法 .   `convertAndSend(Object)` 方法允许您发送POJO对象 .  |
| CompositeItemWriter |将项目传递给 `List` 个 `ItemWriter` 对象中每个项目的 `write` 方法 .  |
| FlatFileItemWriter |写入平面文件 . 包括 `ItemStream` 和可跳过的功能 . 见[FlatFileItemWriter](readersAndWriters.html#flatFileItemWriter) . |
| GemfireItemWriter |使用 `GemfireOperations` 对象，可以根据删除标志的配置从Gemfire实例中写入或删除项目 .  |
| HibernateItemWriter |这个项目编写器是Hibernate会话感知并处理一些非“hibernate-aware”项目编写者不需要知道的与事务相关的工作，然后委托给另一个项目编写者来进行实际编写 .  |
| ItemWriterAdapter |将任何类调整为 `ItemWriter` 接口 .  |
| JdbcBatchItemWriter |使用 `PreparedStatement` 中的批处理功能（如果可用），并且可以采取基本步骤在 `flush` 期间查找故障 .  |
| JmsItemWriter |使用 `JmsOperations` 对象，项目通过 `JmsOperations#convertAndSend()` 方法写入默认队列 . |
| JpaItemWriter |这个项目编写器是JPA EntityManager感知的，并处理一些非"JPA-aware"  `ItemWriter` 不需要知道的与事务相关的工作，然后委托给另一个编写者来进行实际编写 .  |
| MimeMessageItemWriter |使用Spring的 `JavaMailSender` ， `MimeMessage` 类型的项目将作为邮件消息发送 .  |
| MongoItemWriter |给定 `MongoOperations` 对象，项目通过 `MongoOperations.save(Object)` 方法写入 . 实际写入被延迟到事务提交之前的最后一刻 .  |
| Neo4jItemWriter |给定一个 `Neo4jOperations` 对象，项目通过 `save(Object)` 方法持久化，或者通过 `delete(Object)` 按 `ItemWriter’s` 配置删除 . 
| PropertyExtractingDelegatingItemWriter |动态扩展 `AbstractMethodInvokingDelegator` 创建参数 . 根据注入的字段名称数组，通过从要处理的项目（通过 `SpringBeanWrapper` ）中的字段中检索值来创建参数 .  |
| RepositoryItemWriter |给定Spring Data  `CrudRepository` 实现，项目通过配置中指定的方法保存 .  |
| StaxEventItemWriter |使用 `Marshaller` 实现将每个项目转换为XML，然后使用StAX将其写入XML文件 .  |
| JsonFileItemWriter |使用 `JsonObjectMarshaller` 实现将每个项目转换为Json，然后将其写入Json文件 .  |
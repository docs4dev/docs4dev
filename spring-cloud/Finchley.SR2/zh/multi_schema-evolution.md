## 31.架构演进支持

Spring Cloud Stream为模式演变提供支持，以便数据可以随着时间的推移而发展，并且仍然可以与较旧或较新的生产环境者和消费者一起使用，反之亦然.大多数序列化模型，特别是那些旨在跨不同平台和语言进行可移植性的模型，依赖于描述如何在二进制有效负载中序列化数据的模式.为了序列化数据然后解释它，发送方和接收方都必须能够访问描述二进制格式的模式.在某些情况下，可以从序列化的有效负载类型或反序列化的目标类型推断出模式.但是，许多应用程序可以访问描述二进制数据格式的显式模式.模式注册表允许您以文本格式（通常是JSON）存储模式信息，并使该信息可供需要它以二进制格式接收和发送数据的各种应用程序访问.模式可作为元组引用，包括：

- A主题是模式的逻辑名称

- 架构版本

- 架构格式，描述数据的二进制格式

以下部分将详细介绍模式演变过程中涉及的各个组件.

## 31.1架构注册表客户端

用于与模式注册表服务器交互的客户端抽象是 `SchemaRegistryClient` 接口，它具有以下结构：

```java
public interface SchemaRegistryClient {

SchemaRegistrationResponse register(String subject, String format, String schema);

String fetch(SchemaReference schemaReference);

String fetch(Integer id);

}
```

Spring Cloud Stream提供了开箱即用的实现，可以与自己的架构服务器进行交互，并与Confluent Schema Registry进行交互.

可以使用 `@EnableSchemaRegistryClient` 配置Spring Cloud Stream模式注册表的客户端，如下所示：

```java
@EnableBinding(Sink.class)
@SpringBootApplication
@EnableSchemaRegistryClient
public static class AvroSinkApplication {
...
}
```

> 默认转换器经过优化，不仅可以缓存来自远程服务器的模式，还可以缓存非常昂贵的 `parse()` 和 `toString()` 方法.因此，它使用不缓存响应的 `DefaultSchemaRegistryClient` .如果您打算更改默认行为，可以直接在代码上使用客户端并将其覆盖到所需的结果.为此，您必须将属性 `spring.cloud.stream.schemaRegistryClient.cached=true` 添加到应用程序属性中.

### 31.1.1架构注册表客户端属性

Schema Registry Client支持以下属性：

spring.cloud.stream.schemaRegistryClient.endpoint架构服务器的位置.设置此项时，请使用完整的URL，包括协议（http或https），端口和上下文路径.默认http：// localhost：8990 / spring.cloud.stream.schemaRegistryClient.cached客户端是否应缓存架构服务器响应.通常设置为false，因为缓存发生在消息转换器中.使用模式注册表客户端的客户端应将此设置为true.默认为true

## 31.2 Avro架构注册表客户端消息转换器

对于在应用程序上下文中注册了SchemaRegistryClient bean的应用程序，Spring Cloud Stream会自动配置Apache Avro消息转换器以进行模式管理.这样可以简化模式演变，因为接收消息的应用程序可以轻松访问可与其自己的读取器模式协调的编写器模式.

对于出站消息，如果通道的内容类型设置为 `application/*+avro` ，则会激活 `MessageConverter` ，如以下示例所示：

```java
spring.cloud.stream.bindings.output.contentType=application/*+avro
```

在出站转换期间，消息转换器尝试使用 `SchemaRegistryClient` 推断每个出站消息的模式（基于其类型）并将其注册到主题（基于有效内容类型）.如果已找到相同的模式，则检索对其的引用.如果不是，则注册模式，并提供新的版本号.使用以下方案通过 `contentType` 标头发送消息： `application/[prefix].[subject].v[version]+avro` ，其中 `prefix` 是可配置的， `subject` 是从有效负载类型推导出的.

例如， `User` 类型的消息可能作为二进制有效内容发送，内容类型为 `application/vnd.user.v2+avro` ，其中 `user` 是主题， `2` 是版本号.

接收消息时，转换器会从传入消息的标头中推断出架构引用，并尝试检索它.该模式在反序列化过程中用作编写器模式.

### 31.2.1 Avro架构注册表消息转换器属性

如果通过设置 `spring.cloud.stream.bindings.output.contentType=application/*+avro` 启用了基于Avro的架构注册表客户端，则可以通过设置以下属性来自定义注册行为.

spring.cloud.stream.schema.avro.dynamicSchemaGenerationEnabled如果希望转换器使用反射从POJO推断架构，则启用.默认值：false spring.cloud.stream.schema.avro.readerSchema Avro通过查看writer模式（origin payload）和reader schema（您的应用程序有效负载）来比较模式版本.有关更多信息，请参阅Avro文档.如果设置，则会覆盖架构服务器上的任何查找，并使用本地架构作为读取器架构.默认值：null spring.cloud.stream.schema.avro.schemaLocations使用架构服务器注册此属性中列出的所有.avsc文件.默认值：空spring.cloud.stream.schema.avro.prefix要在Content-Type标头上使用的前缀.默认值：vnd

## 31.3 Apache Avro消息转换器

Spring Cloud Stream通过其 `spring-cloud-stream-schema` 模块为基于模式的消息转换器提供支持.目前，基于模式的消息转换器开箱即用的唯一序列化格式是Apache Avro，未来版本中将添加更多格式.

`spring-cloud-stream-schema` 模块包含两种类型的消息转换器，可用于Apache Avro序列化：

- Converters使用序列化或反序列化对象的类信息或具有启动时已知位置的模式.

- 使用模式注册表的转换器.他们在运行时定位模式，并在域对象发展时动态注册新模式.

## 31.4具有架构支持的转换器

`AvroSchemaMessageConverter` 支持通过使用预定义的模式或使用类中可用的模式信息（反射或包含在 `SpecificRecord` 中）来序列化和反序列化消息.如果您提供自定义转换器，则不会创建默认的AvroSchemaMessageConverter bean.以下示例显示了自定义转换器：

要使用自定义转换器，只需将其添加到应用程序上下文中，可以选择指定一个或多个与之关联的 `MimeTypes` .默认 `MimeType` 是 `application/avro` .

如果转换的目标类型是 `GenericRecord` ，则必须设置架构.

以下示例显示如何通过在没有预定义模式的情况下注册Apache Avro  `MessageConverter` 来在接收器应用程序中配置转换器.在此示例中，请注意mime类型值为 `avro/bytes` ，而不是默认值 `application/avro` .

```java
@EnableBinding(Sink.class)
@SpringBootApplication
public static class SinkApplication {

...

@Bean
public MessageConverter userMessageConverter() {
return new AvroSchemaMessageConverter(MimeType.valueOf("avro/bytes"));
}
}
```

相反，以下应用程序使用预定义模式（在类路径中找到）注册转换器：

```java
@EnableBinding(Sink.class)
@SpringBootApplication
public static class SinkApplication {

...

@Bean
public MessageConverter userMessageConverter() {
AvroSchemaMessageConverter converter = new AvroSchemaMessageConverter(MimeType.valueOf("avro/bytes"));
converter.setSchemaLocation(new ClassPathResource("schemas/User.avro"));
return converter;
}
}
```

## 31.5架构注册服务器

Spring Cloud Stream提供架构注册服务器实现.要使用它，您可以将 `spring-cloud-stream-schema-server` 工件添加到项目中并使用 `@EnableSchemaRegistryServer` 注释，它将架构注册表服务器REST控制器添加到您的应用程序.此批注旨在与Spring Boot Web应用程序一起使用，并且服务器的侦听端口由 `server.port` 属性控制.  `spring.cloud.stream.schema.server.path` 属性可用于控制模式服务器的根路径（特别是当它嵌入到其他应用程序中时）.  `spring.cloud.stream.schema.server.allowSchemaDeletion` 布尔属性允许删除模式.默认情况下，禁用此功能.

模式注册表服务器使用关系数据库来存储模式.默认情况下，它使用嵌入式数据库.您可以使用[Spring Boot SQL database and JDBC configuration options](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-sql)自定义架构存储.

以下示例显示了启用架构注册表的Spring Boot应用程序：

```java
@SpringBootApplication
@EnableSchemaRegistryServer
public class SchemaRegistryServerApplication {
public static void main(String[] args) {
SpringApplication.run(SchemaRegistryServerApplication.class, args);
}
}
```

### 31.5.1 Schema Registry Server API

Schema Registry Server API包含以下操作：

-  `POST /`   - 见“[the section called “Registering a New Schema”](multi_schema-evolution.html#spring-cloud-stream-overview-registering-new-schema)”

- 'GET / {subject} / {format} / {version}' - 请参阅“[the section called “Retrieving an Existing Schema by Subject, Format, and Version”](multi_schema-evolution.html#spring-cloud-stream-overview-retrieve-schema-subject-format-version)”

-  `GET /{subject}/{format}`   - 请参阅“[the section called “Retrieving an Existing Schema by Subject and Format”](multi_schema-evolution.html#spring-cloud-stream-overview-retrieve-schema-subject-format)”

-  `GET /schemas/{id}`   - 请参阅“[the section called “Retrieving an Existing Schema by ID”](multi_schema-evolution.html#spring-cloud-stream-overview-retrieve-schema-id)”

-  `DELETE /{subject}/{format}/{version}`   - 见“[the section called “Deleting a Schema by Subject, Format, and Version”](multi_schema-evolution.html#spring-cloud-stream-overview-deleting-schema-subject-format-version)”

-  `DELETE /schemas/{id}`   - 请参阅“[the section called “Deleting a Schema by ID”](multi_schema-evolution.html#spring-cloud-stream-overview-deleting-schema-id)”

-  `DELETE /{subject}`   - 见“[the section called “Deleting a Schema by Subject”](multi_schema-evolution.html#spring-cloud-stream-overview-deleting-schema-subject)”

#### 注册新架构

要注册新架构，请将 `POST` 请求发送到 `/` endpoints.

`/` 接受带有以下字段的JSON有效内容：

-  `subject` ：架构主题

-  `format` ：架构格式

-  `definition` ：架构定义

它的响应是JSON中的模式对象，包含以下字段：

-  `id` ：架构ID

-  `subject` ：架构主题

-  `format` ：架构格式

-  `version` ：架构版本

-  `definition` ：架构定义

#### 按主题，格式和版本检索现有架构

要按主题，格式和版本检索现有架构，请将 `GET` 请求发送到 `/{subject}/{format}/{version}` endpoints.

它的响应是JSON中的模式对象，包含以下字段：

-  `id` ：架构ID

-  `subject` ：架构主题

-  `format` ：架构格式

-  `version` ：架构版本

-  `definition` ：架构定义

#### 按主题和格式检索现有架构

要按主题和格式检索现有架构，请将 `GET` 请求发送到 `/subject/format` endpoints.

它的响应是JSON中每个模式对象的模式列表，包含以下字段：

-  `id` ：架构ID

-  `subject` ：架构主题

-  `format` ：架构格式

-  `version` ：架构版本

-  `definition` ：架构定义

#### 按ID检索现有架构

要通过其ID检索架构，请将 `GET` 请求发送到 `/schemas/{id}` endpoints.

它的响应是JSON中的模式对象，包含以下字段：

-  `id` ：架构ID

-  `subject` ：架构主题

-  `format` ：架构格式

-  `version` ：架构版本

-  `definition` ：架构定义

#### 按主题，格式和版本删除架构

要删除由其主题，格式和版本标识的模式，请向 `/{subject}/{format}/{version}` endpoints发送 `DELETE` 请求.

#### 按ID删除架构

要通过其ID删除架构，请将 `DELETE` 请求发送到 `/schemas/{id}` endpoints.

#### 按主题删除架构

`DELETE /{subject}` 

按主题删除现有架构.

> T此说明仅适用于Spring Cloud Stream 1.1.0.RELEASE的用户. Spring Cloud Stream 1.1.0.RELEASE使用表名 `schema` 来存储 `Schema` 对象.  `Schema` 是许多数据库实现中的关键字.为了避免将来出现任何冲突，从1.1.1.RELEASE开始，我们选择了存储表的名称 `SCHEMA_REPOSITORY` .任何升级的Spring Cloud Stream 1.1.0.RELEASE用户都应该在升级之前将其现有架构迁移到新表.

### 31.5.2使用Confluent的架构注册表

默认配置创建 `DefaultSchemaRegistryClient`  bean.如果要使用Confluent模式注册表，则需要创建 `ConfluentSchemaRegistryClient` 类型的bean，该bean将取代框架默认配置的bean.以下示例显示如何创建此类bean：

```java
@Bean
public SchemaRegistryClient schemaRegistryClient(@Value("${spring.cloud.stream.schemaRegistryClient.endpoint}") String endpoint){
ConfluentSchemaRegistryClient client = new ConfluentSchemaRegistryClient();
client.setEndpoint(endpoint);
return client;
}
```

>  ConfluentSchemaRegistryClient针对Confluent平台版本4.0.0进行了测试.

## 31.6架构注册和解决方案

为了更好地了解Spring Cloud Stream如何注册和解析新架构及其对Avro架构比较功能的使用，我们提供了两个单独的小节：

_0009_“[Section 31.6.1, “Schema Registration Process (Serialization)”](multi_schema-evolution.html#spring-cloud-stream-overview-schema-registration-process)”
_0009_“[Section 31.6.2, “Schema Resolution Process (Deserialization)”](multi_schema-evolution.html#spring-cloud-stream-overview-schema-resolution-process)”

### 31.6.1架构注册流程（序列化）

注册过程的第一部分是从通过通道发送的有效负载中提取模式.诸如 `SpecificRecord` 或 `GenericRecord` 之类的Avro类型已经包含一个模式，可以立即从实例中检索该模式.对于POJO，如果 `spring.cloud.stream.schema.avro.dynamicSchemaGenerationEnabled` 属性设置为 `true` （默认值），则会推断出模式.

**Figure 31.1. Schema Writer Resolution Process** 

图片/ schema_resolution.png

获得模式，转换器从远程服务器加载其元数据（版本）.首先，它查询本地缓存.如果未找到任何结果，则会将数据提交给服务器，服务器会回复版本信息.转换器始终缓存结果，以避免为每个需要序列化的新消息查询架构服务器的开销.

**Figure 31.2. Schema Registration Process** 

图片/ registration.png

使用模式版本信息，转换器设置消息的 `contentType` 标头以携带版本信息 - 例如： `application/vnd.user.v1+avro` .

### 31.6.2架构解析过程（反序列化）

当读取包含版本信息的消息（即带有类似“[Section 31.6.1, “Schema Registration Process (Serialization)”](multi_schema-evolution.html#spring-cloud-stream-overview-schema-registration-process)”中描述的方案的 `contentType` 标头）时，转换器会查询Schema服务器以获取消息的writer模式.一旦找到传入消息的正确模式，它就会检索读取器模式，并通过使用Avro的模式解析支持将其读入读取器定义（设置默认值和任何缺少的属性）.

**Figure 31.3. Schema Reading Resolution Process** 

图片/ schema_reading.png

> 您应该了解编写器架构（编写消息的应用程序）和读取器架构（接收应用程序）之间的区别.我们建议花一点时间阅读[the Avro terminology](https://avro.apache.org/docs/1.7.6/spec.html)并理解这个过程. Spring Cloud Stream始终提取writer模式以确定如何阅读消息.如果您希望Avro的架构演进支持正常工作，您需要确保为您的应用程序正确设置了 `readerSchema` .


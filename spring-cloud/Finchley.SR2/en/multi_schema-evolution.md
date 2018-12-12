## 31. Schema Evolution Support

Spring Cloud Stream provides support for schema evolution so that the data can be evolved over time and still work with older or newer producers and consumers and vice versa. Most serialization models, especially the ones that aim for portability across different platforms and languages, rely on a schema that describes how the data is serialized in the binary payload. In order to serialize the data and then to interpret it, both the sending and receiving sides must have access to a schema that describes the binary format. In certain cases, the schema can be inferred from the payload type on serialization or from the target type on deserialization. However, many applications benefit from having access to an explicit schema that describes the binary data format. A schema registry lets you store schema information in a textual format (typically JSON) and makes that information accessible to various applications that need it to receive and send data in binary format. A schema is referenceable as a tuple consisting of:

- A subject that is the logical name of the schema

- The schema version

- The schema format, which describes the binary format of the data

This following sections goes through the details of various components involved in schema evolution process.

## 31.1 Schema Registry Client

The client-side abstraction for interacting with schema registry servers is the  `SchemaRegistryClient`  interface, which has the following structure:

```java
public interface SchemaRegistryClient {

SchemaRegistrationResponse register(String subject, String format, String schema);

String fetch(SchemaReference schemaReference);

String fetch(Integer id);

}
```

Spring Cloud Stream provides out-of-the-box implementations for interacting with its own schema server and for interacting with the Confluent Schema Registry.

A client for the Spring Cloud Stream schema registry can be configured by using the  `@EnableSchemaRegistryClient` , as follows:

```java
@EnableBinding(Sink.class)
@SpringBootApplication
@EnableSchemaRegistryClient
public static class AvroSinkApplication {
...
}
```

> The default converter is optimized to cache not only the schemas from the remote server but also the  `parse()`  and  `toString()`  methods, which are quite expensive. Because of this, it uses a  `DefaultSchemaRegistryClient`  that does not cache responses. If you intend to change the default behavior, you can use the client directly on your code and override it to the desired outcome. To do so, you have to add the property  `spring.cloud.stream.schemaRegistryClient.cached=true`  to your application properties.

### 31.1.1 Schema Registry Client Properties

The Schema Registry Client supports the following properties:

spring.cloud.stream.schemaRegistryClient.endpoint The location of the schema-server. When setting this, use a full URL, including protocol (http or https) , port, and context path. Default http://localhost:8990/ spring.cloud.stream.schemaRegistryClient.cached Whether the client should cache schema server responses. Normally set to false, as the caching happens in the message converter. Clients using the schema registry client should set this to true. Default true

## 31.2 Avro Schema Registry Client Message Converters

For applications that have a SchemaRegistryClient bean registered with the application context, Spring Cloud Stream auto configures an Apache Avro message converter for schema management. This eases schema evolution, as applications that receive messages can get easy access to a writer schema that can be reconciled with their own reader schema.

For outbound messages, if the content type of the channel is set to  `application/*+avro` , the  `MessageConverter`  is activated, as shown in the following example:

```java
spring.cloud.stream.bindings.output.contentType=application/*+avro
```

During the outbound conversion, the message converter tries to infer the schema of each outbound messages (based on its type) and register it to a subject (based on the payload type) by using the  `SchemaRegistryClient` . If an identical schema is already found, then a reference to it is retrieved. If not, the schema is registered, and a new version number is provided. The message is sent with a  `contentType`  header by using the following scheme:  `application/[prefix].[subject].v[version]+avro` , where  `prefix`  is configurable and  `subject`  is deduced from the payload type.

For example, a message of the type  `User`  might be sent as a binary payload with a content type of  `application/vnd.user.v2+avro` , where  `user`  is the subject and  `2`  is the version number.

When receiving messages, the converter infers the schema reference from the header of the incoming message and tries to retrieve it. The schema is used as the writer schema in the deserialization process.

### 31.2.1 Avro Schema Registry Message Converter Properties

If you have enabled Avro based schema registry client by setting  `spring.cloud.stream.bindings.output.contentType=application/*+avro` , you can customize the behavior of the registration by setting the following properties.

spring.cloud.stream.schema.avro.dynamicSchemaGenerationEnabled Enable if you want the converter to use reflection to infer a Schema from a POJO. Default: false spring.cloud.stream.schema.avro.readerSchema Avro compares schema versions by looking at a writer schema (origin payload) and a reader schema (your application payload). See the Avro documentation for more information. If set, this overrides any lookups at the schema server and uses the local schema as the reader schema. Default: null spring.cloud.stream.schema.avro.schemaLocations Registers any .avsc files listed in this property with the Schema Server. Default: empty spring.cloud.stream.schema.avro.prefix The prefix to be used on the Content-Type header. Default: vnd

## 31.3 Apache Avro Message Converters

Spring Cloud Stream provides support for schema-based message converters through its  `spring-cloud-stream-schema`  module. Currently, the only serialization format supported out of the box for schema-based message converters is Apache Avro, with more formats to be added in future versions.

The  `spring-cloud-stream-schema`  module contains two types of message converters that can be used for Apache Avro serialization:

- Converters that use the class information of the serialized or deserialized objects or a schema with a location known at startup.

- Converters that use a schema registry. They locate the schemas at runtime and dynamically register new schemas as domain objects evolve.

## 31.4 Converters with Schema Support

The  `AvroSchemaMessageConverter`  supports serializing and deserializing messages either by using a predefined schema or by using the schema information available in the class (either reflectively or contained in the  `SpecificRecord` ). If you provide a custom converter, then the default AvroSchemaMessageConverter bean is not created. The following example shows a custom converter:

To use custom converters, you can simply add it to the application context, optionally specifying one or more  `MimeTypes`  with which to associate it. The default  `MimeType`  is  `application/avro` .

If the target type of the conversion is a  `GenericRecord` , a schema must be set.

The following example shows how to configure a converter in a sink application by registering the Apache Avro  `MessageConverter`  without a predefined schema. In this example, note that the mime type value is  `avro/bytes` , not the default  `application/avro` .

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

Conversely, the following application registers a converter with a predefined schema (found on the classpath):

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

## 31.5 Schema Registry Server

Spring Cloud Stream provides a schema registry server implementation. To use it, you can add the  `spring-cloud-stream-schema-server`  artifact to your project and use the  `@EnableSchemaRegistryServer`  annotation, which adds the schema registry server REST controller to your application. This annotation is intended to be used with Spring Boot web applications, and the listening port of the server is controlled by the  `server.port`  property. The  `spring.cloud.stream.schema.server.path`  property can be used to control the root path of the schema server (especially when it is embedded in other applications). The  `spring.cloud.stream.schema.server.allowSchemaDeletion`  boolean property enables the deletion of a schema. By default, this is disabled.

The schema registry server uses a relational database to store the schemas. By default, it uses an embedded database. You can customize the schema storage by using the [Spring Boot SQL database and JDBC configuration options](https://docs.spring.io/spring-boot/docs/current-SNAPSHOT/reference/htmlsingle/#boot-features-sql).

The following example shows a Spring Boot application that enables the schema registry:

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

The Schema Registry Server API consists of the following operations:

-  `POST /`  — see “[the section called “Registering a New Schema”](multi_schema-evolution.html#spring-cloud-stream-overview-registering-new-schema)”

- 'GET /{subject}/{format}/{version}' — see “[the section called “Retrieving an Existing Schema by Subject, Format, and Version”](multi_schema-evolution.html#spring-cloud-stream-overview-retrieve-schema-subject-format-version)”

-  `GET /{subject}/{format}`  — see “[the section called “Retrieving an Existing Schema by Subject and Format”](multi_schema-evolution.html#spring-cloud-stream-overview-retrieve-schema-subject-format)”

-  `GET /schemas/{id}`  — see “[the section called “Retrieving an Existing Schema by ID”](multi_schema-evolution.html#spring-cloud-stream-overview-retrieve-schema-id)”

-  `DELETE /{subject}/{format}/{version}`  — see “[the section called “Deleting a Schema by Subject, Format, and Version”](multi_schema-evolution.html#spring-cloud-stream-overview-deleting-schema-subject-format-version)”

-  `DELETE /schemas/{id}`  — see “[the section called “Deleting a Schema by ID”](multi_schema-evolution.html#spring-cloud-stream-overview-deleting-schema-id)”

-  `DELETE /{subject}`  — see “[the section called “Deleting a Schema by Subject”](multi_schema-evolution.html#spring-cloud-stream-overview-deleting-schema-subject)”

#### Registering a New Schema

To register a new schema, send a  `POST`  request to the  `/`  endpoint.

The  `/`  accepts a JSON payload with the following fields:

-  `subject` : The schema subject

-  `format` : The schema format

-  `definition` : The schema definition

Its response is a schema object in JSON, with the following fields:

-  `id` : The schema ID

-  `subject` : The schema subject

-  `format` : The schema format

-  `version` : The schema version

-  `definition` : The schema definition

#### Retrieving an Existing Schema by Subject, Format, and Version

To retrieve an existing schema by subject, format, and version, send  `GET`  request to the  `/{subject}/{format}/{version}`  endpoint.

Its response is a schema object in JSON, with the following fields:

-  `id` : The schema ID

-  `subject` : The schema subject

-  `format` : The schema format

-  `version` : The schema version

-  `definition` : The schema definition

#### Retrieving an Existing Schema by Subject and Format

To retrieve an existing schema by subject and format, send a  `GET`  request to the  `/subject/format`  endpoint.

Its response is a list of schemas with each schema object in JSON, with the following fields:

-  `id` : The schema ID

-  `subject` : The schema subject

-  `format` : The schema format

-  `version` : The schema version

-  `definition` : The schema definition

#### Retrieving an Existing Schema by ID

To retrieve a schema by its ID, send a  `GET`  request to the  `/schemas/{id}`  endpoint.

Its response is a schema object in JSON, with the following fields:

-  `id` : The schema ID

-  `subject` : The schema subject

-  `format` : The schema format

-  `version` : The schema version

-  `definition` : The schema definition

#### Deleting a Schema by Subject, Format, and Version

To delete a schema identified by its subject, format, and version, send a  `DELETE`  request to the  `/{subject}/{format}/{version}`  endpoint.

#### Deleting a Schema by ID

To delete a schema by its ID, send a  `DELETE`  request to the  `/schemas/{id}`  endpoint.

#### Deleting a Schema by Subject

`DELETE /{subject}` 

Delete existing schemas by their subject.

> This note applies to users of Spring Cloud Stream 1.1.0.RELEASE only. Spring Cloud Stream 1.1.0.RELEASE used the table name,  `schema` , for storing  `Schema`  objects.  `Schema`  is a keyword in a number of database implementations. To avoid any conflicts in the future, starting with 1.1.1.RELEASE, we have opted for the name  `SCHEMA_REPOSITORY`  for the storage table. Any Spring Cloud Stream 1.1.0.RELEASE users who upgrade should migrate their existing schemas to the new table before upgrading.

### 31.5.2 Using Confluent’s Schema Registry

The default configuration creates a  `DefaultSchemaRegistryClient`  bean. If you want to use the Confluent schema registry, you need to create a bean of type  `ConfluentSchemaRegistryClient` , which supersedes the one configured by default by the framework. The following example shows how to create such a bean:

```java
@Bean
public SchemaRegistryClient schemaRegistryClient(@Value("${spring.cloud.stream.schemaRegistryClient.endpoint}") String endpoint){
ConfluentSchemaRegistryClient client = new ConfluentSchemaRegistryClient();
client.setEndpoint(endpoint);
return client;
}
```

> The ConfluentSchemaRegistryClient is tested against Confluent platform version 4.0.0.

## 31.6 Schema Registration and Resolution

To better understand how Spring Cloud Stream registers and resolves new schemas and its use of Avro schema comparison features, we provide two separate subsections:

- “[Section 31.6.1, “Schema Registration Process (Serialization)”](multi_schema-evolution.html#spring-cloud-stream-overview-schema-registration-process)”

- “[Section 31.6.2, “Schema Resolution Process (Deserialization)”](multi_schema-evolution.html#spring-cloud-stream-overview-schema-resolution-process)”

### 31.6.1 Schema Registration Process (Serialization)

The first part of the registration process is extracting a schema from the payload that is being sent over a channel. Avro types such as  `SpecificRecord`  or  `GenericRecord`  already contain a schema, which can be retrieved immediately from the instance. In the case of POJOs, a schema is inferred if the  `spring.cloud.stream.schema.avro.dynamicSchemaGenerationEnabled`  property is set to  `true`  (the default).

**Figure 31.1. Schema Writer Resolution Process** 

images/schema_resolution.png

Ones a schema is obtained, the converter loads its metadata (version) from the remote server. First, it queries a local cache. If no result is found, it submits the data to the server, which replies with versioning information. The converter always caches the results to avoid the overhead of querying the Schema Server for every new message that needs to be serialized.

**Figure 31.2. Schema Registration Process** 

images/registration.png

With the schema version information, the converter sets the  `contentType`  header of the message to carry the version information — for example:  `application/vnd.user.v1+avro` .

### 31.6.2 Schema Resolution Process (Deserialization)

When reading messages that contain version information (that is, a  `contentType`  header with a scheme like the one described under “[Section 31.6.1, “Schema Registration Process (Serialization)”](multi_schema-evolution.html#spring-cloud-stream-overview-schema-registration-process)”), the converter queries the Schema server to fetch the writer schema of the message. Once it has found the correct schema of the incoming message, it retrieves the reader schema and, by using Avro’s schema resolution support, reads it into the reader definition (setting defaults and any missing properties).

**Figure 31.3. Schema Reading Resolution Process** 

images/schema_reading.png

> You should understand the difference between a writer schema (the application that wrote the message) and a reader schema (the receiving application). We suggest taking a moment to read [the Avro terminology](https://avro.apache.org/docs/1.7.6/spec.html) and understand the process. Spring Cloud Stream always fetches the writer schema to determine how to read a message. If you want to get Avro’s schema evolution support working, you need to make sure that a  `readerSchema`  was properly set for your application.


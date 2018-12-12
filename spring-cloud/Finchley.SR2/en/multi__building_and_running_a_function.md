## 125. Building and Running a Function

The sample  `@SpringBootApplication`  above has a function that can be decorated at runtime by Spring Cloud Function to be an HTTP endpoint, or a Stream processor, for instance with RabbitMQ, Apache Kafka or JMS.

The  `@Beans`  can be  `Function` ,  `Consumer`  or  `Supplier`  (all from  `java.util` ), and their parametric types can be String or POJO. A  `Function`  is exposed as a Spring Cloud Stream  `Processor`  if  `spring-cloud-function-stream`  is on the classpath. A  `Consumer`  is also exposed as a Stream  `Sink`  and a  `Supplier`  translates to a Stream  `Source` . HTTP endpoints are exposed if the Stream binder is  `spring-cloud-stream-binder-servlet` .

Functions can be of  `Flux<String>`  or  `Flux<Pojo>`  and Spring Cloud Function takes care of converting the data to and from the desired types, as long as it comes in as plain text or (in the case of the POJO) JSON. TBD: support for  `Flux<Message<Pojo>>`  and maybe plain  `Pojo`  types (Fluxes implied and implemented by the framework).

Functions can be grouped together in a single application, or deployed one-per-jar. It’s up to the developer to choose. An app with multiple functions can be deployed multiple times in different "personalities", exposing different functions over different physical transports.

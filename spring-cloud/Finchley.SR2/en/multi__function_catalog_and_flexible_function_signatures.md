## 126. Function Catalog and Flexible Function Signatures

One of the main features of Spring Cloud Function is to adapt and support a range of type signatures for user-defined functions. So users can supply a bean of type  `Function<String,String>` , for instance, and the  `FunctionCatalog`  will wrap it into a  `Function<Flux<String>,Flux<String>>` . Users don’t normally have to care about the  `FunctionCatalog`  at all, but it is useful to know what kind of functions are supported in user code.

Generally speaking users can expect that if they write a function for a plain old Java type (or primitive wrapper), then the function catalog will wrap it to a  `Flux`  of the same type. If the user writes a function using  `Message`  (from spring-messaging) it will receive and transmit headers from any adapter that supports key-value metadata (e.g. HTTP headers). Here are the details.

|User Function|Catalog Registration| |
|----|----|----|
| `Function<S,T>`  | `Function<Flux<S>, Flux<T>>`  | |
| `Function<Message<S>,Message<T>>`  | `Function<Flux<Message<S>>, Flux<Message<T>>>`  | |
| `Function<Flux<S>, Flux<T>>`  | `Function<Flux<S>, Flux<T>>`  (pass through) | |
| `Supplier<T>`  | `Supplier<Flux<T>>`  | |
| `Supplier<Flux<T>>`  | `Supplier<Flux<T>>`  | |
| `Consumer<T>`  | `Function<Flux<T>, Mono<Void>>`  | |
| `Consumer<Message<T>>`  | `Function<Flux<Message<T>>, Mono<Void>>`  | |
| `Consumer<Flux<T>>`  | `Consumer<Flux<T>>`  | |

Consumer is a little bit special because it has a  `void`  return type, which implies blocking, at least potentially. Most likely you will not need to write  `Consumer<Flux<?>>` , but if you do need to do that, remember to subscribe to the input flux. If you declare a  `Consumer`  of a non publisher type (which is normal), it will be converted to a function that returns a publisher, so that it can be subscribed to in a controlled way.

A function catalog can contain a  `Supplier`  and a  `Function`  (or  `Consumer` ) with the same name (like a GET and a POST to the same resource). It can even contain a  `Consumer<Flux<>>`  with the same name as a  `Function` , but it cannot contain a  `Consumer<T>`  and a  `Function<T,S>`  with the same name when  `T`  is not a  `Publisher`  because the consumer would be converted to a  `Function`  and only one of them can be registered.

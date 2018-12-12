## 126.功能目录和灵活功能签名

Spring Cloud Function的主要功能之一是为用户定义的函数调整和支持一系列类型签名.因此，用户可以提供 `Function<String,String>` 类型的bean， `FunctionCatalog` 将它包装成 `Function<Flux<String>,Flux<String>>` .用户通常不必关心 `FunctionCatalog` ，但了解用户代码中支持哪种功能很有用.

一般来说，如果用户为普通的旧Java类型（或原始包装器）编写函数，那么函数目录会将其包装为相同类型的 `Flux` .如果用户使用 `Message` （来自spring-messaging）编写函数，它将从任何支持键值元数据的适配器（例如HTTP头）接收和传输标头.这是细节.

|用户功能|目录注册| |
| ---- | ---- | ---- |
|  `Function<S,T>`  |  `Function<Flux<S>, Flux<T>>`  | |
|  `Function<Message<S>,Message<T>>`  |  `Function<Flux<Message<S>>, Flux<Message<T>>>`  | |
|  `Function<Flux<S>, Flux<T>>`  |  `Function<Flux<S>, Flux<T>>` （通过）| |
|  `Supplier<T>`  |  `Supplier<Flux<T>>`  | |
|  `Supplier<Flux<T>>`  |  `Supplier<Flux<T>>`  | |
|  `Consumer<T>`  |  `Function<Flux<T>, Mono<Void>>`  | |
|  `Consumer<Message<T>>`  |  `Function<Flux<Message<T>>, Mono<Void>>`  | |
|  `Consumer<Flux<T>>`  |  `Consumer<Flux<T>>`  | |

消费者有点特殊，因为它有一个 `void` 返回类型，这意味着阻塞，至少可能.很可能你不需要写 `Consumer<Flux<?>>` ，但如果你确实需要这样做，请记住订阅输入通量.如果声明非发布者类型的 `Consumer` （这是正常的），它将转换为返回发布者的函数，以便可以以受控方式订阅它.

函数目录可以包含具有相同名称的 `Supplier` 和 `Function` （或 `Consumer` ）（如同一资源的GET和POST）.它甚至可以包含与 `Function` 同名的 `Consumer<Flux<>>` ，但是当 `T` 不是 `Publisher` 时，它不能包含具有相同名称的 `Consumer<T>` 和 `Function<T,S>` ，因为消费者将被转换为 `Function` 并且只能注册其中一个.

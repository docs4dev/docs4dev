## 127.独立Web应用程序

`spring-cloud-function-web` 模块具有自动配置，当它包含在Spring Boot Web应用程序（具有MVC支持）时会激活.还有一个 `spring-cloud-starter-function-web` 可以收集所有可选的依赖项，以防您只需要简单的入门体验.

激活Web配置后，您的应用程序将具有MVCendpoints（默认为"/"，但可配置为 `spring.cloud.function.web.path` ），可用于访问应用程序上下文中的功能.支持的内容类型是纯文本和JSON.

|方法|路径|请求|响应|状态|
| ---- | ---- | ---- | ---- | ---- |
| GET | / {supplier} |  -  |来自指定供应商的项目| 200 OK |
| POST | / {consumer} | JSON对象或文本|镜像输入并将请求主体推送到消费者中| 202 Accepted |
| POST | / {consumer} |带有新行的JSON数组或文本|镜像输入并逐个将主体推送到消费者中| 202接受|
| POST | / {function} | JSON对象或文本|应用命名函数的结果| 200 OK |
| POST | / {function} |带有新行的JSON数组或文本|应用指定函数的结果| 200 OK |
| GET | / {function} / {item} |  -  |将项目转换为对象并返回应用函数的结果| 200 OK |

如上表所示，endpoints的行为取决于方法以及传入请求数据的类型.当传入数据是单值的，并且目标函数被声明为明显单值（即不返回集合或 `Flux` ）时，响应也将包含单个值.对于多值响应，客户端可以通过发送“Accept：text / event-stream”来请求服务器发送的事件流.如果只有一个函数（使用者等），则路径中的名称是可选的.复合函数可以使用管道或逗号来分隔函数名称（管道在URL路径中是合法的，但在命令行上输入有点尴尬）.

在 `Message<?>` 中使用输入和输出声明的函数和使用者将在输入消息上看到请求标头，并且输出消息标头将转换为HTTP标头.

POST文本时，响应格式可能与Spring Boot 2.0和旧版本不同，具体取决于内容协商（提供内容类型和accpt标头以获得最佳结果）.

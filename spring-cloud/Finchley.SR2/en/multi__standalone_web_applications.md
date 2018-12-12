## 127. Standalone Web Applications

The  `spring-cloud-function-web`  module has autoconfiguration that activates when it is included in a Spring Boot web application (with MVC support). There is also a  `spring-cloud-starter-function-web`  to collect all the optional dependnecies in case you just want a simple getting started experience.

With the web configurations activated your app will have an MVC endpoint (on "/" by default, but configurable with  `spring.cloud.function.web.path` ) that can be used to access the functions in the application context. The supported content types are plain text and JSON.

|Method|Path|Request|Response|Status|
|----|----|----|----|----|
|GET |/{supplier} |- |Items from the named supplier |200 OK |
|POST |/{consumer} |JSON object or text |Mirrors input and pushes request body into consumer |202 Accepted |
|POST |/{consumer} |JSON array or text with new lines |Mirrors input and pushes body into consumer one by one |202 Accepted |
|POST |/{function} |JSON object or text |The result of applying the named function |200 OK |
|POST |/{function} |JSON array or text with new lines |The result of applying the named function |200 OK |
|GET |/{function}/{item} |- |Convert the item into an object and return the result of applying the function |200 OK |

As the table above shows the behaviour of the endpoint depends on the method and also the type of incoming request data. When the incoming data is single valued, and the target function is declared as obviously single valued (i.e. not returning a collection or  `Flux` ), then the response will also contain a single value. For multi-valued responses the client can ask for a server-sent event stream by sending `Accept: text/event-stream". If there is only one function (consumer etc.) then the name in the path is optional. Composite functions can be addressed using pipes or commas to separate function names (pipes are legal in URL paths, but a bit awkward to type on the command line).

Functions and consumers that are declared with input and output in  `Message<?>`  will see the request headers on the input messages, and the output message headers will be converted to HTTP headers.

When POSTing text the response format might be different with Spring Boot 2.0 and older versions, depending on the content negotiation (provide content type and accpt headers for the best results).

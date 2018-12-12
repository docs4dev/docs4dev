## 21. HTTP客户端

Spring Cloud Netflix会自动为您创建Ribbon，Feign和Zuul使用的HTTP客户端.但是，您也可以根据需要自定义自己的HTTP客户端.为此，如果使用的是Apache Http Cient，则可以创建 `ClosableHttpClient` 类型的bean，如果使用的是OK，则可以创建 `OkHttpClient` .

> 当您创建自己的HTTP客户端时，您还负责为这些客户端实施正确的连接管理策略.不正确地这样做会导致资源管理问题.


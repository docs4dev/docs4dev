## 113.工作原理

![Spring Cloud Gateway Diagram](https://www.docs4dev.com/images/86675a8e-26e0-4336-96a7-22dc7202d4b6.png)

客户端向Spring Cloud Gateway发出请求.如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序.此处理程序运行通过特定于请求的过滤器链发送请求.滤波器被虚线划分的原因是滤波器可以在发送代理请求之前或之后执行逻辑.执行所有“预”过滤器逻辑，然后进行代理请求.在发出代理请求之后，执行“post”过滤器逻辑.

在没有端口的路由中定义的
> URI将分别为HTTP和HTTPS URI获取默认端口设置为80和443.


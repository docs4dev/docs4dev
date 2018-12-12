## 113. How It Works

![Spring Cloud Gateway Diagram](https://www.docs4dev.com/images/86675a8e-26e0-4336-96a7-22dc7202d4b6.png)

Clients make requests to Spring Cloud Gateway. If the Gateway Handler Mapping determines that a request matches a Route, it is sent to the Gateway Web Handler. This handler runs sends the request through a filter chain that is specific to the request. The reason the filters are divided by the dotted line, is that filters may execute logic before the proxy request is sent or after. All "pre" filter logic is executed, then the proxy request is made. After the proxy request is made, the "post" filter logic is executed.

> URIs defined in routes without a port will get a default port set to 80 and 443 for HTTP and HTTPS URIs respectively.


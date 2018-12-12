## 65.Consul代理

所有Spring Cloud Consul应用程序都必须可以使用Consul Agent客户端.默认情况下，代理客户端应为 `localhost:8500` .有关如何启动代理客户端以及如何连接到Consul Agent Server群集的详细信息，请参阅[Agent documentation](https://consul.io/docs/agent/basics.html).对于开发，在安装了consul之后，您可以使用以下命令启动Consul Agent：

```java
./src/main/bash/local_run_consul.sh
```

这将在端口8500上以服务器模式启动代理，在[http://localhost:8500](http://localhost:8500)处可以使用ui

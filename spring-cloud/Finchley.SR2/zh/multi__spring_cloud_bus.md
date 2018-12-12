# Part VII.Spring CloudBus

Spring Cloud Bus使用轻量级消息代理链接分布式系统的节点.然后，此代理可用于广播状态更改（例如配置更改）或其他管理指令.一个关键的想法是总线就像是一个扩展的Spring Boot应用程序的分布式Actuator.但是，它也可以用作应用程序之间的通信通道.该项目为AMQP经纪人或Kafka提供启动作为运输.

> Spring Cloud是在非限制性Apache 2.0许可下发布的.如果您想参与本文档的这一部分，或者如果发现错误，请在[github](https://github.com/spring-cloud/spring-cloud-config/tree/master/docs/src/main/asciidoc)找到项目中的源代码和问题跟踪器.


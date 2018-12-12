## 86.消息

Spring Boot提供了许多包含消息传递的启动器.本节回答使用Spring Boot消息传递产生的问题.

## 86.1禁用已处理的JMS会话

如果您的JMS代理不支持事务会话，则必须完全禁用事务支持.如果您创建自己的 `JmsListenerContainerFactory` ，则无需执行任何操作，因为默认情况下无法进行事务处理.如果要使用 `DefaultJmsListenerContainerFactoryConfigurer` 重用Spring Boot的默认值，可以禁用事务处理会话，如下所示：

```java
@Bean
public DefaultJmsListenerContainerFactory jmsListenerContainerFactory(
		ConnectionFactory connectionFactory,
		DefaultJmsListenerContainerFactoryConfigurer configurer) {
	DefaultJmsListenerContainerFactory listenerFactory =
			new DefaultJmsListenerContainerFactory();
	configurer.configure(listenerFactory, connectionFactory);
	listenerFactory.setTransactionManager(null);
	listenerFactory.setSessionTransacted(false);
	return listenerFactory;
}
```

上面的示例将覆盖默认工厂，并且应该应用于应用程序定义的任何其他工厂（如果有）.


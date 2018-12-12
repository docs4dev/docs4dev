## 17.外部配置：Archaius

[Archaius](https://github.com/Netflix/archaius)是Netflix客户端配置库.它是所有Netflix OSS组件用于配置的库. Archaius是[Apache Commons Configuration](https://commons.apache.org/proper/commons-configuration)项目的扩展.它允许通过轮询源更改或允许源推送更改到客户端来更新配置. Archaius使用Dynamic <Type> Property类作为属性的句柄，如以下示例所示：

**Archaius Example.** 

```java
class ArchaiusTest {
DynamicStringProperty myprop = DynamicPropertyFactory
.getInstance()
.getStringProperty("my.prop");

void doSomething() {
OtherClass.someMethod(myprop.get());
}
}
```

Archaius有自己的一组配置文件和加载优先级. Spring应用程序通常不应直接使用Archaius，但仍然需要本机配置Netflix工具. Spring Cloud有一个Spring Environment Bridge，因此Archaius可以从Spring Environment中读取属性.此桥接器允许Spring Boot项目使用常规配置工具链，同时让他们按照文档（大多数情况下）配置Netflix工具.

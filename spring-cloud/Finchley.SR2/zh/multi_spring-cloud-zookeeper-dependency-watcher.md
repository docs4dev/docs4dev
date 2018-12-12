## 77. Spring Cloud Zookeeper依赖观察者

Dependency Watcher机制允许您向依赖项注册侦听器.事实上，该功能是 `Observator` 模式的实现.当依赖项发生更改时，其状态（向上或向下）可以应用某些自定义逻辑.

## 77.1激活

需要启用Spring Cloud Zookeeper依赖项功能才能使用Dependency Watcher机制.

## 77.2注册监听器

要注册侦听器，必须实现名为 `org.springframework.cloud.zookeeper.discovery.watcher.DependencyWatcherListener` 的接口并将其注册为bean.界面为您提供了一种方法：

```java
void stateChanged(String dependencyName, DependencyState newState);
```

如果要为特定依赖项注册侦听器， `dependencyName` 将是具体实现的鉴别器.  `newState` 为您提供有关您的依赖项是否已更改为 `CONNECTED` 或 `DISCONNECTED` 的信息.

## 77.3使用Presence Checker

与Dependency Watcher绑定的是名为Presence Checker的功能.它允许您在应用程序引导时提供自定义行为，以根据依赖项的状态做出反应.

abstract  `org.springframework.cloud.zookeeper.discovery.watcher.presence.DependencyPresenceOnStartupVerifier` 类的默认实现是 `org.springframework.cloud.zookeeper.discovery.watcher.presence.DefaultDependencyPresenceOnStartupVerifier` ，它按以下方式工作.

如果依赖项标记为我们需要且不在Zookeeper中，则在应用程序启动时，它会抛出异常并关闭.如果不需要依赖关系，则org.springframework.cloud.zookeeper.discovery.watcher.presence.LogMissingDependencyChecker会记录WARN级别缺少依赖关系.

因为 `DefaultDependencyPresenceOnStartupVerifier` 仅在没有 `DependencyPresenceOnStartupVerifier` 类型的bean时注册，所以可以覆盖此功能.


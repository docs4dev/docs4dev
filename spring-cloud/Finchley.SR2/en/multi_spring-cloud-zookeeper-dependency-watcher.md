## 77. Spring Cloud Zookeeper Dependency Watcher

The Dependency Watcher mechanism lets you register listeners to your dependencies. The functionality is, in fact, an implementation of the  `Observator`  pattern. When a dependency changes, its state (to either UP or DOWN), some custom logic can be applied.

## 77.1 Activating

Spring Cloud Zookeeper Dependencies functionality needs to be enabled for you to use the Dependency Watcher mechanism.

## 77.2 Registering a Listener

To register a listener, you must implement an interface called  `org.springframework.cloud.zookeeper.discovery.watcher.DependencyWatcherListener`  and register it as a bean. The interface gives you one method:

```java
void stateChanged(String dependencyName, DependencyState newState);
```

If you want to register a listener for a particular dependency, the  `dependencyName`  would be the discriminator for your concrete implementation.  `newState`  provides you with information about whether your dependency has changed to  `CONNECTED`  or  `DISCONNECTED` .

## 77.3 Using the Presence Checker

Bound with the Dependency Watcher is the functionality called Presence Checker. It lets you provide custom behavior when your application boots, to react according to the state of your dependencies.

The default implementation of the abstract  `org.springframework.cloud.zookeeper.discovery.watcher.presence.DependencyPresenceOnStartupVerifier`  class is the  `org.springframework.cloud.zookeeper.discovery.watcher.presence.DefaultDependencyPresenceOnStartupVerifier` , which works in the following way.

If the dependency is marked us required and is not in Zookeeper, when your application boots, it throws an exception and shuts down. If the dependency is not required, the org.springframework.cloud.zookeeper.discovery.watcher.presence.LogMissingDependencyChecker logs that the dependency is missing at the WARN level.

Because the  `DefaultDependencyPresenceOnStartupVerifier`  is registered only when there is no bean of type  `DependencyPresenceOnStartupVerifier` , this functionality can be overridden.


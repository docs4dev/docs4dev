## 47. Broadcasting Your Own Events

The Bus can carry any event of type  `RemoteApplicationEvent` . The default transport is JSON, and the deserializer needs to know which types are going to be used ahead of time. To register a new type, you must put it in a subpackage of  `org.springframework.cloud.bus.event` .

To customise the event name, you can use  `@JsonTypeName`  on your custom class or rely on the default strategy, which is to use the simple name of the class.

> Both the producer and the consumer need access to the class definition.

## 47.1 Registering events in custom packages

If you cannot or do not want to use a subpackage of  `org.springframework.cloud.bus.event`  for your custom events, you must specify which packages to scan for events of type  `RemoteApplicationEvent`  by using the  `@RemoteApplicationEventScan`  annotation. Packages specified with  `@RemoteApplicationEventScan`  include subpackages.

For example, consider the following custom event, called  `MyEvent` :

```java
package com.acme;

public class MyEvent extends RemoteApplicationEvent {
...
}
```

You can register that event with the deserializer in the following way:

```java
package com.acme;

@Configuration
@RemoteApplicationEventScan
public class BusConfiguration {
...
}
```

Without specifying a value, the package of the class where  `@RemoteApplicationEventScan`  is used is registered. In this example,  `com.acme`  is registered by using the package of  `BusConfiguration` .

You can also explicitly specify the packages to scan by using the  `value` ,  `basePackages`  or  `basePackageClasses`  properties on  `@RemoteApplicationEventScan` , as shown in the following example:

```java
package com.acme;

@Configuration
//@RemoteApplicationEventScan({"com.acme", "foo.bar"})
//@RemoteApplicationEventScan(basePackages = {"com.acme", "foo.bar", "fizz.buzz"})
@RemoteApplicationEventScan(basePackageClasses = BusConfiguration.class)
public class BusConfiguration {
...
}
```

All of the preceding examples of  `@RemoteApplicationEventScan`  are equivalent, in that the  `com.acme`  package is registered by explicitly specifying the packages on  `@RemoteApplicationEventScan` .

> You can specify multiple base packages to scan.


## 67. Distributed Configuration with Consul

Consul provides a [Key/Value Store](https://consul.io/docs/agent/http/kv.html) for storing configuration and other metadata. Spring Cloud Consul Config is an alternative to the [Config Server and Client](https://github.com/spring-cloud/spring-cloud-config). Configuration is loaded into the Spring Environment during the special "bootstrap" phase. Configuration is stored in the  `/config`  folder by default. Multiple  `PropertySource`  instances are created based on the application’s name and the active profiles that mimicks the Spring Cloud Config order of resolving properties. For example, an application with the name "testApp" and with the "dev" profile will have the following property sources created:

```java
config/testApp,dev/
config/testApp/
config/application,dev/
config/application/
```

The most specific property source is at the top, with the least specific at the bottom. Properties in the  `config/application`  folder are applicable to all applications using consul for configuration. Properties in the  `config/testApp`  folder are only available to the instances of the service named "testApp".

Configuration is currently read on startup of the application. Sending a HTTP POST to  `/refresh`  will cause the configuration to be reloaded. [Section 67.3, “Config Watch”](multi_spring-cloud-consul-config.html#spring-cloud-consul-config-watch) will also automatically detect changes and reload the application context.

## 67.1 How to activate

To get started with Consul Configuration use the starter with group  `org.springframework.cloud`  and artifact id  `spring-cloud-starter-consul-config` . See the [Spring Cloud Project page](https://projects.spring.io/spring-cloud/) for details on setting up your build system with the current Spring Cloud Release Train.

This will enable auto-configuration that will setup Spring Cloud Consul Config.

## 67.2 Customizing

Consul Config may be customized using the following properties:

**bootstrap.yml.**  

```java
spring:
cloud:
consul:
config:
enabled: true
prefix: configuration
defaultContext: apps
profileSeparator: '::'
```

-  `enabled`  setting this value to "false" disables Consul Config

-  `prefix`  sets the base folder for configuration values

-  `defaultContext`  sets the folder name used by all applications

-  `profileSeparator`  sets the value of the separator used to separate the profile name in property sources with profiles

## 67.3 Config Watch

The Consul Config Watch takes advantage of the ability of consul to [watch a key prefix](https://www.consul.io/docs/agent/watches.html#keyprefix). The Config Watch makes a blocking Consul HTTP API call to determine if any relevant configuration data has changed for the current application. If there is new configuration data a Refresh Event is published. This is equivalent to calling the  `/refresh`  actuator endpoint.

To change the frequency of when the Config Watch is called change  `spring.cloud.consul.config.watch.delay` . The default value is 1000, which is in milliseconds. The delay is the amount of time after the end of the previous invocation and the start of the next.

To disable the Config Watch set  `spring.cloud.consul.config.watch.enabled=false` .

The watch uses a Spring  `TaskScheduler`  to schedule the call to consul. By default it is a  `ThreadPoolTaskScheduler`  with a  `poolSize`  of 1. To change the  `TaskScheduler` , create a bean of type  `TaskScheduler`  named with the  `ConsulConfigAutoConfiguration.CONFIG_WATCH_TASK_SCHEDULER_NAME`  constant.

## 67.4 YAML or Properties with Config

It may be more convenient to store a blob of properties in YAML or Properties format as opposed to individual key/value pairs. Set the  `spring.cloud.consul.config.format`  property to  `YAML`  or  `PROPERTIES` . For example to use YAML:

**bootstrap.yml.**  

```java
spring:
cloud:
consul:
config:
format: YAML
```

YAML must be set in the appropriate  `data`  key in consul. Using the defaults above the keys would look like:

```java
config/testApp,dev/data
config/testApp/data
config/application,dev/data
config/application/data
```

You could store a YAML document in any of the keys listed above.

You can change the data key using  `spring.cloud.consul.config.data-key` .

## 67.5 git2consul with Config

git2consul is a Consul community project that loads files from a git repository to individual keys into Consul. By default the names of the keys are names of the files. YAML and Properties files are supported with file extensions of  `.yml`  and  `.properties`  respectively. Set the  `spring.cloud.consul.config.format`  property to  `FILES` . For example:

**bootstrap.yml.**  

```java
spring:
cloud:
consul:
config:
format: FILES
```

Given the following keys in  `/config` , the  `development`  profile and an application name of  `foo` :

```java
.gitignore
application.yml
bar.properties
foo-development.properties
foo-production.yml
foo.properties
master.ref
```

the following property sources would be created:

```java
config/foo-development.properties
config/foo.properties
config/application.yml
```

The value of each key needs to be a properly formatted YAML or Properties file.

## 67.6 Fail Fast

It may be convenient in certain circumstances (like local development or certain test scenarios) to not fail if consul isn’t available for configuration. Setting  `spring.cloud.consul.config.failFast=false`  in  `bootstrap.yml`  will cause the configuration module to log a warning rather than throw an exception. This will allow the application to continue startup normally.


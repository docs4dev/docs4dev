## 72.安装Zookeeper

有关如何安装Zookeeper的说明，请参阅[installation documentation](https://zookeeper.apache.org/doc/current/zookeeperStarted.html).

Spring Cloud Zookeeper在幕后使用Apache Curator.虽然Zookeeper 3.5.x仍然被Zookeeper开发团队视为“测试版”，但实际情况是它被许多用户用于生产环境.但是，Zookeeper 3.4.x也用于生产环境.在Apache Curator 4.0之前，两个版本的Apache Curator都支持两个版本的Zookeeper.从Curator 4.0开始，两个版本的Zookeeper都通过相同的Curator库支持.

如果要与3.4版集成，则需要更改 `curator` 附带的Zookeeper依赖项，从而更改 `spring-cloud-zookeeper` .为此，只需排除该依赖项并添加3.4.x版本，如下所示.

**maven.** 

```xml
<dependency>
<groupId>org.springframework.cloud</groupId>
<artifactId>spring-cloud-starter-zookeeper-all</artifactId>
<exclusions>
<exclusion>
<groupId>org.apache.zookeeper</groupId>
<artifactId>zookeeper</artifactId>
</exclusion>
</exclusions>
</dependency>
<dependency>
<groupId>org.apache.zookeeper</groupId>
<artifactId>zookeeper</artifactId>
<version>3.4.12</version>
<exclusions>
<exclusion>
<groupId>org.slf4j</groupId>
<artifactId>slf4j-log4j12</artifactId>
</exclusion>
</exclusions>
</dependency>
```

**gradle.** 

```java
compile('org.springframework.cloud:spring-cloud-starter-zookeeper-all') {
exclude group: 'org.apache.zookeeper', module: 'zookeeper'
}
compile('org.apache.zookeeper:zookeeper:3.4.12') {
exclude group: 'org.slf4j', module: 'slf4j-log4j12'
}
```


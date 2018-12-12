## 65. Consul Agent

A Consul Agent client must be available to all Spring Cloud Consul applications. By default, the Agent client is expected to be at  `localhost:8500` . See the [Agent documentation](https://consul.io/docs/agent/basics.html) for specifics on how to start an Agent client and how to connect to a cluster of Consul Agent Servers. For development, after you have installed consul, you may start a Consul Agent using the following command:

```java
./src/main/bash/local_run_consul.sh
```

This will start an agent in server mode on port 8500, with the ui available at [http://localhost:8500](http://localhost:8500)

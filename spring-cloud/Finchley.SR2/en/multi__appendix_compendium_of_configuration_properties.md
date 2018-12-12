# Part XVIII. Appendix: Compendium of Configuration Properties

|Name|Default|Description|
|----|----|----|
|encrypt.fail-on-error |true |Flag to say that a process should fail if there is an encryption or decryption error. |
|encrypt.key | |A symmetric key. As a stronger alternative consider using a keystore. |
|encrypt.key-store.alias | |Alias for a key in the store. |
|encrypt.key-store.location | |Location of the key store file, e.g. classpath:/keystore.jks. |
|encrypt.key-store.password | |Password that locks the keystore. |
|encrypt.key-store.secret | |Secret protecting the key (defaults to the same as the password). |
|encrypt.rsa.algorithm | |The RSA algorithm to use (DEFAULT or OEAP). Once it is set do not change it (or existing ciphers will not a decryptable). |
|encrypt.rsa.salt |deadbeef |Salt for the random secret used to encrypt cipher text. Once it is set do not change it (or existing ciphers will not a decryptable). |
|encrypt.rsa.strong |false |Flag to indicate that "strong" AES encryption should be used internally. If true then the GCM algorithm is applied to the AES encrypted bytes. Default is false (in which case "standard" CBC is used instead). Once it is set do not change it (or existing ciphers will not a decryptable). |
|encrypt.salt |deadbeef |A salt for the symmetric key in the form of a hex-encoded byte array. As a stronger alternative consider using a keystore. |
|endpoints.zookeeper.enabled |true |Enable the /zookeeper endpoint to inspect the state of zookeeper. |
|eureka.client.allow-redirects |false |Indicates whether server can redirect a client request to a backup server/cluster. If set to false, the server will handle the request directly, If set to true, it may send HTTP redirect to the client, with a new server location. |
|eureka.client.availability-zones | |Gets the list of availability zones (used in AWS data centers) for the region in which this instance resides. The changes are effective at runtime at the next registry fetch cycle as specified by registryFetchIntervalSeconds. |
|eureka.client.backup-registry-impl | |Gets the name of the implementation which implements BackupRegistry to fetch the registry information as a fall back option for only the first time when the eureka client starts. This may be needed for applications which needs additional resiliency for registry information without which it cannot operate. |
|eureka.client.cache-refresh-executor-exponential-back-off-bound |10 |Cache refresh executor exponential back off related property. It is a maximum multiplier value for retry delay, in case where a sequence of timeouts occurred. |
|eureka.client.cache-refresh-executor-thread-pool-size |2 |The thread pool size for the cacheRefreshExecutor to initialise with |
|eureka.client.client-data-accept | |EurekaAccept name for client data accept |
|eureka.client.decoder-name | |This is a transient config and once the latest codecs are stable, can be removed (as there will only be one) |
|eureka.client.disable-delta |false |Indicates whether the eureka client should disable fetching of delta and should rather resort to getting the full registry information. Note that the delta fetches can reduce the traffic tremendously, because the rate of change with the eureka server is normally much lower than the rate of fetches. The changes are effective at runtime at the next registry fetch cycle as specified by registryFetchIntervalSeconds |
|eureka.client.dollar-replacement |_- |Get a replacement string for Dollar sign <code>$</code> during serializing/deserializing information in eureka server. |
|eureka.client.enabled |true |Flag to indicate that the Eureka client is enabled. |
|eureka.client.encoder-name | |This is a transient config and once the latest codecs are stable, can be removed (as there will only be one) |
|eureka.client.escape-char-replacement |__ |Get a replacement string for underscore sign <code>_</code> during serializing/deserializing information in eureka server. |
|eureka.client.eureka-connection-idle-timeout-seconds |30 |Indicates how much time (in seconds) that the HTTP connections to eureka server can stay idle before it can be closed. In the AWS environment, it is recommended that the values is 30 seconds or less, since the firewall cleans up the connection information after a few mins leaving the connection hanging in limbo |
|eureka.client.eureka-server-connect-timeout-seconds |5 |Indicates how long to wait (in seconds) before a connection to eureka server needs to timeout. Note that the connections in the client are pooled by org.apache.http.client.HttpClient and this setting affects the actual connection creation and also the wait time to get the connection from the pool. |
|eureka.client.eureka-server-d-n-s-name | |Gets the DNS name to be queried to get the list of eureka servers.This information is not required if the contract returns the service urls by implementing serviceUrls. The DNS mechanism is used when useDnsForFetchingServiceUrls is set to true and the eureka client expects the DNS to configured a certain way so that it can fetch changing eureka servers dynamically. The changes are effective at runtime. |
|eureka.client.eureka-server-port | |Gets the port to be used to construct the service url to contact eureka server when the list of eureka servers come from the DNS.This information is not required if the contract returns the service urls eurekaServerServiceUrls(String). The DNS mechanism is used when useDnsForFetchingServiceUrls is set to true and the eureka client expects the DNS to configured a certain way so that it can fetch changing eureka servers dynamically. The changes are effective at runtime. |
|eureka.client.eureka-server-read-timeout-seconds |8 |Indicates how long to wait (in seconds) before a read from eureka server needs to timeout. |
|eureka.client.eureka-server-total-connections |200 |Gets the total number of connections that is allowed from eureka client to all eureka servers. |
|eureka.client.eureka-server-total-connections-per-host |50 |Gets the total number of connections that is allowed from eureka client to a eureka server host. |
|eureka.client.eureka-server-u-r-l-context | |Gets the URL context to be used to construct the service url to contact eureka server when the list of eureka servers come from the DNS. This information is not required if the contract returns the service urls from eurekaServerServiceUrls. The DNS mechanism is used when useDnsForFetchingServiceUrls is set to true and the eureka client expects the DNS to configured a certain way so that it can fetch changing eureka servers dynamically. The changes are effective at runtime. |
|eureka.client.eureka-service-url-poll-interval-seconds |0 |Indicates how often(in seconds) to poll for changes to eureka server information. Eureka servers could be added or removed and this setting controls how soon the eureka clients should know about it. |
|eureka.client.fetch-registry |true |Indicates whether this client should fetch eureka registry information from eureka server. |
|eureka.client.fetch-remote-regions-registry | |Comma separated list of regions for which the eureka registry information will be fetched. It is mandatory to define the availability zones for each of these regions as returned by availabilityZones. Failing to do so, will result in failure of discovery client startup. |
|eureka.client.filter-only-up-instances |true |Indicates whether to get the applications after filtering the applications for instances with only InstanceStatus UP states. |
|eureka.client.g-zip-content |true |Indicates whether the content fetched from eureka server has to be compressed whenever it is supported by the server. The registry information from the eureka server is compressed for optimum network traffic. |
|eureka.client.healthcheck.enabled |true |Enables the Eureka health check handler. |
|eureka.client.heartbeat-executor-exponential-back-off-bound |10 |Heartbeat executor exponential back off related property. It is a maximum multiplier value for retry delay, in case where a sequence of timeouts occurred. |
|eureka.client.heartbeat-executor-thread-pool-size |2 |The thread pool size for the heartbeatExecutor to initialise with |
|eureka.client.initial-instance-info-replication-interval-seconds |40 |Indicates how long initially (in seconds) to replicate instance info to the eureka server |
|eureka.client.instance-info-replication-interval-seconds |30 |Indicates how often(in seconds) to replicate instance changes to be replicated to the eureka server. |
|eureka.client.log-delta-diff |false |Indicates whether to log differences between the eureka server and the eureka client in terms of registry information. Eureka client tries to retrieve only delta changes from eureka server to minimize network traffic. After receiving the deltas, eureka client reconciles the information from the server to verify it has not missed out some information. Reconciliation failures could happen when the client has had network issues communicating to server.If the reconciliation fails, eureka client gets the full registry information. While getting the full registry information, the eureka client can log the differences between the client and the server and this setting controls that. The changes are effective at runtime at the next registry fetch cycle as specified by registryFetchIntervalSecondsr |
|eureka.client.on-demand-update-status-change |true |If set to true, local status updates via ApplicationInfoManager will trigger on-demand (but rate limited) register/updates to remote eureka servers |
|eureka.client.prefer-same-zone-eureka |true |Indicates whether or not this instance should try to use the eureka server in the same zone for latency and/or other reason. Ideally eureka clients are configured to talk to servers in the same zone The changes are effective at runtime at the next registry fetch cycle as specified by registryFetchIntervalSeconds |
|eureka.client.property-resolver | | |
|eureka.client.proxy-host | |Gets the proxy host to eureka server if any. |
|eureka.client.proxy-password | |Gets the proxy password if any. |
|eureka.client.proxy-port | |Gets the proxy port to eureka server if any. |
|eureka.client.proxy-user-name | |Gets the proxy user name if any. |
|eureka.client.region |us-east-1 |Gets the region (used in AWS datacenters) where this instance resides. |
|eureka.client.register-with-eureka |true |Indicates whether or not this instance should register its information with eureka server for discovery by others. In some cases, you do not want your instances to be discovered whereas you just want do discover other instances. |
|eureka.client.registry-fetch-interval-seconds |30 |Indicates how often(in seconds) to fetch the registry information from the eureka server. |
|eureka.client.registry-refresh-single-vip-address | |Indicates whether the client is only interested in the registry information for a single VIP. |
|eureka.client.service-url | |Map of availability zone to list of fully qualified URLs to communicate with eureka server. Each value can be a single URL or a comma separated list of alternative locations. Typically the eureka server URLs carry protocol,host,port,context and version information if any. Example: [http://ec2-256-156-243-129.compute-1.amazonaws.com:7001/eureka/](http://ec2-256-156-243-129.compute-1.amazonaws.com:7001/eureka/) The changes are effective at runtime at the next service url refresh cycle as specified by eurekaServiceUrlPollIntervalSeconds. |
|eureka.client.should-enforce-registration-at-init |false |Indicates whether the client should enforce registration during initialization. Defaults to false. |
|eureka.client.should-unregister-on-shutdown |true |Indicates whether the client should explicitly unregister itself from the remote server on client shutdown. |
|eureka.client.use-dns-for-fetching-service-urls |false |Indicates whether the eureka client should use the DNS mechanism to fetch a list of eureka servers to talk to. When the DNS name is updated to have additional servers, that information is used immediately after the eureka client polls for that information as specified in eurekaServiceUrlPollIntervalSeconds. Alternatively, the service urls can be returned serviceUrls, but the users should implement their own mechanism to return the updated list in case of changes. The changes are effective at runtime. |
|eureka.dashboard.enabled |true |Flag to enable the Eureka dashboard. Default true. |
|eureka.dashboard.path |/ |The path to the Eureka dashboard (relative to the servlet path). Defaults to "/". |
|eureka.instance.a-s-g-name | |Gets the AWS autoscaling group name associated with this instance. This information is specifically used in an AWS environment to automatically put an instance out of service after the instance is launched and it has been disabled for traffic.. |
|eureka.instance.app-group-name | |Get the name of the application group to be registered with eureka. |
|eureka.instance.appname |unknown |Get the name of the application to be registered with eureka. |
|eureka.instance.data-center-info | |Returns the data center this instance is deployed. This information is used to get some AWS specific instance information if the instance is deployed in AWS. |
|eureka.instance.default-address-resolution-order |[] | |
|eureka.instance.environment | | |
|eureka.instance.health-check-url | |Gets the absolute health check page URL for this instance. The users can provide the healthCheckUrlPath if the health check page resides in the same instance talking to eureka, else in the cases where the instance is a proxy for some other server, users can provide the full URL. If the full URL is provided it takes precedence. <p> It is normally used for making educated decisions based on the health of the instance - for example, it can be used to determine whether to proceed deployments to an entire farm or stop the deployments without causing further damage. The full URL should follow the format [http://${eureka.hostname}:7001/](http://${eureka.hostname}:7001/) where the value ${eureka.hostname} is replaced at runtime. |
|eureka.instance.health-check-url-path | |Gets the relative health check URL path for this instance. The health check page URL is then constructed out of the hostname and the type of communication - secure or unsecure as specified in securePort and nonSecurePort. It is normally used for making educated decisions based on the health of the instance - for example, it can be used to determine whether to proceed deployments to an entire farm or stop the deployments without causing further damage. |
|eureka.instance.home-page-url | |Gets the absolute home page URL for this instance. The users can provide the homePageUrlPath if the home page resides in the same instance talking to eureka, else in the cases where the instance is a proxy for some other server, users can provide the full URL. If the full URL is provided it takes precedence. It is normally used for informational purposes for other services to use it as a landing page. The full URL should follow the format [http://${eureka.hostname}:7001/](http://${eureka.hostname}:7001/) where the value ${eureka.hostname} is replaced at runtime. |
|eureka.instance.home-page-url-path |/ |Gets the relative home page URL Path for this instance. The home page URL is then constructed out of the hostName and the type of communication - secure or unsecure. It is normally used for informational purposes for other services to use it as a landing page. |
|eureka.instance.hostname | |The hostname if it can be determined at configuration time (otherwise it will be guessed from OS primitives). |
|eureka.instance.initial-status | |Initial status to register with rmeote Eureka server. |
|eureka.instance.instance-enabled-onit |false |Indicates whether the instance should be enabled for taking traffic as soon as it is registered with eureka. Sometimes the application might need to do some pre-processing before it is ready to take traffic. |
|eureka.instance.instance-id | |Get the unique Id (within the scope of the appName) of this instance to be registered with eureka. |
|eureka.instance.ip-address | |Get the IPAdress of the instance. This information is for academic purposes only as the communication from other instances primarily happen using the information supplied in {@link #getHostName(boolean)}. |
|eureka.instance.lease-expiration-duration-in-seconds |90 |Indicates the time in seconds that the eureka server waits since it received the last heartbeat before it can remove this instance from its view and there by disallowing traffic to this instance. Setting this value too long could mean that the traffic could be routed to the instance even though the instance is not alive. Setting this value too small could mean, the instance may be taken out of traffic because of temporary network glitches.This value to be set to atleast higher than the value specified in leaseRenewalIntervalInSeconds. |
|eureka.instance.lease-renewal-interval-in-seconds |30 |Indicates how often (in seconds) the eureka client needs to send heartbeats to eureka server to indicate that it is still alive. If the heartbeats are not received for the period specified in leaseExpirationDurationInSeconds, eureka server will remove the instance from its view, there by disallowing traffic to this instance. Note that the instance could still not take traffic if it implements HealthCheckCallback and then decides to make itself unavailable. |
|eureka.instance.metadata-map | |Gets the metadata name/value pairs associated with this instance. This information is sent to eureka server and can be used by other instances. |
|eureka.instance.namespace |eureka |Get the namespace used to find properties. Ignored in Spring Cloud. |
|eureka.instance.non-secure-port |80 |Get the non-secure port on which the instance should receive traffic. |
|eureka.instance.non-secure-port-enabled |true |Indicates whether the non-secure port should be enabled for traffic or not. |
|eureka.instance.prefer-ip-address |false |Flag to say that, when guessing a hostname, the IP address of the server should be used in prference to the hostname reported by the OS. |
|eureka.instance.registry.default-open-for-traffic-count |1 |Value used in determining when leases are cancelled, default to 1 for standalone. Should be set to 0 for peer replicated eurekas |
|eureka.instance.registry.expected-number-of-renews-per-min |1 | |
|eureka.instance.secure-health-check-url | |Gets the absolute secure health check page URL for this instance. The users can provide the secureHealthCheckUrl if the health check page resides in the same instance talking to eureka, else in the cases where the instance is a proxy for some other server, users can provide the full URL. If the full URL is provided it takes precedence. <p> It is normally used for making educated decisions based on the health of the instance - for example, it can be used to determine whether to proceed deployments to an entire farm or stop the deployments without causing further damage. The full URL should follow the format [http://${eureka.hostname}:7001/](http://${eureka.hostname}:7001/) where the value ${eureka.hostname} is replaced at runtime. |
|eureka.instance.secure-port |443 |Get the Secure port on which the instance should receive traffic. |
|eureka.instance.secure-port-enabled |false |Indicates whether the secure port should be enabled for traffic or not. |
|eureka.instance.secure-virtual-host-name |unknown |Gets the secure virtual host name defined for this instance. This is typically the way other instance would find this instance by using the secure virtual host name.Think of this as similar to the fully qualified domain name, that the users of your services will need to find this instance. |
|eureka.instance.status-page-url | |Gets the absolute status page URL path for this instance. The users can provide the statusPageUrlPath if the status page resides in the same instance talking to eureka, else in the cases where the instance is a proxy for some other server, users can provide the full URL. If the full URL is provided it takes precedence. It is normally used for informational purposes for other services to find about the status of this instance. Users can provide a simple HTML indicating what is the current status of the instance. |
|eureka.instance.status-page-url-path | |Gets the relative status page URL path for this instance. The status page URL is then constructed out of the hostName and the type of communication - secure or unsecure as specified in securePort and nonSecurePort. It is normally used for informational purposes for other services to find about the status of this instance. Users can provide a simple HTML indicating what is the current status of the instance. |
|eureka.instance.virtual-host-name |unknown |Gets the virtual host name defined for this instance. This is typically the way other instance would find this instance by using the virtual host name.Think of this as similar to the fully qualified domain name, that the users of your services will need to find this instance. |
|eureka.server.a-s-g-cache-expiry-timeout-ms |0 | |
|eureka.server.a-s-g-query-timeout-ms |300 | |
|eureka.server.a-s-g-update-interval-ms |0 | |
|eureka.server.a-w-s-access-id | | |
|eureka.server.a-w-s-secret-key | | |
|eureka.server.batch-replication |false | |
|eureka.server.binding-strategy | | |
|eureka.server.delta-retention-timer-interval-in-ms |0 | |
|eureka.server.disable-delta |false | |
|eureka.server.disable-delta-for-remote-regions |false | |
|eureka.server.disable-transparent-fallback-to-other-region |false | |
|eureka.server.e-i-p-bind-rebind-retries |3 | |
|eureka.server.e-i-p-binding-retry-interval-ms |0 | |
|eureka.server.e-i-p-binding-retry-interval-ms-when-unbound |0 | |
|eureka.server.enable-replicated-request-compression |false | |
|eureka.server.enable-self-preservation |true | |
|eureka.server.eviction-interval-timer-in-ms |0 | |
|eureka.server.g-zip-content-from-remote-region |true | |
|eureka.server.json-codec-name | | |
|eureka.server.list-auto-scaling-groups-role-name |ListAutoScalingGroups | |
|eureka.server.log-identity-headers |true | |
|eureka.server.max-elements-in-peer-replication-pool |10000 | |
|eureka.server.max-elements-in-status-replication-pool |10000 | |
|eureka.server.max-idle-thread-age-in-minutes-for-peer-replication |15 | |
|eureka.server.max-idle-thread-in-minutes-age-for-status-replication |10 | |
|eureka.server.max-threads-for-peer-replication |20 | |
|eureka.server.max-threads-for-status-replication |1 | |
|eureka.server.max-time-for-replication |30000 | |
|eureka.server.min-available-instances-for-peer-replication |-1 | |
|eureka.server.min-threads-for-peer-replication |5 | |
|eureka.server.min-threads-for-status-replication |1 | |
|eureka.server.number-of-replication-retries |5 | |
|eureka.server.peer-eureka-nodes-update-interval-ms |0 | |
|eureka.server.peer-eureka-status-refresh-time-interval-ms |0 | |
|eureka.server.peer-node-connect-timeout-ms |200 | |
|eureka.server.peer-node-connection-idle-timeout-seconds |30 | |
|eureka.server.peer-node-read-timeout-ms |200 | |
|eureka.server.peer-node-total-connections |1000 | |
|eureka.server.peer-node-total-connections-per-host |500 | |
|eureka.server.prime-aws-replica-connections |true | |
|eureka.server.property-resolver | | |
|eureka.server.rate-limiter-burst-size |10 | |
|eureka.server.rate-limiter-enabled |false | |
|eureka.server.rate-limiter-full-fetch-average-rate |100 | |
|eureka.server.rate-limiter-privileged-clients | | |
|eureka.server.rate-limiter-registry-fetch-average-rate |500 | |
|eureka.server.rate-limiter-throttle-standard-clients |false | |
|eureka.server.registry-sync-retries |0 | |
|eureka.server.registry-sync-retry-wait-ms |0 | |
|eureka.server.remote-region-app-whitelist | | |
|eureka.server.remote-region-connect-timeout-ms |1000 | |
|eureka.server.remote-region-connection-idle-timeout-seconds |30 | |
|eureka.server.remote-region-fetch-thread-pool-size |20 | |
|eureka.server.remote-region-read-timeout-ms |1000 | |
|eureka.server.remote-region-registry-fetch-interval |30 | |
|eureka.server.remote-region-total-connections |1000 | |
|eureka.server.remote-region-total-connections-per-host |500 | |
|eureka.server.remote-region-trust-store | | |
|eureka.server.remote-region-trust-store-password |changeit | |
|eureka.server.remote-region-urls | | |
|eureka.server.remote-region-urls-with-name | | |
|eureka.server.renewal-percent-threshold |0.85 | |
|eureka.server.renewal-threshold-update-interval-ms |0 | |
|eureka.server.response-cache-auto-expiration-in-seconds |180 | |
|eureka.server.response-cache-update-interval-ms |0 | |
|eureka.server.retention-time-in-m-s-in-delta-queue |0 | |
|eureka.server.route53-bind-rebind-retries |3 | |
|eureka.server.route53-binding-retry-interval-ms |0 | |
|eureka.server.route53-domain-t-t-l |30 | |
|eureka.server.sync-when-timestamp-differs |true | |
|eureka.server.use-read-only-response-cache |true | |
|eureka.server.wait-time-in-ms-when-sync-empty |0 | |
|eureka.server.xml-codec-name | | |
|health.config.enabled |false |Flag to indicate that the config server health indicator should be installed. |
|health.config.time-to-live |0 |Time to live for cached result, in milliseconds. Default 300000 (5 min). |
|hystrix.metrics.enabled |true |Enable Hystrix metrics polling. Defaults to true. |
|hystrix.metrics.polling-interval-ms |2000 |Interval between subsequent polling of metrics. Defaults to 2000 ms. |
|hystrix.shareSecurityContext |false |Enables auto-configuration of the Hystrix concurrency strategy plugin hook who will transfer the  `SecurityContext`  from your main thread to the one used by the Hystrix command. |
|management.endpoint.bindings.cache.time-to-live |0ms |Maximum time that a response can be cached. |
|management.endpoint.bindings.enabled |true |Whether to enable the bindings endpoint. |
|management.endpoint.bus-env.enabled |true |Whether to enable the bus-env endpoint. |
|management.endpoint.bus-refresh.enabled |true |Whether to enable the bus-refresh endpoint. |
|management.endpoint.channels.cache.time-to-live |0ms |Maximum time that a response can be cached. |
|management.endpoint.channels.enabled |true |Whether to enable the channels endpoint. |
|management.endpoint.consul.cache.time-to-live |0ms |Maximum time that a response can be cached. |
|management.endpoint.consul.enabled |true |Whether to enable the consul endpoint. |
|management.endpoint.env.post.enabled |true |Enable changing the Environment through a POST to /env. |
|management.endpoint.features.cache.time-to-live |0ms |Maximum time that a response can be cached. |
|management.endpoint.features.enabled |true |Whether to enable the features endpoint. |
|management.endpoint.gateway.enabled |true |Whether to enable the gateway endpoint. |
|management.endpoint.hystrix.config | |Hystrix settings. These are traditionally set using servlet parameters. Refer to the documentation of Hystrix for more details. |
|management.endpoint.hystrix.stream.enabled |true |Whether to enable the hystrix.stream endpoint. |
|management.endpoint.pause.enabled |true |Enable the /pause endpoint (to send Lifecycle.stop()). |
|management.endpoint.refresh.enabled |true |Enable the /refresh endpoint to refresh configuration and re-initialize refresh scoped beans. |
|management.endpoint.restart.enabled |true |Enable the /restart endpoint to restart the application context. |
|management.endpoint.resume.enabled |true |Enable the /resume endpoint (to send Lifecycle.start()). |
|management.endpoint.service-registry.cache.time-to-live |0ms |Maximum time that a response can be cached. |
|management.endpoint.service-registry.enabled |true |Whether to enable the service-registry endpoint. |
|management.health.refresh.enabled |true |Enable the health endpoint for the refresh scope. |
|management.health.zookeeper.enabled |true |Enable the health endpoint for zookeeper. |
|management.metrics.binders.hystrix.enabled |true |Enables creation of OK Http Client factory beans. |
|proxy.auth.load-balanced |false | |
|proxy.auth.routes | |Authentication strategy per route. |
|ribbon.eager-load.clients | | |
|ribbon.eager-load.enabled |false | |
|ribbon.eureka.enabled |true |Enables the use of Eureka with Ribbon. |
|ribbon.http.client.enabled |false |Deprecated property to enable Ribbon RestClient. |
|ribbon.okhttp.enabled |false |Enables the use of the OK HTTP Client with Ribbon. |
|ribbon.restclient.enabled |false |Enables the use of the deprecated Ribbon RestClient. |
|ribbon.secure-ports | | |
|spring.cloud.bus.ack.destination-service | |Service that wants to listen to acks. By default null (meaning all services). |
|spring.cloud.bus.ack.enabled |true |Flag to switch off acks (default on). |
|spring.cloud.bus.destination |springCloudBus |Name of Spring Cloud Stream destination for messages. |
|spring.cloud.bus.enabled |true |Flag to indicate that the bus is enabled. |
|spring.cloud.bus.env.enabled |true |Flag to switch off environment change events (default on). |
|spring.cloud.bus.id |application |The identifier for this application instance. |
|spring.cloud.bus.refresh.enabled |true |Flag to switch off refresh events (default on). |
|spring.cloud.bus.trace.enabled |false |Flag to switch on tracing of acks (default off). |
|spring.cloud.cloudfoundry.discovery.default-server-port |80 |Port to use when no port is defined by ribbon. |
|spring.cloud.cloudfoundry.discovery.enabled |true |Flag to indicate that discovery is enabled. |
|spring.cloud.cloudfoundry.discovery.heartbeat-frequency |5000 |Frequency in milliseconds of poll for heart beat. The client will poll on this frequency and broadcast a list of service ids. |
|spring.cloud.cloudfoundry.org | |Organization name to initially target. |
|spring.cloud.cloudfoundry.password | |Password for user to authenticate and obtain token. |
|spring.cloud.cloudfoundry.skip-ssl-validation |false | |
|spring.cloud.cloudfoundry.space | |Space name to initially target. |
|spring.cloud.cloudfoundry.url | |URL of Cloud Foundry API (Cloud Controller). |
|spring.cloud.cloudfoundry.username | |Username to authenticate (usually an email address). |
|spring.cloud.config.allow-override |true |Flag to indicate that {@link #isOverrideSystemProperties() systemPropertiesOverride} can be used. Set to false to prevent users from changing the default accidentally. Default true. |
|spring.cloud.config.discovery.enabled |false |Flag to indicate that config server discovery is enabled (config server URL will be looked up via discovery). |
|spring.cloud.config.discovery.service-id |configserver |Service id to locate config server. |
|spring.cloud.config.enabled |true |Flag to say that remote configuration is enabled. Default true; |
|spring.cloud.config.fail-fast |false |Flag to indicate that failure to connect to the server is fatal (default false). |
|spring.cloud.config.headers | |Additional headers used to create the client request. |
|spring.cloud.config.label | |The label name to use to pull remote configuration properties. The default is set on the server (generally "master" for a git based server). |
|spring.cloud.config.name | |Name of application used to fetch remote properties. |
|spring.cloud.config.override-none |false |Flag to indicate that when {@link #setAllowOverride(boolean) allowOverride} is true, external properties should take lowest priority, and not override any existing property sources (including local config files). Default false. |
|spring.cloud.config.override-system-properties |true |Flag to indicate that the external properties should override system properties. Default true. |
|spring.cloud.config.password | |The password to use (HTTP Basic) when contacting the remote server. |
|spring.cloud.config.profile |default |The default profile to use when fetching remote configuration (comma-separated). Default is "default". |
|spring.cloud.config.request-read-timeout |0 |timeout on waiting to read data from the Config Server. |
|spring.cloud.config.retry.initial-interval |1000 |Initial retry interval in milliseconds. |
|spring.cloud.config.retry.max-attempts |6 |Maximum number of attempts. |
|spring.cloud.config.retry.max-interval |2000 |Maximum interval for backoff. |
|spring.cloud.config.retry.multiplier |1.1 |Multiplier for next interval. |
|spring.cloud.config.send-state |true |Flag to indicate whether to send state. Default true. |
|spring.cloud.config.server.accept-empty |true |Flag to indicate that If HTTP 404 needs to be sent if Application is not Found |
|spring.cloud.config.server.bootstrap |false |Flag indicating that the config server should initialize its own Environment with properties from the remote repository. Off by default because it delays startup but can be useful when embedding the server in another application. |
|spring.cloud.config.server.default-application-name |application |Default application name when incoming requests do not have a specific one. |
|spring.cloud.config.server.default-label | |Default repository label when incoming requests do not have a specific label. |
|spring.cloud.config.server.default-profile |default |Default application profile when incoming requests do not have a specific one. |
|spring.cloud.config.server.encrypt.enabled |true |Enable decryption of environment properties before sending to client. |
|spring.cloud.config.server.git.basedir | |Base directory for local working copy of repository. |
|spring.cloud.config.server.git.clone-on-start |false |Flag to indicate that the repository should be cloned on startup (not on demand). Generally leads to slower startup but faster first query. |
|spring.cloud.config.server.git.default-label | |The default label to be used with the remore repository |
|spring.cloud.config.server.git.delete-untracked-branches |false |Flag to indicate that the branch should be deleted locally if it’s origin tracked branch was removed. |
|spring.cloud.config.server.git.force-pull |false |Flag to indicate that the repository should force pull. If true discard any local changes and take from remote repository. |
|spring.cloud.config.server.git.host-key | |Valid SSH host key. Must be set if hostKeyAlgorithm is also set. |
|spring.cloud.config.server.git.host-key-algorithm | |One of ssh-dss, ssh-rsa, ecdsa-sha2-nistp256, ecdsa-sha2-nistp384, or ecdsa-sha2-nistp521. Must be set if hostKey is also set. |
|spring.cloud.config.server.git.ignore-local-ssh-settings |false |If true, use property-based instead of file-based SSH config. |
|spring.cloud.config.server.git.known-hosts-file | |Location of custom .known_hosts file. |
|spring.cloud.config.server.git.order | |The order of the environment repository. |
|spring.cloud.config.server.git.passphrase | |Passphrase for unlocking your ssh private key. |
|spring.cloud.config.server.git.password | |Password for authentication with remote repository. |
|spring.cloud.config.server.git.preferred-authentications | |Override server authentication method order. This should allow for evading login prompts if server has keyboard-interactive authentication before the publickey method. |
|spring.cloud.config.server.git.private-key | |Valid SSH private key. Must be set if ignoreLocalSshSettings is true and Git URI is SSH format. |
|spring.cloud.config.server.git.proxy | |HTTP proxy configuration. |
|spring.cloud.config.server.git.refresh-rate |0 |Time (in seconds) between refresh of the git repository |
|spring.cloud.config.server.git.repos | |Map of repository identifier to location and other properties. |
|spring.cloud.config.server.git.search-paths | |Search paths to use within local working copy. By default searches only the root. |
|spring.cloud.config.server.git.skip-ssl-validation |false |Flag to indicate that SSL certificate validation should be bypassed when communicating with a repository served over an HTTPS connection. |
|spring.cloud.config.server.git.strict-host-key-checking |true |If false, ignore errors with host key |
|spring.cloud.config.server.git.timeout |5 |Timeout (in seconds) for obtaining HTTP or SSH connection (if applicable), defaults to 5 seconds. |
|spring.cloud.config.server.git.uri | |URI of remote repository. |
|spring.cloud.config.server.git.username | |Username for authentication with remote repository. |
|spring.cloud.config.server.health.repositories | | |
|spring.cloud.config.server.jdbc.order |0 | |
|spring.cloud.config.server.jdbc.sql |SELECT KEY, VALUE from PROPERTIES where APPLICATION=? and PROFILE=? and LABEL=? |SQL used to query database for keys and values |
|spring.cloud.config.server.native.add-label-locations |true |Flag to determine whether label locations should be added. |
|spring.cloud.config.server.native.default-label |master | |
|spring.cloud.config.server.native.fail-on-error |false |Flag to determine how to handle exceptions during decryption (default false). |
|spring.cloud.config.server.native.order | | |
|spring.cloud.config.server.native.search-locations |[] |Locations to search for configuration files. Defaults to the same as a Spring Boot app so [classpath:/,classpath:/config/,file:./,file:./config/]. |
|spring.cloud.config.server.native.version | |Version string to be reported for native repository |
|spring.cloud.config.server.overrides | |Extra map for a property source to be sent to all clients unconditionally. |
|spring.cloud.config.server.prefix | |Prefix for configuration resource paths (default is empty). Useful when embedding in another application when you don’t want to change the context path or servlet path. |
|spring.cloud.config.server.strip-document-from-yaml |true |Flag to indicate that YAML documents that are text or collections (not a map) should be returned in "native" form. |
|spring.cloud.config.server.svn.basedir | |Base directory for local working copy of repository. |
|spring.cloud.config.server.svn.default-label | |The default label to be used with the remore repository |
|spring.cloud.config.server.svn.order | |The order of the environment repository. |
|spring.cloud.config.server.svn.passphrase | |Passphrase for unlocking your ssh private key. |
|spring.cloud.config.server.svn.password | |Password for authentication with remote repository. |
|spring.cloud.config.server.svn.search-paths | |Search paths to use within local working copy. By default searches only the root. |
|spring.cloud.config.server.svn.strict-host-key-checking |true |Reject incoming SSH host keys from remote servers not in the known host list. |
|spring.cloud.config.server.svn.uri | |URI of remote repository. |
|spring.cloud.config.server.svn.username | |Username for authentication with remote repository. |
|spring.cloud.config.server.vault.backend |secret |Vault backend. Defaults to secret. |
|spring.cloud.config.server.vault.default-key |application |The key in vault shared by all applications. Defaults to application. Set to empty to disable. |
|spring.cloud.config.server.vault.host |127.0.0.1 |Vault host. Defaults to 127.0.0.1. |
|spring.cloud.config.server.vault.kv-version |1 |Value to indicate which version of Vault kv backend is used. Defaults to 1. |
|spring.cloud.config.server.vault.order | | |
|spring.cloud.config.server.vault.port |8200 |Vault port. Defaults to 8200. |
|spring.cloud.config.server.vault.profile-separator |, |Vault profile separator. Defaults to comma. |
|spring.cloud.config.server.vault.proxy | |HTTP proxy configuration. |
|spring.cloud.config.server.vault.scheme |http |Vault scheme. Defaults to http. |
|spring.cloud.config.server.vault.skip-ssl-validation |false |Flag to indicate that SSL certificate validation should be bypassed when communicating with a repository served over an HTTPS connection. |
|spring.cloud.config.server.vault.timeout |5 |Timeout (in seconds) for obtaining HTTP connection, defaults to 5 seconds. |
|spring.cloud.config.token | |Security Token passed thru to underlying environment repository. |
|spring.cloud.config.uri |[[http://localhost:8888](http://localhost:8888)] |The URI of the remote server (default [http://localhost:8888](http://localhost:8888)). |
|spring.cloud.config.username | |The username to use (HTTP Basic) when contacting the remote server. |
|spring.cloud.consul.config.acl-token | | |
|spring.cloud.consul.config.data-key |data |If format is Format.PROPERTIES or Format.YAML then the following field is used as key to look up consul for configuration. |
|spring.cloud.consul.config.default-context |application | |
|spring.cloud.consul.config.enabled |true | |
|spring.cloud.consul.config.fail-fast |true |Throw exceptions during config lookup if true, otherwise, log warnings. |
|spring.cloud.consul.config.format | | |
|spring.cloud.consul.config.name | |Alternative to spring.application.name to use in looking up values in consul KV. |
|spring.cloud.consul.config.prefix |config | |
|spring.cloud.consul.config.profile-separator |, | |
|spring.cloud.consul.config.watch.delay |1000 |The value of the fixed delay for the watch in millis. Defaults to 1000. |
|spring.cloud.consul.config.watch.enabled |true |If the watch is enabled. Defaults to true. |
|spring.cloud.consul.config.watch.wait-time |55 |The number of seconds to wait (or block) for watch query, defaults to 55. Needs to be less than default ConsulClient (defaults to 60). To increase ConsulClient timeout create a ConsulClient bean with a custom ConsulRawClient with a custom HttpClient. |
|spring.cloud.consul.discovery.acl-token | | |
|spring.cloud.consul.discovery.catalog-services-watch-delay |1000 |The delay between calls to watch consul catalog in millis, default is 1000. |
|spring.cloud.consul.discovery.catalog-services-watch-timeout |2 |The number of seconds to block while watching consul catalog, default is 2. |
|spring.cloud.consul.discovery.datacenters | |Map of serviceId’s → datacenter to query for in server list. This allows looking up services in another datacenters. |
|spring.cloud.consul.discovery.default-query-tag | |Tag to query for in service list if one is not listed in serverListQueryTags. |
|spring.cloud.consul.discovery.default-zone-metadata-name |zone |Service instance zone comes from metadata. This allows changing the metadata tag name. |
|spring.cloud.consul.discovery.deregister |true |Disable automatic de-registration of service in consul. |
|spring.cloud.consul.discovery.enabled |true |Is service discovery enabled? |
|spring.cloud.consul.discovery.fail-fast |true |Throw exceptions during service registration if true, otherwise, log warnings (defaults to true). |
|spring.cloud.consul.discovery.health-check-critical-timeout | |Timeout to deregister services critical for longer than timeout (e.g. 30m). Requires consul version 7.x or higher. |
|spring.cloud.consul.discovery.health-check-interval |10s |How often to perform the health check (e.g. 10s), defaults to 10s. |
|spring.cloud.consul.discovery.health-check-path |/actuator/health |Alternate server path to invoke for health checking |
|spring.cloud.consul.discovery.health-check-timeout | |Timeout for health check (e.g. 10s). |
|spring.cloud.consul.discovery.health-check-tls-skip-verify | |Skips certificate verification during service checks if true, otherwise runs certificate verification. |
|spring.cloud.consul.discovery.health-check-url | |Custom health check url to override default |
|spring.cloud.consul.discovery.heartbeat.enabled |false | |
|spring.cloud.consul.discovery.heartbeat.interval-ratio | | |
|spring.cloud.consul.discovery.heartbeat.ttl-unit |s | |
|spring.cloud.consul.discovery.heartbeat.ttl-value |30 | |
|spring.cloud.consul.discovery.hostname | |Hostname to use when accessing server |
|spring.cloud.consul.discovery.instance-group | |Service instance group |
|spring.cloud.consul.discovery.instance-id | |Unique service instance id |
|spring.cloud.consul.discovery.instance-zone | |Service instance zone |
|spring.cloud.consul.discovery.ip-address | |IP address to use when accessing service (must also set preferIpAddress to use) |
|spring.cloud.consul.discovery.lifecycle.enabled |true | |
|spring.cloud.consul.discovery.management-port | |Port to register the management service under (defaults to management port) |
|spring.cloud.consul.discovery.management-suffix |management |Suffix to use when registering management service |
|spring.cloud.consul.discovery.management-tags | |Tags to use when registering management service |
|spring.cloud.consul.discovery.port | |Port to register the service under (defaults to listening port) |
|spring.cloud.consul.discovery.prefer-agent-address |false |Source of how we will determine the address to use |
|spring.cloud.consul.discovery.prefer-ip-address |false |Use ip address rather than hostname during registration |
|spring.cloud.consul.discovery.query-passing |false |Add the 'passing` parameter to /v1/health/service/serviceName. This pushes health check passing to the server. |
|spring.cloud.consul.discovery.register |true |Register as a service in consul. |
|spring.cloud.consul.discovery.register-health-check |true |Register health check in consul. Useful during development of a service. |
|spring.cloud.consul.discovery.scheme |http |Whether to register an http or https service |
|spring.cloud.consul.discovery.server-list-query-tags | |Map of serviceId’s → tag to query for in server list. This allows filtering services by a single tag. |
|spring.cloud.consul.discovery.service-name | |Service name |
|spring.cloud.consul.discovery.tags | |Tags to use when registering service |
|spring.cloud.consul.enabled |true |Is spring cloud consul enabled |
|spring.cloud.consul.host |localhost |Consul agent hostname. Defaults to 'localhost'. |
|spring.cloud.consul.port |8500 |Consul agent port. Defaults to '8500'. |
|spring.cloud.consul.retry.initial-interval |1000 |Initial retry interval in milliseconds. |
|spring.cloud.consul.retry.max-attempts |6 |Maximum number of attempts. |
|spring.cloud.consul.retry.max-interval |2000 |Maximum interval for backoff. |
|spring.cloud.consul.retry.multiplier |1.1 |Multiplier for next interval. |
|spring.cloud.consul.scheme | |Consul agent scheme (HTTP/HTTPS). If there is no scheme in address - client will use HTTP. |
|spring.cloud.consul.tls.certificate-password | |Password to open the certificate. |
|spring.cloud.consul.tls.certificate-path | |File path to the certificate. |
|spring.cloud.consul.tls.key-store-instance-type | |Type of key framework to use. |
|spring.cloud.consul.tls.key-store-password | |Password to an external keystore |
|spring.cloud.consul.tls.key-store-path | |Path to an external keystore |
|spring.cloud.discovery.client.health-indicator.enabled |true | |
|spring.cloud.discovery.client.health-indicator.include-description |false | |
|spring.cloud.discovery.client.simple.instances | | |
|spring.cloud.discovery.client.simple.local.metadata | |Metadata for the service instance. Can be used by discovery clients to modify their behaviour per instance, e.g. when load balancing. |
|spring.cloud.discovery.client.simple.local.service-id | |The identifier or name for the service. Multiple instances might share the same service id. |
|spring.cloud.discovery.client.simple.local.uri | |The URI of the service instance. Will be parsed to extract the scheme, hos and port. |
|spring.cloud.gateway.default-filters | |List of filter definitions that are applied to every route. |
|spring.cloud.gateway.discovery.locator.enabled |false |Flag that enables DiscoveryClient gateway integration |
|spring.cloud.gateway.discovery.locator.filters | | |
|spring.cloud.gateway.discovery.locator.include-expression |true |SpEL expression that will evaluate whether to include a service in gateway integration or not, defaults to: true |
|spring.cloud.gateway.discovery.locator.lower-case-service-id |false |Option to lower case serviceId in predicates and filters, defaults to false. Useful with eureka when it automatically uppercases serviceId. so MYSERIVCE, would match /myservice/** |
|spring.cloud.gateway.discovery.locator.predicates | | |
|spring.cloud.gateway.discovery.locator.route-id-prefix | |The prefix for the routeId, defaults to discoveryClient.getClass().getSimpleName() + "_". Service Id will be appended to create the routeId. |
|spring.cloud.gateway.discovery.locator.url-expression |'lb://'+serviceId |SpEL expression that create the uri for each route, defaults to: 'lb://'+serviceId |
|spring.cloud.gateway.enabled |true |Enables gateway functionality. |
|spring.cloud.gateway.filter.remove-hop-by-hop.headers | | |
|spring.cloud.gateway.filter.remove-hop-by-hop.order | | |
|spring.cloud.gateway.filter.secure-headers.content-security-policy |default-src 'self' https:; font-src 'self' https: data:; img-src 'self' https: data:; object-src 'none'; script-src https:; style-src 'self' https: 'unsafe-inline' | |
|spring.cloud.gateway.filter.secure-headers.content-type-options |nosniff | |
|spring.cloud.gateway.filter.secure-headers.download-options |noopen | |
|spring.cloud.gateway.filter.secure-headers.frame-options |DENY | |
|spring.cloud.gateway.filter.secure-headers.permitted-cross-domain-policies |none | |
|spring.cloud.gateway.filter.secure-headers.referrer-policy |no-referrer | |
|spring.cloud.gateway.filter.secure-headers.strict-transport-security |max-age=631138519 | |
|spring.cloud.gateway.filter.secure-headers.xss-protection-header |1 ; mode=block | |
|spring.cloud.gateway.forwarded.enabled |true |Enables the ForwardedHeadersFilter. |
|spring.cloud.gateway.globalcors.cors-configurations | | |
|spring.cloud.gateway.httpclient.connect-timeout | |The connect timeout in millis, the default is 45s. |
|spring.cloud.gateway.httpclient.pool.acquire-timeout | |Only for type FIXED, the maximum time in millis to wait for aquiring. |
|spring.cloud.gateway.httpclient.pool.max-connections | |Only for type FIXED, the maximum number of connections before starting pending acquisition on existing ones. |
|spring.cloud.gateway.httpclient.pool.name |proxy |The channel pool map name, defaults to proxy. |
|spring.cloud.gateway.httpclient.pool.type | |Type of pool for HttpClient to use, defaults to ELASTIC. |
|spring.cloud.gateway.httpclient.proxy.host | |Hostname for proxy configuration of Netty HttpClient. |
|spring.cloud.gateway.httpclient.proxy.non-proxy-hosts-pattern | |Regular expression (Java) for a configured list of hosts that should be reached directly, bypassing the proxy |
|spring.cloud.gateway.httpclient.proxy.password | |Password for proxy configuration of Netty HttpClient. |
|spring.cloud.gateway.httpclient.proxy.port | |Port for proxy configuration of Netty HttpClient. |
|spring.cloud.gateway.httpclient.proxy.username | |Username for proxy configuration of Netty HttpClient. |
|spring.cloud.gateway.httpclient.response-timeout | |The response timeout. |
|spring.cloud.gateway.httpclient.ssl.close-notify-flush-timeout-millis |3000 | |
|spring.cloud.gateway.httpclient.ssl.close-notify-read-timeout-millis |0 | |
|spring.cloud.gateway.httpclient.ssl.handshake-timeout-millis |10000 | |
|spring.cloud.gateway.httpclient.ssl.trusted-x509-certificates | | |
|spring.cloud.gateway.httpclient.ssl.use-insecure-trust-manager |false |Installs the netty InsecureTrustManagerFactory. This is insecure and not suitable for production. |
|spring.cloud.gateway.metrics.enabled |false |Enables the collection of metrics data. |
|spring.cloud.gateway.proxy.headers | |Fixed header values that will be added to all downstream requests. |
|spring.cloud.gateway.proxy.sensitive | |A set of sensitive header names that will not be sent downstream by default. |
|spring.cloud.gateway.redis-rate-limiter.burst-capacity-header |X-RateLimit-Burst-Capacity |The name of the header that returns the burst capacity configuration. |
|spring.cloud.gateway.redis-rate-limiter.config | | |
|spring.cloud.gateway.redis-rate-limiter.include-headers |true |Whether or not to include headers containing rate limiter information, defaults to true. |
|spring.cloud.gateway.redis-rate-limiter.remaining-header |X-RateLimit-Remaining |The name of the header that returns number of remaining requests during the current second. |
|spring.cloud.gateway.redis-rate-limiter.replenish-rate-header |X-RateLimit-Replenish-Rate |The name of the header that returns the replenish rate configuration. |
|spring.cloud.gateway.routes | |List of Routes |
|spring.cloud.gateway.streaming-media-types | | |
|spring.cloud.gateway.x-forwarded.enabled |true |If the XForwardedHeadersFilter is enabled. |
|spring.cloud.gateway.x-forwarded.for-append |true |If appending X-Forwarded-For as a list is enabled. |
|spring.cloud.gateway.x-forwarded.for-enabled |true |If X-Forwarded-For is enabled. |
|spring.cloud.gateway.x-forwarded.host-append |true |If appending X-Forwarded-Host as a list is enabled. |
|spring.cloud.gateway.x-forwarded.host-enabled |true |If X-Forwarded-Host is enabled. |
|spring.cloud.gateway.x-forwarded.order |0 |The order of the XForwardedHeadersFilter. |
|spring.cloud.gateway.x-forwarded.port-append |true |If appending X-Forwarded-Port as a list is enabled. |
|spring.cloud.gateway.x-forwarded.port-enabled |true |If X-Forwarded-Port is enabled. |
|spring.cloud.gateway.x-forwarded.prefix-append |true |If appending X-Forwarded-Prefix as a list is enabled. |
|spring.cloud.gateway.x-forwarded.prefix-enabled |true |If X-Forwarded-Prefix is enabled. |
|spring.cloud.gateway.x-forwarded.proto-append |true |If appending X-Forwarded-Proto as a list is enabled. |
|spring.cloud.gateway.x-forwarded.proto-enabled |true |If X-Forwarded-Proto is enabled. |
|spring.cloud.hypermedia.refresh.fixed-delay |5000 | |
|spring.cloud.hypermedia.refresh.initial-delay |10000 | |
|spring.cloud.inetutils.default-hostname |localhost |The default hostname. Used in case of errors. |
|spring.cloud.inetutils.default-ip-address |127.0.0.1 |The default ipaddress. Used in case of errors. |
|spring.cloud.inetutils.ignored-interfaces | |List of Java regex expressions for network interfaces that will be ignored. |
|spring.cloud.inetutils.preferred-networks | |List of Java regex expressions for network addresses that will be preferred. |
|spring.cloud.inetutils.timeout-seconds |1 |Timeout in seconds for calculating hostname. |
|spring.cloud.inetutils.use-only-site-local-interfaces |false |Use only interfaces with site local addresses. See {@link InetAddress#isSiteLocalAddress()} for more details. |
|spring.cloud.loadbalancer.retry.enabled |true | |
|spring.cloud.refresh.extra-refreshable |true |Additional class names for beans to post process into refresh scope. |
|spring.cloud.service-registry.auto-registration.enabled |true |If Auto-Service Registration is enabled, default to true. |
|spring.cloud.service-registry.auto-registration.fail-fast |false |Should startup fail if there is no AutoServiceRegistration, default to false. |
|spring.cloud.service-registry.auto-registration.register-management |true |Whether to register the management as a service, defaults to true |
|spring.cloud.stream.binders | |Additional per-binder properties (see {@link BinderProperties}) if more then one binder of the same type is used (i.e., connect to multiple instances of RabbitMq). Here you can specify multiple binder configurations, each with different environment settings. For example; spring.cloud.stream.binders.rabbit1.environment. . . , spring.cloud.stream.binders.rabbit2.environment. . . |
|spring.cloud.stream.binding-retry-interval |30 |Retry interval (in seconds) used to schedule binding attempts. Default: 30 sec. |
|spring.cloud.stream.bindings | |Additional binding properties (see {@link BinderProperties}) per binding name (e.g., 'input`).  For example; This sets the content-type for the 'input' binding of a Sink application: 'spring.cloud.stream.bindings.input.contentType=text/plain' |
|spring.cloud.stream.consul.binder.event-timeout |5 | |
|spring.cloud.stream.default-binder | |The name of the binder to use by all bindings in the event multiple binders available (e.g., 'rabbit'); |
|spring.cloud.stream.dynamic-destinations |[] |A list of destinations that can be bound dynamically. If set, only listed destinations can be bound. |
|spring.cloud.stream.instance-count |1 |The number of deployed instances of an application. Default: 1. NOTE: Could also be managed per individual binding "spring.cloud.stream.bindings.foo.consumer.instance-count" where 'foo' is the name of the binding. |
|spring.cloud.stream.instance-index |0 |The instance id of the application: a number from 0 to instanceCount-1. Used for partitioning and with Kafka. NOTE: Could also be managed per individual binding "spring.cloud.stream.bindings.foo.consumer.instance-index" where 'foo' is the name of the binding. |
|spring.cloud.stream.integration.message-handler-not-propagated-headers | |Message header names that will NOT be copied from the inbound message. |
|spring.cloud.stream.metrics.export-properties | |List of properties that are going to be appended to each message. This gets populate by onApplicationEvent, once the context refreshes to avoid overhead of doing per message basis. |
|spring.cloud.stream.metrics.key | |The name of the metric being emitted. Should be an unique value per application. Defaults to: ${spring.application.name:${vcap.application.name:${spring.config.name:application}}} |
|spring.cloud.stream.metrics.meter-filter | |Pattern to control the 'meters' one wants to capture. By default all 'meters' will be captured. For example, 'spring.integration.*' will only capture metric information for meters whose name starts with 'spring.integration'. |
|spring.cloud.stream.metrics.properties | |Application properties that should be added to the metrics payload For example:  `spring.application**`  |
|spring.cloud.stream.metrics.schedule-interval |60s |Interval expressed as Duration for scheduling metrics snapshots publishing. Defaults to 60 seconds |
|spring.cloud.stream.rabbit.binder.admin-addresses |[] |Urls for management plugins; only needed for queue affinity. |
|spring.cloud.stream.rabbit.binder.admin-adresses | | |
|spring.cloud.stream.rabbit.binder.compression-level |0 |Compression level for compressed bindings; see 'java.util.zip.Deflator'. |
|spring.cloud.stream.rabbit.binder.connection-name-prefix | |Prefix for connection names from this binder. |
|spring.cloud.stream.rabbit.binder.nodes |[] |Cluster member node names; only needed for queue affinity. |
|spring.cloud.stream.rabbit.bindings | | |
|spring.cloud.vault.app-id.app-id-path |app-id |Mount path of the AppId authentication backend. |
|spring.cloud.vault.app-id.network-interface | |Network interface hint for the "MAC_ADDRESS" UserId mechanism. |
|spring.cloud.vault.app-id.user-id |MAC_ADDRESS |UserId mechanism. Can be either "MAC_ADDRESS", "IP_ADDRESS", a string or a class name. |
|spring.cloud.vault.app-role.app-role-path |approle |Mount path of the AppRole authentication backend. |
|spring.cloud.vault.app-role.role | |Name of the role, optional, used for pull-mode. |
|spring.cloud.vault.app-role.role-id | |The RoleId. |
|spring.cloud.vault.app-role.secret-id | |The SecretId. |
|spring.cloud.vault.application-name |application |Application name for AppId authentication. |
|spring.cloud.vault.authentication | | |
|spring.cloud.vault.aws-ec2.aws-ec2-path |aws-ec2 |Mount path of the AWS-EC2 authentication backend. |
|spring.cloud.vault.aws-ec2.identity-document |[http://169.254.169.254/latest/dynamic/instance-identity/pkcs7](http://169.254.169.254/latest/dynamic/instance-identity/pkcs7) |URL of the AWS-EC2 PKCS7 identity document. |
|spring.cloud.vault.aws-ec2.nonce | |Nonce used for AWS-EC2 authentication. An empty nonce defaults to nonce generation. |
|spring.cloud.vault.aws-ec2.role | |Name of the role, optional. |
|spring.cloud.vault.aws-iam.aws-path |aws |Mount path of the AWS authentication backend. |
|spring.cloud.vault.aws-iam.role | |Name of the role, optional. Defaults to the friendly IAM name if not set. |
|spring.cloud.vault.aws-iam.server-name | |Name of the server used to set {@code X-Vault-AWS-IAM-Server-ID} header in the headers of login requests. |
|spring.cloud.vault.aws.access-key-property |cloud.aws.credentials.accessKey |Target property for the obtained access key. |
|spring.cloud.vault.aws.backend |aws |aws backend path. |
|spring.cloud.vault.aws.enabled |false |Enable aws backend usage. |
|spring.cloud.vault.aws.role | |Role name for credentials. |
|spring.cloud.vault.aws.secret-key-property |cloud.aws.credentials.secretKey |Target property for the obtained secret key. |
|spring.cloud.vault.cassandra.backend |cassandra |Cassandra backend path. |
|spring.cloud.vault.cassandra.enabled |false |Enable cassandra backend usage. |
|spring.cloud.vault.cassandra.password-property |spring.data.cassandra.password |Target property for the obtained password. |
|spring.cloud.vault.cassandra.role | |Role name for credentials. |
|spring.cloud.vault.cassandra.username-property |spring.data.cassandra.username |Target property for the obtained username. |
|spring.cloud.vault.config.lifecycle.enabled |true |Enable lifecycle management. |
|spring.cloud.vault.config.order |0 |Used to set a {@link org.springframework.core.env.PropertySource} priority. This is useful to use Vault as an override on other property sources. @see org.springframework.core.PriorityOrdered |
|spring.cloud.vault.connection-timeout |5000 |Connection timeout; |
|spring.cloud.vault.consul.backend |consul |Consul backend path. |
|spring.cloud.vault.consul.enabled |false |Enable consul backend usage. |
|spring.cloud.vault.consul.role | |Role name for credentials. |
|spring.cloud.vault.consul.token-property |spring.cloud.consul.token |Target property for the obtained token. |
|spring.cloud.vault.database.backend |database |Database backend path. |
|spring.cloud.vault.database.enabled |false |Enable database backend usage. |
|spring.cloud.vault.database.password-property |spring.datasource.password |Target property for the obtained password. |
|spring.cloud.vault.database.role | |Role name for credentials. |
|spring.cloud.vault.database.username-property |spring.datasource.username |Target property for the obtained username. |
|spring.cloud.vault.discovery.enabled |false |Flag to indicate that Vault server discovery is enabled (vault server URL will be looked up via discovery). |
|spring.cloud.vault.discovery.service-id |vault |Service id to locate Vault. |
|spring.cloud.vault.enabled |true |Enable Vault config server. |
|spring.cloud.vault.fail-fast |false |Fail fast if data cannot be obtained from Vault. |
|spring.cloud.vault.generic.application-name |application |Application name to be used for the context. |
|spring.cloud.vault.generic.backend |secret |Name of the default backend. |
|spring.cloud.vault.generic.default-context |application |Name of the default context. |
|spring.cloud.vault.generic.enabled |true |Enable the generic backend. |
|spring.cloud.vault.generic.profile-separator |/ |Profile-separator to combine application name and profile. |
|spring.cloud.vault.host |localhost |Vault server host. |
|spring.cloud.vault.kubernetes.kubernetes-path |kubernetes |Mount path of the Kubernetes authentication backend. |
|spring.cloud.vault.kubernetes.role | |Name of the role against which the login is being attempted. |
|spring.cloud.vault.kubernetes.service-account-token-file |/var/run/secrets/kubernetes.io/serviceaccount/token |Path to the service account token file. |
|spring.cloud.vault.kv.application-name |application |Application name to be used for the context. |
|spring.cloud.vault.kv.backend |secret |Name of the default backend. |
|spring.cloud.vault.kv.backend-version |2 |Key-Value backend version. Currently supported versions are: <ul> <li>Version 1 (unversioned key-value backend).</li> <li>Version 2 (versioned key-value backend).</li> </ul> |
|spring.cloud.vault.kv.default-context |application |Name of the default context. |
|spring.cloud.vault.kv.enabled |false |Enable the kev-value backend. |
|spring.cloud.vault.kv.profile-separator |/ |Profile-separator to combine application name and profile. |
|spring.cloud.vault.mongodb.backend |mongodb |Cassandra backend path. |
|spring.cloud.vault.mongodb.enabled |false |Enable mongodb backend usage. |
|spring.cloud.vault.mongodb.password-property |spring.data.mongodb.password |Target property for the obtained password. |
|spring.cloud.vault.mongodb.role | |Role name for credentials. |
|spring.cloud.vault.mongodb.username-property |spring.data.mongodb.username |Target property for the obtained username. |
|spring.cloud.vault.mysql.backend |mysql |mysql backend path. |
|spring.cloud.vault.mysql.enabled |false |Enable mysql backend usage. |
|spring.cloud.vault.mysql.password-property |spring.datasource.password |Target property for the obtained username. |
|spring.cloud.vault.mysql.role | |Role name for credentials. |
|spring.cloud.vault.mysql.username-property |spring.datasource.username |Target property for the obtained username. |
|spring.cloud.vault.port |8200 |Vault server port. |
|spring.cloud.vault.postgresql.backend |postgresql |postgresql backend path. |
|spring.cloud.vault.postgresql.enabled |false |Enable postgresql backend usage. |
|spring.cloud.vault.postgresql.password-property |spring.datasource.password |Target property for the obtained username. |
|spring.cloud.vault.postgresql.role | |Role name for credentials. |
|spring.cloud.vault.postgresql.username-property |spring.datasource.username |Target property for the obtained username. |
|spring.cloud.vault.rabbitmq.backend |rabbitmq |rabbitmq backend path. |
|spring.cloud.vault.rabbitmq.enabled |false |Enable rabbitmq backend usage. |
|spring.cloud.vault.rabbitmq.password-property |spring.rabbitmq.password |Target property for the obtained password. |
|spring.cloud.vault.rabbitmq.role | |Role name for credentials. |
|spring.cloud.vault.rabbitmq.username-property |spring.rabbitmq.username |Target property for the obtained username. |
|spring.cloud.vault.read-timeout |15000 |Read timeout; |
|spring.cloud.vault.scheme |https |Protocol scheme. Can be either "http" or "https". |
|spring.cloud.vault.ssl.cert-auth-path |cert |Mount path of the TLS cert authentication backend. |
|spring.cloud.vault.ssl.key-store | |Trust store that holds certificates and private keys. |
|spring.cloud.vault.ssl.key-store-password | |Password used to access the key store. |
|spring.cloud.vault.ssl.trust-store | |Trust store that holds SSL certificates. |
|spring.cloud.vault.ssl.trust-store-password | |Password used to access the trust store. |
|spring.cloud.vault.token | |Static vault token. Required if {@link #authentication} is {@code TOKEN}. |
|spring.cloud.vault.uri | |Vault URI. Can be set with scheme, host and port. |
|spring.cloud.zookeeper.base-sleep-time-ms |50 |Initial amount of time to wait between retries |
|spring.cloud.zookeeper.block-until-connected-unit | |The unit of time related to blocking on connection to Zookeeper |
|spring.cloud.zookeeper.block-until-connected-wait |10 |Wait time to block on connection to Zookeeper |
|spring.cloud.zookeeper.connect-string |localhost:2181 |Connection string to the Zookeeper cluster |
|spring.cloud.zookeeper.default-health-endpoint | |Default health endpoint that will be checked to verify that a dependency is alive |
|spring.cloud.zookeeper.dependencies | |Mapping of alias to ZookeeperDependency. From Ribbon perspective the alias is actually serviceID since Ribbon can’t accept nested structures in serviceID |
|spring.cloud.zookeeper.dependency-configurations | | |
|spring.cloud.zookeeper.dependency-names | | |
|spring.cloud.zookeeper.discovery.enabled |true | |
|spring.cloud.zookeeper.discovery.initial-status | |The initial status of this instance (defaults to {@link StatusConstants#STATUS_UP}). |
|spring.cloud.zookeeper.discovery.instance-host | |Predefined host with which a service can register itself in Zookeeper. Corresponds to the {code address} from the URI spec. |
|spring.cloud.zookeeper.discovery.instance-id | |Id used to register with zookeeper. Defaults to a random UUID. |
|spring.cloud.zookeeper.discovery.instance-port | |Port to register the service under (defaults to listening port) |
|spring.cloud.zookeeper.discovery.instance-ssl-port | |Ssl port of the registered service. |
|spring.cloud.zookeeper.discovery.metadata | |Gets the metadata name/value pairs associated with this instance. This information is sent to zookeeper and can be used by other instances. |
|spring.cloud.zookeeper.discovery.register |true |Register as a service in zookeeper. |
|spring.cloud.zookeeper.discovery.root |/services |Root Zookeeper folder in which all instances are registered |
|spring.cloud.zookeeper.discovery.uri-spec |{scheme}://{address}:{port} |The URI specification to resolve during service registration in Zookeeper |
|spring.cloud.zookeeper.enabled |true |Is Zookeeper enabled |
|spring.cloud.zookeeper.max-retries |10 |Max number of times to retry |
|spring.cloud.zookeeper.max-sleep-ms |500 |Max time in ms to sleep on each retry |
|spring.cloud.zookeeper.prefix | |Common prefix that will be applied to all Zookeeper dependencies' paths |
|spring.integration.poller.fixed-delay |1000 |Fixed delay for default poller. |
|spring.integration.poller.max-messages-per-poll |1 |Maximum messages per poll for the default poller. |
|spring.sleuth.annotation.enabled |true | |
|spring.sleuth.async.configurer.enabled |true |Enable default AsyncConfigurer. |
|spring.sleuth.async.enabled |true |Enable instrumenting async related components so that the tracing information is passed between threads. |
|spring.sleuth.baggage-keys | |List of baggage key names that should be propagated out of process. These keys will be prefixed with  `baggage`  before the actual key. This property is set in order to be backward compatible with previous Sleuth versions. @see brave.propagation.ExtraFieldPropagation.FactoryBuilder#addPrefixedFields(String, java.util.Collection) |
|spring.sleuth.enabled |true | |
|spring.sleuth.feign.enabled |true |Enable span information propagation when using Feign. |
|spring.sleuth.feign.processor.enabled |true |Enable post processor that wraps Feign Context in its tracing representations. |
|spring.sleuth.http.enabled |true | |
|spring.sleuth.http.legacy.enabled |false | |
|spring.sleuth.hystrix.strategy.enabled |true |Enable custom HystrixConcurrencyStrategy that wraps all Callable instances into their Sleuth representative - the TraceCallable. |
|spring.sleuth.integration.enabled |true |Enable Spring Integration sleuth instrumentation. |
|spring.sleuth.integration.patterns |[!hystrixStreamOutput*, *] |An array of patterns against which channel names will be matched. @see org.springframework.integration.config.GlobalChannelInterceptor#patterns(). Defaults to any channel name not matching the Hystrix Stream channel name. |
|spring.sleuth.integration.websockets.enabled |true |Enable tracing for WebSockets. |
|spring.sleuth.keys.http.headers | |Additional headers that should be added as tags if they exist. If the header value is multi-valued, the tag value will be a comma-separated, single-quoted list. |
|spring.sleuth.keys.http.prefix |http. |Prefix for header names if they are added as tags. |
|spring.sleuth.log.slf4j.enabled |true |Enable a {@link Slf4jCurrentTraceContext} that prints tracing information in the logs. |
|spring.sleuth.messaging.enabled |false | |
|spring.sleuth.messaging.kafka.enabled |false | |
|spring.sleuth.messaging.kafka.remote-service-name |kafka | |
|spring.sleuth.messaging.rabbit.enabled |false | |
|spring.sleuth.messaging.rabbit.remote-service-name |rabbitmq | |
|spring.sleuth.opentracing.enabled |true | |
|spring.sleuth.propagation-keys | |List of fields that are referenced the same in-process as it is on the wire. For example, the name "x-vcap-request-id" would be set as-is including the prefix. <p>Note: {@code fieldName} will be implicitly lower-cased. @see brave.propagation.ExtraFieldPropagation.FactoryBuilder#addField(String) |
|spring.sleuth.reactor.enabled.enabled |true |When true enables instrumentation for reactor |
|spring.sleuth.rxjava.schedulers.hook.enabled |true |Enable support for RxJava via RxJavaSchedulersHook. |
|spring.sleuth.rxjava.schedulers.ignoredthreads |[HystrixMetricPoller, ^RxComputation.*$] |Thread names for which spans will not be sampled. |
|spring.sleuth.sampler.probability |0.1 |Probability of requests that should be sampled. E.g. 1.0 - 100% requests should be sampled. The precision is whole-numbers only (i.e. there’s no support for 0.1% of the traces). |
|spring.sleuth.scheduled.enabled |true |Enable tracing for {@link org.springframework.scheduling.annotation.Scheduled}. |
|spring.sleuth.scheduled.skip-pattern |org.springframework.cloud.netflix.hystrix.stream.HystrixStreamTask |Pattern for the fully qualified name of a class that should be skipped. |
|spring.sleuth.supports-join |true |True means the tracing system supports sharing a span ID between a client and server. |
|spring.sleuth.trace-id128 |false |When true, generate 128-bit trace IDs instead of 64-bit ones. |
|spring.sleuth.web.additional-skip-pattern | |Additional pattern for URLs that should be skipped in tracing. This will be appended to the {@link SleuthWebProperties#skipPattern} |
|spring.sleuth.web.client.enabled |true |Enable interceptor injecting into {@link org.springframework.web.client.RestTemplate} |
|spring.sleuth.web.enabled |true |When true enables instrumentation for web applications |
|spring.sleuth.web.exception-throwing-filter-enabled |true |Flag to toggle the presence of a filter that logs thrown exceptions |
|spring.sleuth.web.filter-order | |Order in which the tracing filters should be registered. Defaults to {@link TraceHttpAutoConfiguration#TRACING_FILTER_ORDER} |
|spring.sleuth.web.skip-pattern |/api-docs.* |/autoconfig |
|/configprops |/dump |/health |
|/info |/metrics.* |/mappings |
|/trace |/swagger.* |.*\.png |
|.*\.css |.*\.js |.*\.html |
|/favicon.ico |/hystrix.stream |/application/.* |
|/actuator.* |/cloudfoundryapplication |Pattern for URLs that should be skipped in tracing |
|spring.sleuth.zuul.enabled |true |Enable span information propagation when using Zuul. |
|stubrunner.amqp.enabled |false |Whether to enable support for Stub Runner and AMQP. |
|stubrunner.amqp.mockCOnnection |true |Whether to enable support for Stub Runner and AMQP mocked connection factory. |
|stubrunner.classifier |stubs |The classifier to use by default in ivy co-ordinates for a stub. |
|stubrunner.cloud.consul.enabled |true |Whether to enable stubs registration in Consul. |
|stubrunner.cloud.delegate.enabled |true |Whether to enable DiscoveryClient’s Stub Runner implementation. |
|stubrunner.cloud.enabled |true |Whether to enable Spring Cloud support for Stub Runner. |
|stubrunner.cloud.eureka.enabled |true |Whether to enable stubs registration in Eureka. |
|stubrunner.cloud.ribbon.enabled |true |Whether to enable Stub Runner’s Ribbon integration. |
|stubrunner.cloud.stubbed.discovery.enabled |true |Whether Service Discovery should be stubbed for Stub Runner. If set to false, stubs will get registered in real service discovery. |
|stubrunner.cloud.zookeeper.enabled |true |Whether to enable stubs registration in Zookeeper. |
|stubrunner.consumer-name | |You can override the default {@code spring.application.name} of this field by setting a value to this parameter. |
|stubrunner.delete-stubs-after-test |true |If set to {@code false} will NOT delete stubs from a temporary folder after running tests |
|stubrunner.ids |[] |The ids of the stubs to run in "ivy" notation ([groupId]:artifactId:[version]:[classifier][:port]). {@code groupId}, {@code classifier}, {@code version} and {@code port} can be optional. |
|stubrunner.ids-to-service-ids | |Mapping of Ivy notation based ids to serviceIds inside your application Example "a:b" → "myService" "artifactId" → "myOtherService" |
|stubrunner.integration.enabled |true |Whether to enable Stub Runner integration with Spring Integration. |
|stubrunner.mappings-output-folder | |Dumps the mappings of each HTTP server to the selected folder |
|stubrunner.max-port |15000 |Max value of a port for the automatically started WireMock server |
|stubrunner.min-port |10000 |Min value of a port for the automatically started WireMock server |
|stubrunner.password | |Repository password |
|stubrunner.properties | |Map of properties that can be passed to custom {@link org.springframework.cloud.contract.stubrunner.StubDownloaderBuilder} |
|stubrunner.proxy-host | |Repository proxy host |
|stubrunner.proxy-port | |Repository proxy port |
|stubrunner.snapshot-check-skip |false |If set to {@code true} will not assert whether the downloaded stubs / contract JAR was downloaded from a remote location or a local one(only applicable to Maven repos, not Git or Pact) |
|stubrunner.stream.enabled |true |Whether to enable Stub Runner integration with Spring Cloud Stream. |
|stubrunner.stubs-mode | |Pick where the stubs should come from |
|stubrunner.stubs-per-consumer |false |Should only stubs for this particular consumer get registered in HTTP server stub. |
|stubrunner.username | |Repository username |


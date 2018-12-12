# Part XVIII.附录：配置属性简编

|名称|设为首页|简介|
| ---- | ---- | ---- |
| encrypt.fail-on-error | true |标记如果存在加密或解密错误，进程应该失败. |
| encrypt.key | |对称密钥.作为更强的替代方案，请考虑使用密钥库. |
| encrypt.key-store.alias | |商店中密钥的别名. |
| encrypt.key-store.location | |密钥库文件的位置，例如类路径：/keystore.jks. |
| encrypt.key-store.password | |用于锁定密钥库的密码. |
| encrypt.key-store.secret | |密钥保护密钥（默认与密码相同）. |
| encrypt.rsa.algorithm | |要使用的RSA算法（DEFAULT或OEAP）.一旦设置，不要改变它（或现有的密码不会解密）. |
| encrypt.rsa.salt | deadbeef |用于加密密文的随机密码的salt.一旦设置，不要改变它（或现有的密码不会解密）. |
| encrypt.rsa.strong | false |标记表示应在内部使用“强”AES加密.如果为true，则GCM算法应用于AES加密字节.默认值为false（在这种情况下，使用“标准”CBC代替）.一旦设置，不要改变它（或现有的密码不会解密）. |
| encrypt.salt | deadbeef |十六进制编码字节数组形式的对称密钥的salt.作为更强的替代方案，请考虑使用密钥库. |
| endpoints.zookeeper.enabled | true |启用/ zookeeperendpoints以检查zookeeper的状态. |
| eureka.client.allow-redirects | false |指示服务器是否可以将客户端请求重定向到备份服务器/群集.如果设置为false，则服务器将直接处理请求，如果设置为true，则可以使用新的服务器位置将HTTP重定向发送到客户端. |
| eureka.client.availability-zones | |获取此实例所在区域的可用区域列表（在AWS数据中心中使用）.这些更改在运行时在registryFetchIntervalSeconds指定的下一个注册表获取周期中有效. |
| eureka.client.backup-registry-impl | |获取实现BackupRegistry的实现的名称，该实现仅在eureka客户端启动时第一次作为后备选项获取注册表信息.对于需要额外的注册表信息弹性的应用程序，可能需要这样做，否则它将无法运行. |
| eureka.client.cache-refresh-executor-exponential-back-off-bound | 10 |高速缓存刷新Actuator指数退避相关属性.在发生一系列超时的情况下，它是重试延迟的最大乘数值. |
| eureka.client.cache-refresh-executor-thread-pool-size | 2 | cacheRefreshExecutor初始化的线程池大小|
| eureka.client.client-data-accept | | Eureka接受客户数据接受的名称|
| eureka.client.decoder-name | |这是一个瞬态配置和一次最新的编解码器是稳定的，可以删除（因为只有一个）|
| eureka.client.disable-delta | false |指示eureka客户端是否应禁用delta的提取，而应该使用获取完整的注册表信息.请注意，增量提取可以极大地减少流量，因为使用eureka服务器的更改速率通常远低于提取速率.这些更改在下一个注册表获取周期的运行时生效，如registryFetchIntervalSeconds |所指定
| eureka.client.dollar-replacement | _- |在eureka服务器中序列化/反序列化信息期间获取Dollar sign <code> $ </ code>的替换字符串. |
| eureka.client.enabled | true |标记表示启用了Eureka客户端. |
| eureka.client.encoder-name | |这是一个瞬态配置，一旦最新的编解码器稳定，就可以删除（因为只有一个）|
| eureka.client.escape-char-replacement | __ |在eureka服务器中序列化/反序列化信息期间获取下划线符号<code> _ </ code>的替换字符串. |
| eureka.client.eureka-connection-idle-timeout-seconds | 30 |表示与eureka服务器的HTTP连接在关闭之前可以保持空闲的时间（以秒为单位）.在AWS环境中，建议值为30秒或更短，因为防火墙会在几分钟后清除连接信息，使连接挂起.
| eureka.client.eureka-server-connect-timeout-seconds | 5 |表示连接到eureka服务器需要超时之前等待的时间（以秒为单位）.请注意，客户端中的连接由org.apache.http.client.HttpClient池化，此设置会影响实际的连接创建以及从池中获取连接的等待时间. |
| eureka.client.eureka-server-d-n-s-name | |获取要查询以获取eureka服务器列表的DNS名称.如果Contract通过实现serviceUrls返回服务URL，则不需要此信息.当useDnsForFetchingServiceUrls设置为true并且eureka客户端期望DNS以某种方式配置以便它可以动态地获取更改的eureka服务器时，使用DNS机制.这些更改在运行时有效. |
| eureka.client.eureka-server-port | |当eureka服务器列表来自DNS时，获取用于构造服务URL以联系eureka服务器的端口.如果Contract返回服务URL eurekaServerServiceUrls（String），则不需要此信息.当useDnsForFetchingServiceUrls设置为true并且eureka客户端期望DNS以某种方式配置以便它可以动态地获取更改的eureka服务器时，使用DNS机制.这些更改在运行时有效. |
| eureka.client.eureka-server-read-timeout-seconds | 8 |表示从eureka服务器读取超时之前等待的时间（以秒为单位）. |
| eureka.client.eureka-server-total-connections | 200 |获取从eureka客户端到所有eureka服务器的连接总数. |
| eureka.client.eureka-server-total-connections-per-host | 50 |获取从eureka客户端到eureka服务器主机的连接总数. |
| eureka.client.eureka-server-u-r-l-context | |获取当eureka服务器列表来自DNS时，用于构建服务URL以联系eureka服务器的URL上下文.如果Contract从eurekaServerServiceUrls返回服务URL，则不需要此信息.当useDnsForFetchingServiceUrls设置为true并且eureka客户端期望DNS以某种方式配置以便它可以动态地获取更改的eureka服务器时，使用DNS机制.这些更改在运行时有效. |
| eureka.client.eureka-service-url-poll-interval-seconds | 0 |表示轮询更改eureka服务器信息的频率（以秒为单位）.可以添加或删除Eureka服务器，此设置控制eureka客户应该多久了解它. |
| eureka.client.fetch-registry | true |指示此客户端是否应从eureka服务器获取eureka注册表信息. |
| eureka.client.fetch-remote-regions-registry | |以逗号分隔的区域列表，将为其提取eureka注册表信息.必须为availabilityZones返回的每个区域定义可用区域.如果不这样做，将导致发现客户端启动失败. |
| eureka.client.filter-only-up-instances | true |指示是否在仅为InstanceStatus UP状态过滤应用程序后获取应用程序. |
| eureka.client.g-zip-content | true |指示只要服务器支持从eureka服务器获取的内容，就必须对其进行压缩.来自eureka服务器的注册表信息被压缩以获得最佳网络流量. |
| eureka.client.healthcheck.enabled | true |启用Eureka运行状况检查处理程序.|
| eureka.client.heartbeat-executor-exponential-back-off-bound | 10 |心跳Actuator指数退避相关属性.在发生一系列超时的情况下，它是重试延迟的最大乘数值. |
| eureka.client.heartbeat-executor-thread-pool-size | 2 | heartbeatExecutor初始化的线程池大小|
| eureka.client.initial-instance-info-replication-interval-seconds | 40 |表示将实例信息复制到eureka服务器的最初时间（以秒为单位）|
| eureka.client.instance-info-replication-interval-seconds | 30 |表示复制要复制到eureka服务器的实例更改的频率（以秒为单位）. |
| eureka.client.log-delta-diff | false |指示是否在注册表信息方面记录eureka服务器和eureka客户端之间的差异. Eureka客户端尝试仅从eureka服务器检索增量更改，以最大限度地减少网络流量.在收到增量后，eureka客户端会协调来自服务器的信息，以确认它没有遗漏一些信息.当客户端遇到与服务器通信的网络问题时，可能会发生协调失败.如果协调失败，eureka客户端将获取完整的注册表信息.在获取完整的注册表信息时，eureka客户端可以记录客户端和服务器之间的差异，此设置可以控制它.这些更改在registryFetchIntervalSecondsr |指定的下一个注册表获取周期的运行时有效
| eureka.client.on-demand-update-status-change | true |如果设置为true，则通过ApplicationInfoManager进行本地状态更新将触发对远程eureka服务器的按需（但速率受限）注册/更新
| eureka.client.prefer-same-zone-eureka | true |指示此实例是否应尝试在同一区域中使用eureka服务器以获取延迟和/或其他原因.理想情况下，eureka客户端配置为与同一区域中的服务器通信.这些更改在下一个注册表获取周期的运行时有效，如registryFetchIntervalSeconds |
| eureka.client.property-resolver | | |
| eureka.client.proxy-host | |获取eureka服务器的代理主机（如果有）. |
| eureka.client.proxy-password | |获取代理密码（如果有）. |
| eureka.client.proxy-port | |获取eureka服务器的代理端口（如果有）. |
| eureka.client.proxy-user-name | |获取代理用户名（如果有）. |
| eureka.client.region | us-east-1 |获取此实例所在的区域（在AWS数据中心中使用）. |
| eureka.client.register-with-eureka | true |指示此实例是否应将其信息注册到eureka服务器以供其他人发现.在某些情况下，您不希望发现您的实例，而您只是想要发现其他实例. |
| eureka.client.registry-fetch-interval-seconds | 30 |表示从eureka服务器获取注册表信息的频率（以秒为单位）. |
| eureka.client.registry-refresh-single-vip-address | |表示客户端是否仅对单个VIP的注册表信息感兴趣. |
| eureka.client.service-url | |可用区域映射到与eureka服务器通信的完全限定URL列表.每个值可以是单个URL或逗号分隔的备用位置列表.通常，eureka服务器URL携带协议，主机，端口，上下文和版本信息（如果有）.示例：[http://ec2-256-156-243-129.compute-1.amazonaws.com:7001/eureka/](http://ec2-256-156-243-129.compute-1.amazonaws.com:7001/eureka/)更改在运行时在eurekaServiceUrlPollIntervalSeconds指定的下一个服务URL刷新周期生效. |
| eureka.client.should-enforce-registration-at-init | false |指示客户端是否应在初始化期间强制注册.默认为false. |
| eureka.client.should-unregister-on-shutdown | true |指示客户端是否应在客户端关闭时从远程服务器显式取消注册. |
| eureka.client.use-dns-for-fetching-service-urls | false |指示eureka客户端是否应使用DNS机制来获取要与之通信的eureka服务器列表.当DNS名称更新为具有其他服务器时，eureka客户端轮询eurekaServiceUrlPollIntervalSeconds中指定的信息后立即使用该信息.或者，服务URL可以返回serviceUrls，但是用户应该实现自己的机制，以便在发生更改时返回更新的列表.这些更改在运行时有效. |
| eureka.dashboard.enabled | true |标记以启用Eureka仪表板.默认为true. |
| eureka.dashboard.path | / | Eureka仪表板的路径（相对于servlet路径）.默认为“/”. |
| eureka.instance.a-s-g-name | |获取与此实例关联的AWS自动缩放组名称.此信息专门用于AWS环境，以在实例启动后自动将实例停止服务，并且已禁用该流量.
| eureka.instance.app-group-name | |获取要在eureka注册的应用程序组的名称.|
| eureka.instance.appname | unknown |获取要在eureka注册的应用程序的名称. |
| eureka.instance.data-center-info | |返回部署此实例的数据中心.如果在AWS中部署实例，则此信息用于获取某些AWS特定实例信息. |
| eureka.instance.default-address-resolution-order | [] | |
| eureka.instance.environment | | |
| eureka.instance.health-check-url | |获取此实例的绝对运行状况检查页面URL.如果运行状况检查页面位于与eureka通信的同一实例中，则用户可以提供healthCheckUrlPath，否则，在实例是某个其他服务器的代理的情况下，用户可以提供完整的URL.如果提供完整的URL，则优先. <p>它通常用于根据实例的运行状况做出有根据的决策 - 例如，它可用于确定是继续部署到整个服务器场还是停止部署而不会造成进一步的损害.完整URL应遵循格式[http://${eureka.hostname}:7001/](http://${eureka.hostname}:7001/)，其中值$ {eureka.hostname}在运行时被替换. |
| eureka.instance.health-check-url-path | |获取此实例的相对运行状况检查URL路径.然后，Health检查页面URL由主机名和通信类型构成 - 安全或不安全，如securePort和nonSecurePort中所指定.它通常用于根据实例的运行状况做出有根据的决策 - 例如，它可用于确定是否继续部署到整个服务器场或停止部署而不会造成进一步的损害. |
| eureka.instance.home-page-url | |获取此实例的绝对主页URL.如果主页位于与eureka交谈的同一实例中，则用户可以提供homePageUrlPath，否则在实例是某些其他服务器的代理的情况下，用户可以提供完整的URL.如果提供完整的URL，则优先.它通常用于其他服务的信息目的，以将其用作登录页面.完整URL应遵循格式[http://${eureka.hostname}:7001/](http://${eureka.hostname}:7001/)，其中值$ {eureka.hostname}在运行时被替换. |
| eureka.instance.home-page-url-path | / |获取此实例的相对主页URL路径.然后，主机URL由hostName和通信类型构成 - 安全或不安全.它通常用于其他服务的信息目的，以将其用作登录页面. |
| eureka.instance.hostname | |如果可以在配置时确定主机名（否则将从OS原语中猜出）. |
| eureka.instance.initial-status | |注册rmeote Eureka服务器的初始状态. |
| eureka.instance.instance-enabled-onit | false |指示是否应该在eureka注册后立即启用实例以获取流量.有时，应用程序可能需要在准备好接收流量之前进行一些预处理. |
| eureka.instance.instance-id | |获取要在eureka中注册的此实例的唯一ID（在appName范围内）. |
| eureka.instance.ip-address | |获取实例的IP地址.此信息仅用于学术目的，因为来自其他实例的通信主要使用{@link #getHostName（boolean）}中提供的信息进行. |
| eureka.instance.lease-expiration-duration-in-seconds | 90 |表示eureka服务器在收到最后一次心跳之前等待的时间（以秒为单位），然后才能从该视图中删除此实例，并禁止此实例的流量.将此值设置得太长可能意味着即使实例不活动，流量也可以路由到实例.将此值设置得太小可能意味着，由于临时网络故障，实例可能会被取消流量.此值设置为至少高于leaseRenewalIntervalInSeconds中指定的值. |
| eureka.instance.lease-renewal-interval-in-seconds | 30 |表示eureka客户端向Eureka服务器发送心跳以指示其仍处于活动状态所需的频率（以秒为单位）.如果在leaseExpirationDurationInSeconds中指定的时间段内未收到心跳，则eureka服务器将从其视图中删除该实例，此处不允许此实例的流量.请注意，如果实例实现HealthCheckCallback，然后决定使其自身不可用，则实例仍然无法获取流量. |
| eureka.instance.metadata-map | |获取与此实例关联的元数据名称/值对.此信息将发送到eureka服务器，并可供其他实例使用. |
| eureka.instance.namespace | eureka |获取用于查找属性的命名空间.在Spring Cloud中被忽略. |
| eureka.instance.non-secure-port | 80 |获取实例应接收流量的非安全端口. |
| eureka.instance.non-secure-port-enabled | true |指示是否应为流量启用非安全端口. |
| eureka.instance.prefer-ip-address | false|标志着，在猜测主机名时，应该使用服务器的IP地址来引用操作系统报告的主机名. |
| eureka.instance.registry.default-open-for-traffic-count | 1 |用于确定何时取消租约的值，默认为1表示独立.对于同行复制的eurekas，应该设置为0
| eureka.instance.registry.expected-of-renews-per-min | 1 | |
| eureka.instance.secure-health-check-url | |获取此实例的绝对安全运行状况检查页面URL.如果运行状况检查页面位于与eureka通信的同一实例中，则用户可以提供secureHealthCheckUrl，否则，在实例是某些其他服务器的代理的情况下，用户可以提供完整的URL.如果提供完整的URL，则优先. <p>它通常用于根据实例的运行状况做出有根据的决策 - 例如，它可用于确定是继续部署到整个服务器场还是停止部署而不会造成进一步的损害.完整URL应遵循格式[http://${eureka.hostname}:7001/](http://${eureka.hostname}:7001/)，其中值$ {eureka.hostname}在运行时被替换. |
| eureka.instance.secure-port | 443 |获取实例应接收流量的安全端口. |
| eureka.instance.secure-port-enabled | false |指示是否应为流量启用安全端口. |
| eureka.instance.secure-virtual-host-name | unknown |获取为此实例定义的安全虚拟主机名.这通常是其他实例通过使用安全虚拟主机名来查找此实例的方式.这与完全限定的域名类似，您的服务用户需要查找此实例. |
| eureka.instance.status-page-url | |获取此实例的绝对状态页面URL路径.如果状态页面驻留在与eureka交谈的同一实例中，则用户可以提供statusPageUrlPath，否则，在实例是某个其他服务器的代理的情况下，用户可以提供完整的URL.如果提供完整的URL，则优先.它通常用于其他服务的信息目的，以查找此实例的状态.用户可以提供一个简单的HTML，指示实例的当前状态. |
| eureka.instance.status-page-url-path | |获取此实例的相对状态页面URL路径.然后，根据hostName和通信类型构建状态页面URL  - 安全或不安全，如securePort和nonSecurePort中所指定.它通常用于其他服务的信息目的，以查找此实例的状态.用户可以提供一个简单的HTML，指示实例的当前状态. |
| eureka.instance.virtual-host-name | unknown |获取为此实例定义的虚拟主机名.这通常是其他实例通过使用虚拟主机名来查找此实例的方式.这与完全限定的域名类似，您的服务用户需要找到此实例. |
| eureka.server.a-s-g-cache-expiry-timeout-ms | 0 | |
| eureka.server.a-s-g-query-timeout-ms | 300 | |
| eureka.server.a-s-g-update-interval-ms | 0 | |
| eureka.server.a-w-s-access-id | | |
| eureka.server.a-w-s-secret-key | | |
| eureka.server.batch-replication | false | |
| eureka.server.binding-strategy | | |
| eureka.server.delta-retention-timer-interval-in-ms | 0 | |
| eureka.server.disable-delta | false | |
| eureka.server.disable-delta-for-remote-regions | false | |
| eureka.server.disable-transparent-fallback-to-other-region | false | |
| eureka.server.e-i-p-bind-rebind-retries | 3 | |
| eureka.server.e-i-p-binding-retry-interval-ms | 0 | |
| eureka.server.e-i-p-binding-retry-interval-ms-when-unbound | 0 | |
| eureka.server.enable-replicated-request-compression | false | |
| eureka.server.enable-self-preservation | true | |
| eureka.server.eviction-interval-timer-in-ms | 0 | |
| eureka.server.g-zip-content-from-remote-region | true | |
| eureka.server.json-codec-name | | |
| eureka.server.list-auto-scaling-groups-role-name | ListAutoScalingGroups | |
| eureka.server.log-identity-headers | true | |
| eureka.server.max-elements-in-peer-replication-pool | 10000 | |
| eureka.server.max-elements-in-status-replication-pool | 10000 | |
| eureka.server.max-idle-thread-age-in-minutes-for-peer-replication | 15 | |
| eureka.server.max-idle-thread-in-minutes-age-for-status-replication | 10 | |
| eureka.server.max-threads-for-peer-replication | 20 | |
| eureka.server.max-threads-for-status-replication | 1 | |
| eureka.server.max-复制时间| 30000 | |
| eureka.server.min-available-instances-for-peer-replication | -1 | |
| eureka.server.min-threads-for-peer-replication | 5 | |
| eureka.server.min-threads-for-status-replication | 1 | |
| eureka.server.number-of-replication-retries | 5 | |
| eureka.server.peer-eureka-nodes-update-interval-ms | 0 | |
| eureka.server.peer-eureka-status-refresh-time-interval-ms | 0 | |
| eureka.server.peer-node-connect-timeout-ms | 200 ||
| eureka.server.peer-node-connection-idle-timeout-seconds | 30 | |
| eureka.server.peer-node-read-timeout-ms | 200 | |
| eureka.server.peer-node-total-connections | 1000 | |
| eureka.server.peer-node-total-connections-per-host | 500 | |
| eureka.server.prime-aws-replica-connections | true | |
| eureka.server.property-resolver | | |
| eureka.server.rate-limiter-burst-size | 10 | |
| eureka.server.rate-limiter-enabled | false | |
| eureka.server.rate-limiter-full-fetch-average-rate | 100 | |
| eureka.server.rate-limiter-privileged-clients | | |
| eureka.server.rate-limiter-registry-fetch-average-rate | 500 | |
| eureka.server.rate-limiter-throttle-standard-clients | false | |
| eureka.server.registry-sync-retries | 0 | |
| eureka.server.registry-sync-retry-wait-ms | 0 | |
| eureka.server.remote-region-app-whitelist | | |
| eureka.server.remote-region-connect-timeout-ms | 1000 | |
| eureka.server.remote-region-connection-idle-timeout-seconds | 30 | |
| eureka.server.remote-region-fetch-thread-pool-size | 20 | |
| eureka.server.remote-region-read-timeout-ms | 1000 | |
| eureka.server.remote-region-registry-fetch-interval | 30 | |
| eureka.server.remote-region-total-connections | 1000 | |
| eureka.server.remote-region-total-connections-per-host | 500 | |
| eureka.server.remote-region-trust-store | | |
| eureka.server.remote-region-trust-store-password | changeit | |
| eureka.server.remote-region-urls | | |
| eureka.server.remote-region-urls-with-name | | |
| eureka.server.renewal-percent-threshold | 0.85 | |
| eureka.server.renewal-threshold-update-interval-ms | 0 | |
| eureka.server.response-cache-auto-expiration-in-seconds | 180 | |
| eureka.server.response-cache-update-interval-ms | 0 | |
| eureka.server.retention-time-in-m-s-in-delta-queue | 0 | |
| eureka.server.route53-bind-rebind-retries | 3 | |
| eureka.server.route53-binding-retry-interval-ms | 0 | |
| eureka.server.route53-domain-t-t-l | 30 | |
| eureka.server.sync-when-timestamp-different | true | |
| eureka.server.use-read-only-response-cache | true | |
| eureka.server.wait-time-in-ms-when-sync-empty | 0 | |
| eureka.server.xml-codec-name | | |
| health.config.enabled | false |标志，指示应安装配置服务器运行状况指示器. |
| health.config.time-to-live | 0 |缓存结果的生存时间（以毫秒为单位）.默认300000（5分钟）. |
| hystrix.metrics.enabled | true |启用Hystrix指标轮询.默认为true. |
| hystrix.metrics.polling-interval-ms | 2000 |后续轮询指标之间的间隔.默认为2000毫秒. |
| hystrix.shareSecurityContext | false |启用Hystrix并发策略插件挂钩的自动配置，该挂钩将 `SecurityContext` 从主线程传输到Hystrix命令使用的线程. |
| management.endpoint.bindings.cache.time-to-live | 0ms |可以缓存响应的最长时间. |
| management.endpoint.bindings.enabled | true |是否启用绑定endpoints. |
| management.endpoint.bus-env.enabled | true |是否启用bus-envendpoints. |
| management.endpoint.bus-refresh.enabled | true |是否启用总线刷新endpoints. |
| management.endpoint.channels.cache.time-to-live | 0ms |可以缓存响应的最长时间. |
| management.endpoint.channels.enabled | true |是否启用通道endpoints. |
| management.endpoint.consul.cache.time-to-live | 0ms |可以缓存响应的最长时间. |
| management.endpoint.consul.enabled | true |是否启用consulendpoints. |
| management.endpoint.env.post.enabled | true |启用通过POST更改环境到/ env. |
| management.endpoint.features.cache.time-to-live | 0ms |可以缓存响应的最长时间. |
| management.endpoint.features.enabled | true |是否启用功能endpoints. |
| management.endpoint.gateway.enabled | true |是否启用网关endpoints. |
| management.endpoint.hystrix.config | | Hystrix设置.传统上使用servlet参数设置它们.有关更多详细信息，请参阅Hystrix的文档. |
| management.endpoint.hystrix.stream.enabled | true |是否启用hystrix.streamendpoints. |
| management.endpoint.pause.enabled | true |启用/ pauseendpoints（发送Lifecycle.stop（））. |
| management.endpoint.refresh.enabled | true |启用/ refreshendpoints以刷新配置并重新初始化刷新范围的bean. |
| management.endpoint.restart.enabled | true |启用/ restartendpoints以重新启动应用程序上下文. |
| management.endpoint.resume.enabled | true |启用/ resume endpoint（发送Lifecycle.start（））. |
| management.endpoint.service-registry.cache.time-to-live | 0ms |可以缓存响应的最长时间. |
| management.endpoint.service-registry.enabled | true |是否启用服务注册表endpoints. |
| management.health.refresh.enabled | true |为刷新范围启用运行状况endpoints. |
| management.health.zookeeper.enabled | true |为zookeeper启用运行状况终结点. |
| management.metrics.binders.hystrix.enabled | true |启用创建OK Http客户端工厂bean. |
| proxy.auth.load-balanced | false | |
| proxy.auth.routes | |每条路线的认证策略. |
| ribbon.eager-load.clients | | |
| ribbon.eager-load.enabled | false | |
| ribbon.eureka.enabled | true |允许使用带有功能区的Eureka. |
| ribbon.http.client.enabled | false |不推荐使用功能以启用Ribbon RestClient. |
| ribbon.okhttp.enabled | false |允许使用带有功能区的OK HTTP客户端. |
| ribbon.restclient.enabled | false |允许使用已弃用的Ribbon RestClient. |
| ribbon.secure-ports | | |
| spring.cloud.bus.ack.destination-service | |想听ack的服务.默认为null（表示所有服务）. |
| spring.cloud.bus.ack.enabled | true |标志以关闭acks（默认打开）. |
| spring.cloud.bus.destination | springCloudBus |消息的Spring Cloud Stream目的地名称. |
| spring.cloud.bus.enabled | true |标志表示总线已启用. |
| spring.cloud.bus.env.enabled | true |标志以关闭环境更改事件（默认打开）. |
| spring.cloud.bus.id | application |此应用程序实例的标识符. |
| spring.cloud.bus.refresh.enabled | true |标志以关闭刷新事件（默认打开）. |
| spring.cloud.bus.trace.enabled | false |用于打开ack跟踪的标志（默认关闭）. |
| spring.cloud.cloudfoundry.discovery.default-server-port | 80 |功能区未定义端口时使用的端口. |
| spring.cloud.cloudfoundry.discovery.enabled | true |标记表示已启用发现. |
| spring.cloud.cloudfoundry.discovery.heartbeat-frequency | 5000 |心跳的轮询频率（以毫秒为单位）.客户端将按此频率进行轮询并广播服务ID列表. |
| spring.cloud.cloudfoundry.org | |最初定位的组织名称. |
| spring.cloud.cloudfoundry.password | |用户验证和获取令牌的密码. |
| spring.cloud.cloudfoundry.skip-ssl-validation | false | |
| spring.cloud.cloudfoundry.space | |最初目标的空间名称. |
| spring.cloud.cloudfoundry.url | | Cloud Foundry API（Cloud控制器）的URL. |
| spring.cloud.cloudfoundry.username | |用于进行身份验证的用户名（通常是电子邮件地址）. |
| spring.cloud.config.allow-override | true |标志，表示可以使用{@link #isOverrideSystemProperties（）systemPropertiesOverride}.设置为false以防止用户意外更改默认值.默认为true. |
| spring.cloud.config.discovery.enabled | false |标记表示启用了配置服务器发现（将通过发现查找配置服务器URL）. |
| spring.cloud.config.discovery.service-id | configserver |用于查找配置服务器的服务ID. |
| spring.cloud.config.enabled | true |标记表示已启用远程配置.默认为true; |
| spring.cloud.config.fail-fast | false |指示无法连接到服务器的标志是致命的（默认为false）. |
| spring.cloud.config.headers | |用于创建客户端请求的其他标头. |
| spring.cloud.config.label | |用于提取远程配置属性的标签名称.默认值在服务器上设置（对于基于git的服务器通常为“master”）. |
| spring.cloud.config.name | |用于获取远程属性的应用程序的名称. |
| spring.cloud.config.override-none | false |标志表示当{@link #setAllowOverride（boolean）allowOverride}为true时，外部属性应采用最低优先级，并且不覆盖任何现有属性源（包括本地配置文件） ）.默认为false. |
| spring.cloud.config.override-system-properties | true |标志，指示外部属性应覆盖系统属性.默认为true. |
| spring.cloud.config.password | |联系远程服务器时要使用的密码（HTTP Basic）. |
| spring.cloud.config.profile | default |获取远程配置时使用的默认配置文件（以逗号分隔）.默认为“默认”. |
| spring.cloud.config.request-read-timeout | 0 |超时等待从Config Server读取数据. |
| spring.cloud.config.retry.initial-interval | 1000 |初始重试间隔（以毫秒为单位）. |
| spring.cloud.config.retry.max-attempts | 6 |最大尝试次数. |
| spring.cloud.config.retry.max-interval | 2000 |退避的最大间隔. |
| spring.cloud.config.retry.multiplier | 1.1 |下一个时间间隔的乘数. |
| spring.cloud.config.send-state | true |指示是否发送状态的标志.默认为true. |
| spring.cloud.config.server.accept-empty | true |标志，表示如果找不到应用程序，则需要发送HTTP 404
| spring.cloud.config.server.bootstrap | false |标志，指示配置服务器应使用远程存储库中的属性初始化其自己的Environment.默认情况下关闭，因为它会延迟启动，但在将服务器嵌入另一个应用程序时非常有用. |
| spring.cloud.config.server.default-application-name | application |传入请求没有特定请求时的默认应用程序名称.|
| spring.cloud.config.server.default-label | |传入请求没有特定标签时的默认存储库标签. |
| spring.cloud.config.server.default-profile | default |传入请求没有特定请求时的默认应用程序配置文件. |
| spring.cloud.config.server.encrypt.enabled | true |在发送到客户端之前启用环境属性的解密. |
| spring.cloud.config.server.git.basedir | |存储库的本地工作副本的基目录. |
| spring.cloud.config.server.git.clone-on-start | false |标志，指示应在启动时克隆存储库（不按需）.通常会导致启动速度变慢但首次查询速度更快|
| spring.cloud.config.server.git.default-label | |与remore存储库一起使用的默认标签
| spring.cloud.config.server.git.delete-untracked-branches | false |标记表示如果删除了原始跟踪分支，则应在本地删除分支. |
| spring.cloud.config.server.git.force-pull | false |标记表示存储库应强制拉取.如果为true，则丢弃任何本地更改并从远程存储库获取. |
| spring.cloud.config.server.git.host-key | |有效的SSH主机密钥.如果还设置了hostKeyAlgorithm，则必须设置. |
| spring.cloud.config.server.git.host-key-algorithm | | ssh-dss，ssh-rsa，ecdsa-sha2-nistp256，ecdsa-sha2-nistp384或ecdsa-sha2-nistp521之一.如果还设置了hostKey，则必须设置. |
| spring.cloud.config.server.git.ignore-local-ssh-settings | false |如果为true，则使用基于属性而不是基于文件的SSH配置. |
| spring.cloud.config.server.git.known-hosts-file | |自定义.known_hosts文件的位置. |
| spring.cloud.config.server.git.order | |环境存储库的顺序. |
| spring.cloud.config.server.git.passphrase | |用于解锁ssh私钥的密码. |
| spring.cloud.config.server.git.password | |用于使用远程存储库进行身份验|
| spring.cloud.config.server.git.preferred-authentications | |覆盖服务器身份验证方法顺序.如果服务器在publickey方法之前具有键盘交互式身份验证，则应允许避免登录提示. |
| spring.cloud.config.server.git.private-key | |有效的SSH私钥.如果ignoreLocalSshSettings为true且Git URI为SSH格式，则必须设置. |
| spring.cloud.config.server.git.proxy | | HTTP代理配置. |
| spring.cloud.config.server.git.refresh-rate | 0 |刷新git存储库之间的时间（以秒为单位）|
| spring.cloud.config.server.git.repos | |存储库标识符到位置和其他属性的映射. |
| spring.cloud.config.server.git.search-paths | |搜索在本地工作副本中使用的路径.默认情况下，仅搜索根. |
| spring.cloud.config.server.git.skip-ssl-validation | false |标记，指示在与通过HTTPS连接提供的存储库进行通信时应绕过SSL证书验证. |
| spring.cloud.config.server.git.strict-host-key-checking | true |如果为false，则忽略主机密钥错误
| spring.cloud.config.server.git.timeout | 5 |获取HTTP或SSH连接的超时（以秒为单位）（如果适用），默认为5秒. |
| spring.cloud.config.server.git.uri | |远程存储库的URI. |
| spring.cloud.config.server.git.username | |用于远程存储库验证的用户名. |
| spring.cloud.config.server.health.repositories | | |
| spring.cloud.config.server.jdbc.order | 0 | |
| spring.cloud.config.server.jdbc.sql | SELECT KEY，来自PROPERTIES的VALUE = APPLICATION =？和PROFILE =？和LABEL =？ | SQL用于查询数据库中的键和值|
| spring.cloud.config.server.native.add-label-locations | true |用于确定是否应添加标签位置的标志. |
| spring.cloud.config.server.native.default-label | master | |
| spring.cloud.config.server.native.fail-on-error | false |用于确定解密期间如何处理异常的标志（默认为false）. |
| spring.cloud.config.server.native.order | | |
| spring.cloud.config.server.native.search-locations | [] |搜索配置文件的位置.默认为与Spring Boot应用程序相同，因此[classpath：/，classpath：/ config /，file：./，file：./ config /]. |
| spring.cloud.config.server.native.version | |要为本机存储库报告的版本字符串|
| spring.cloud.config.server.overrides | |无条件地向所有客户端发送的属性源的额外映射. |
| spring.cloud.config.server.prefix | |配置资源路径的前缀（默认为空）.当您不想更改上下文路径或servlet路径时，在嵌入另一个应用程序时很有用. |
| spring.cloud.config.server.strip-document-from-yaml | true |标记表示应该以“本机”形式返回作为文本或集合（而不是Map）的YAML文档. |
| spring.cloud.config.server.svn.basedir | |存储库的本地工作副本的基目录. |
| spring.cloud.config.server.svn.default-label | |与remore存储库一起使用的默认标签|
| spring.cloud.config.server.svn.order | |环境存储库的顺序. |
| spring.cloud.config.server.svn.passphrase | |用于解锁ssh私钥的密码. |
| spring.cloud.config.server.svn.password | |用于使用远程存储库进行身份验|
| spring.cloud.config.server.svn.search-paths | |搜索在本地工作副本中使用的路径.默认情况下，仅搜索根. |
| spring.cloud.config.server.svn.strict-host-key-checking | true |拒绝来自不在已知主机列表中的远程服务器的传入SSH主机密钥. |
| spring.cloud.config.server.svn.uri | |远程存储库的URI. |
| spring.cloud.config.server.svn.username | |用于远程存储库验证的用户名. |
| spring.cloud.config.server.vault.backend | secret | Vault后端.默认为秘密. |
| spring.cloud.config.server.vault.default-key | application |所有应用程序共享的保管库中的密钥.默认为应用程序.设置为空以禁用. |
| spring.cloud.config.server.vault.host | 127.0.0.1 |保管库主机.默认为127.0.0.1. |
| spring.cloud.config.server.vault.kv-version | 1 |值，指示使用哪个版本的Vault kv后端.默认为1. |
| spring.cloud.config.server.vault.order | | |
| spring.cloud.config.server.vault.port | 8200 | Vault端口.默认为8200.|
| spring.cloud.config.server.vault.profile-separator |，| Vault配置文件分隔符.默认为逗号. |
| spring.cloud.config.server.vault.proxy | | HTTP代理配置. |
| spring.cloud.config.server.vault.scheme | http | Vault计划.默认为http. |
| spring.cloud.config.server.vault.skip-ssl-validation | false |标记，指示在与通过HTTPS连接提供的存储库进行通信时应绕过SSL证书验证. |
| spring.cloud.config.server.vault.timeout | 5 |获取HTTP连接的超时（以秒为单位），默认为5秒. |
| spring.cloud.config.token | |安全令牌通过底层环境存储库. |
| spring.cloud.config.uri | [[http://localhost:8888](http://localhost:8888)] |远程服务器的URI（默认为[http://localhost:8888](http://localhost:8888)）. |
| spring.cloud.config.username | |联系远程服务器时要使用的用户名（HTTP Basic）. |
| spring.cloud.consul.config.acl-token | | |
| spring.cloud.consul.config.data-key | data |如果format是Format.PROPERTIES或Format.YAML，则以下字段用作查找consul进行配置的键. |
| spring.cloud.consul.config.default-context | application | |
| spring.cloud.consul.config.enabled | true | |
| spring.cloud.consul.config.fail-fast | true |如果为true，则在配置查找期间抛出异常，否则返回日志警告. |
| spring.cloud.consul.config.format | | |
| spring.cloud.consul.config.name | | spring.application.name的替代方法，用于在consul KV中查找值. |
| spring.cloud.consul.config.prefix | config | |
| spring.cloud.consul.config.profile-separator |，| |
| spring.cloud.consul.config.watch.delay | 1000 |以毫秒为单位的Watch固定延迟值.默认为1000. |
| spring.cloud.consul.config.watch.enabled | true |如果启用了Watch.默认为true. |
| spring.cloud.consul.config.watch.wait-time | 55 |等待（或阻止）监视查询的秒数，默认为55.需要小于默认的ConsulClient（默认为60）.要增加ConsulClient超时，请使用带有自定义HttpClient的自定义ConsulRawClient创建ConsulClient bean. |
| spring.cloud.consul.discovery.acl-token | | |
| spring.cloud.consul.discovery.catalog-services-watch-delay | 1000 |以毫秒为单位调用consul目录的调用之间的延迟，默认值为1000.
| spring.cloud.consul.discovery.catalog-services-watch-timeout | 2 |观看consul目录时阻塞的秒数，默认为2.
| spring.cloud.consul.discovery.datacenters | | serviceId的Map→要在服务器列表中查询的数据中心.这允许在另一个数据中心中查找服务. |
| spring.cloud.consul.discovery.default-query-tag | |标记为在服务列表中查询（如果未在serverListQueryTags中列出）. |
| spring.cloud.consul.discovery.default-zone-metadata-name | zone |服务实例区域来自元数据.这允许更改元数据标签名称. |
| spring.cloud.consul.discovery.deregister | true |禁用Consul中服务的自动注销. |
| spring.cloud.consul.discovery.enabled | true |是否启用了服务发现？ |
| spring.cloud.consul.discovery.fail-fast | true |如果为true，则在服务注册期间抛出异常，否则为日志警告（默认为true）. |
| spring.cloud.consul.discovery.health-check-critical-timeout | |超时取消注册服务的时间超过超时（例如30米）.需要consul版本7.x或更高版本. |
| spring.cloud.consul.discovery.health-check-interval | 10s |执行运行状况检查的频率（例如10s），默认为10s. |
| spring.cloud.consul.discovery.health-check-path | / actuator / health |要调用以进行运行状况检查的备用服务器路径
| spring.cloud.consul.discovery.health-check-timeout ||Health检查超时（例如10秒）. |
| spring.cloud.consul.discovery.health-check-tls-skip-verify | |如果为true，则在服务检查期间跳过证书验证，否则运行证书验证. |
| spring.cloud.consul.discovery.health-check-url | |自定义运行状况检查URL以覆盖默认值|
| spring.cloud.consul.discovery.heartbeat.enabled | false | |
| spring.cloud.consul.discovery.heartbeat.interval-ratio | | |
| spring.cloud.consul.discovery.heartbeat.ttl-unit | s | |
| spring.cloud.consul.discovery.heartbeat.ttl-value | 30 | |
| spring.cloud.consul.discovery.hostname | |访问服务器时使用的主机名|
| spring.cloud.consul.discovery.instance-group | |服务实例组|
| spring.cloud.consul.discovery.instance-id | |唯一服务实例ID |
| spring.cloud.consul.discovery.instance-zone | |服务实例区|
| spring.cloud.consul.discovery.ip-address | |访问服务时使用的IP地址（还必须设置preferIpAddress使用）|
| spring.cloud.consul.discovery.lifecycle.enabled | true | |
| spring.cloud.consul.discovery.management-port | |在（默认为管理端口）|下注册管理服务的端口
| spring.cloud.consul.discovery.management-suffix | management |注册管理服务时使用的后缀
| spring.cloud.consul.discovery.management-tags | |注册管理服务时使用的标签|
| spring.cloud.consul.discovery.port | |在（默认为侦听端口）|下注册服务的端口
| spring.cloud.consul.discovery.prefer-agent-address | false |我们将如何确定要使用的地址的来源
| spring.cloud.consul.discovery.prefer-ip-address | false |在注册期间使用ip地址而不是主机名
| spring.cloud.consul.discovery.query-passing | false |将'passing`参数添加到/ v1 / health / service / serviceName.这会将运行状况检查传递给服务器. |
| spring.cloud.consul.discovery.register | true |在Consul中注册为服务. |
| spring.cloud.consul.discovery.register-health-check | true |在Consul登记Health检查.在开发服务期间很有用. |
| spring.cloud.consul.discovery.scheme | http |是否注册http或https服务|
| spring.cloud.consul.discovery.server-list-query-tags | | serviceId的Map→要在服务器列表中查询的标记.这允许通过单个标签过滤服务. |
| spring.cloud.consul.discovery.service-name | |服务名称|
| spring.cloud.consul.discovery.tags | |注册服务时使用的标签|
| spring.cloud.consul.enabled | true |启用Spring cloud consul |
| spring.cloud.consul.host | localhost | Consul代理主机名.默认为“localhost”. |
| spring.cloud.consul.port | 8500 |Consul代理端口.默认为'8500'. |
| spring.cloud.consul.retry.initial-interval | 1000 |初始重试间隔（以毫秒为单位）. |
| spring.cloud.consul.retry.max-attempts | 6 |最大尝试次数. |
| spring.cloud.consul.retry.max-interval | 2000 |退避的最大间隔. |
| spring.cloud.consul.retry.multiplier | 1.1 |下一个时间间隔的乘数. |
| spring.cloud.consul.scheme | | Consul代理方案（HTTP / HTTPS）.如果地址中没有方案 - 客户端将使用HTTP. |
| spring.cloud.consul.tls.certificate-password | |用于打开证书的密码. |
| spring.cloud.consul.tls.certificate-path | |证书的文件路径. |
| spring.cloud.consul.tls.key-store-instance-type | |要使用的密钥框架的类型. |
| spring.cloud.consul.tls.key-store-password | |外部密钥库的密码|
| spring.cloud.consul.tls.key-store-path | |外部密钥库的路径|
| spring.cloud.discovery.client.health-indicator.enabled | true | |
| spring.cloud.discovery.client.health-indicator.include-description | false | |
| spring.cloud.discovery.client.simple.instances | | |
| spring.cloud.discovery.client.simple.local.metadata | |服务实例的元数据.发现客户端可以使用它来修改每个实例的行为，例如：负载平衡时. |
| spring.cloud.discovery.client.simple.local.service-id | |服务的标识符或名称.多个实例可能共享相同的服务ID. |
| spring.cloud.discovery.client.simple.local.uri | |服务实例的URI.将被解析以提取方案，hos和端口. |
| spring.cloud.gateway.default-filters | |应用于每个路由的过滤器定义列表. |
| spring.cloud.gateway.discovery.locator.enabled | false |启用DiscoveryClient网关集成的标志
| spring.cloud.gateway.discovery.locator.filters | | |
| spring.cloud.gateway.discovery.locator.include-expression | true |将评估是否在网关集成中包含服务的SpEL表达式，默认为：true |
| spring.cloud.gateway.discovery.locator.lower-case-service-id | false |在谓词和过滤器中小写serviceId的选项，默认为false.当eureka自动大写serviceId时，它很有用.所以MYSERIVCE，会匹配/ myservice / ** |
| spring.cloud.gateway.discovery.locator.predicates || |
| spring.cloud.gateway.discovery.locator.route-id-prefix | | routeId的前缀，默认为discoveryClient.getClass（）.getSimpleName（）“_”.将附加服务ID以创建routeId. |
| spring.cloud.gateway.discovery.locator.url-expression |'lb：//'serviceId |为每条路由创建uri的SpEL表达式，默认为：'lb：//'serviceId |
| spring.cloud.gateway.enabled | true |启用网关功能. |
| spring.cloud.gateway.filter.remove-hop-by-hop.headers | | |
| spring.cloud.gateway.filter.remove-hop-by-hop.order | | |
| spring.cloud.gateway.filter.secure-headers.content-security-policy | default-src'self'https：; font-src'self'https：data：; img-src'self'https：data：; object-src'un'; script-src https：; style-src'self'https：'unsafe-inline'| |
| spring.cloud.gateway.filter.secure-headers.content-type-options | nosniff | |
| spring.cloud.gateway.filter.secure-headers.download-options | noopen | |
| spring.cloud.gateway.filter.secure-headers.frame-options | DENY | |
| spring.cloud.gateway.filter.secure-headers.permitted-cross-domain-policies | none | |
| spring.cloud.gateway.filter.secure-headers.referrer-policy | no-referrer | |
| spring.cloud.gateway.filter.secure-headers.strict-transport-security | max-age = 631138519 | |
| spring.cloud.gateway.filter.secure-headers.xss-protection-header | 1; mode = block | |
| spring.cloud.gateway.forwarded.enabled | true |启用ForwardedHeadersFilter. |
| spring.cloud.gateway.globalcors.cors-configurations | | |
| spring.cloud.gateway.httpclient.connect-timeout | |以毫秒为单位的连接超时，默认值为45秒. |
| spring.cloud.gateway.httpclient.pool.acquire-timeout | |仅适用于类型FIXED，等待查询的最长时间（毫秒）. |
| spring.cloud.gateway.httpclient.pool.max-connections | |仅适用于类型FIXED，即在现有启动等待获取之前的最大连接数. |
| spring.cloud.gateway.httpclient.pool.name | proxy |通道池映射名称，默认为proxy. |
| spring.cloud.gateway.httpclient.pool.type | |要使用的HttpClient池类型，默认为ELASTIC. |
| spring.cloud.gateway.httpclient.proxy.host | | Netty HttpClient代理配置的主机名. |
| spring.cloud.gateway.httpclient.proxy.non-proxy-hosts-pattern | |正则表达式（Java），用于配置的应直接访问的主机列表，绕过代理|
| spring.cloud.gateway.httpclient.proxy.password | | Netty HttpClient的代理配置密码. |
| spring.cloud.gateway.httpclient.proxy.port | | Netty HttpClient的代理配置端口. |
| spring.cloud.gateway.httpclient.proxy.username | | Netty HttpClient代理配置的用户名. |
| spring.cloud.gateway.httpclient.response-timeout | |响应超时. |
| spring.cloud.gateway.httpclient.ssl.close-notify-flush-timeout-millis | 3000 | |
| spring.cloud.gateway.httpclient.ssl.close-notify-read-timeout-millis | 0 | |
| spring.cloud.gateway.httpclient.ssl.handshake-timeout-millis | 10000 | |
| spring.cloud.gateway.httpclient.ssl.trusted-x509-certificates | | |
| spring.cloud.gateway.httpclient.ssl.use-insecure-trust-manager | false |安装netty InsecureTrustManagerFactory.这是不安全的，不适合生产环境. |
| spring.cloud.gateway.metrics.enabled | false |启用指标数据的收集. |
| spring.cloud.gateway.proxy.headers | |修复了将添加到所有下游请求的标头值. |
| spring.cloud.gateway.proxy.sensitive | |默认情况下不会向下游发送的一组敏感标头名称. |
| spring.cloud.gateway.redis-rate-limiter.burst-capacity-header | X-RateLimit-Burst-Capacity |返回突发容量配置的标头名称. |
| spring.cloud.gateway.redis-rate-limiter.config | | |
| spring.cloud.gateway.redis-rate-limiter.include-headers | true |是否包含包含速率限制器信息的标头，默认为true. |
| spring.cloud.gateway.redis-rate-limiter.remaining-header | X-RateLimit-Remaining |返回当前秒数内剩余请求数的标头名称. |
| spring.cloud.gateway.redis-rate-limiter.replenish-rate-header | X-RateLimit-Replenish-Rate |返回补充率配置的Headers的名称. |
| spring.cloud.gateway.routes | |路线列表|
| spring.cloud.gateway.streaming-media-types | | |
| spring.cloud.gateway.x-forwarded.enabled | true |如果启用了XForwardedHeadersFilter. |
| spring.cloud.gateway.x-forwarded.for-append | true |如果启用了附加X-Forwarded-For列表. |
| spring.cloud.gateway.x-forwarded.for-enabled | true |如果启用了X-Forwarded-For. |
| spring.cloud.gateway.x-forwarded.host-append | true |如果启用了附加X-Forwarded-Host作为列表. |
| spring.cloud.gateway.x-forwarded.host-enabled | true |如果启用了X-Forwarded-Host. |
| spring.cloud.gateway.x-forwarded.order | 0 | XForwardedHeadersFilter的顺序. |
| spring.cloud.gateway.x-forwarded.port-append | true | If将X-Forwarded-Port附加为列表已启用. |
| spring.cloud.gateway.x-forwarded.port-enabled | true |如果启用了X-Forwarded-Port. |
| spring.cloud.gateway.x-forwarded.prefix-append | true |如果启用了附加X-Forwarded-Prefix作为列表. |
| spring.cloud.gateway.x-forwarded.prefix-enabled | true |如果启用了X-Forwarded-Prefix. |
| spring.cloud.gateway.x-forwarded.proto-append | true |如果启用了附加X-Forwarded-Proto列表. |
| spring.cloud.gateway.x-forwarded.proto-enabled | true |如果启用了X-Forwarded-Proto. |
| spring.cloud.hypermedia.refresh.fixed-delay | 5000 | |
| spring.cloud.hypermedia.refresh.initial-delay | 10000 | |
| spring.cloud.inetutils.default-hostname | localhost |默认主机名.用于出错的情况. |
| spring.cloud.inetutils.default-ip-address | 127.0.0.1 |默认的ipaddress.用于出错的情况. |
| spring.cloud.inetutils.ignored-interfaces | |将被忽略的网络接口的Java正则表达式列表. |
| spring.cloud.inetutils.preferred-networks | |将首选的网络地址的Java正则表达式列表. |
| spring.cloud.inetutils.timeout-seconds | 1 |计算主机名的超时秒数. |
| spring.cloud.inetutils.use-only-site-local-interfaces | false |仅使用具有站点本地地址的接口.有关更多详细信息，请参阅{@link InetAddress＃isSiteLocalAddress（）}. |
| spring.cloud.loadbalancer.retry.enabled | true | |
| spring.cloud.refresh.extra-refreshable | true |将进程发布到刷新范围的bean的其他类名. |
| spring.cloud.service-registry.auto-registration.enabled | true |如果启用了自动服务注册，则默认为true. |
| spring.cloud.service-registry.auto-registration.fail-fast | false |如果没有AutoServiceRegistration，则启动失败，默认为false. |
| spring.cloud.service-registry.auto-registration.register-management | true |是否将管理注册为服务，默认为true |
| spring.cloud.stream.binders | |额外的每个Binders属性（请参阅{@link BinderProperties}）如果使用多个相同类型的Binders（即连接到RabbitMq的多个实例）.您可以在此处指定多个Binders配置，每个配置具有不同的环境设例如; spring.cloud.stream.binders.rabbit1.environment. . . ，spring.cloud.stream.binders.rabbit2.environment. . . |
| spring.cloud.stream.binding-retry-interval | 30 |用于计划绑定尝试的重试间隔（以秒为单位）.默认值：30秒. |
| spring.cloud.stream.bindings | |每个绑定名称的附加绑定属性（请参阅{@link BinderProperties}）（例如，'input`）.例如;这为Sink应用程序的“输入”绑定设置了内容类型：'spring.cloud.stream.bindings.input.contentType = text / plain'|
| spring.cloud.stream.consul.binder.event-timeout | 5 | |
| spring.cloud.stream.default-binder | |在多个Binders可用的情况下由所有绑定使用的Binders的名称（例如，'rabbit'）; |
| spring.cloud.stream.dynamic-destinations | [] |可动态绑定的目标列表.如果设置，则只能绑定列出的目标. |
| spring.cloud.stream.instance-count | 1 |应用程序的已部署实例数.默认值：1.注意：也可以按照单个绑定“spring.cloud.stream.bindings.foo.consumer.instance-count”进行管理，其中“foo”是绑定的名称. |
| spring.cloud.stream.instance-index | 0 |应用程序的实例ID：从0到instanceCount-1的数字.用于分区和Kafka.注意：也可以按照单个绑定“spring.cloud.stream.bindings.foo.consumer.instance-index”进行管理，其中“foo”是绑定的名称. |
| spring.cloud.stream.integration.message-handler-not-propagated-headers | |不会从入站邮件中复制的邮件头名称. |
| spring.cloud.stream.metrics.export-properties | |将附加到每条消息的属性列表.一旦上下文刷新以避免基于每个消息执行的开销，这将由onApplicationEvent填充. |
| spring.cloud.stream.metrics.key | |要发出的度量标准的名称.每个应用程序应该是唯一值.默认为：$ {spring.application.name：$ {vcap.application.name:${spring.config.name:application}}} |
| spring.cloud.stream.metrics.meter-filter | |用于控制想要捕捉的'米'的模式.默认情况下，将捕获所有“米”.例如，'spring.integration.*'仅捕获名称以'spring.integration'开头的米的度量信息. |
| spring.cloud.stream.metrics.properties | |应添加到度量标准负载的应用程序属性例如： `spring.application**`  |
| spring.cloud.stream.metrics.schedule-interval | 60s | Interval表示为调度度量快照发布的持续时间.默认为60秒|
| spring.cloud.stream.rabbit.binder.admin-addresses | [] |管理插件的网址;仅需要队列关联.|
| spring.cloud.stream.rabbit.binder.admin-adresses | | |
| spring.cloud.stream.rabbit.binder.compression-level | 0 |压缩绑定的压缩级别;参见'java.util.zip.Deflator'. |
| spring.cloud.stream.rabbit.binder.connection-name-prefix | |此Binders中连接名称的前缀. |
| spring.cloud.stream.rabbit.binder.nodes | [] |集群成员节点名称;仅需要队列关联. |
| spring.cloud.stream.rabbit.bindings | | |
| spring.cloud.vault.app-id.app-id-path | app-id | AppId身份验证后端的挂载路径. |
| spring.cloud.vault.app-id.network-interface | |网络接口提示“MAC_ADDRESS”UserId机制. |
| spring.cloud.vault.app-id.user-id | MAC_ADDRESS | UserId机制.可以是“MAC_ADDRESS”，“IP_ADDRESS”，字符串或类名. |
| spring.cloud.vault.app-role.app-role-path | approle | AppRole身份验证后端的挂载路径. |
| spring.cloud.vault.app-role.role | |角色名称，可选，用于拉模式. |
| spring.cloud.vault.app-role.role-id | | RoleId. |
| spring.cloud.vault.app-role.secret-id | | SecretId. |
| spring.cloud.vault.application-name | application | AppId身份验证的应用程序名称. |
| spring.cloud.vault.authentication | | |
| spring.cloud.vault.aws-ec2.aws-ec2-path | aws-ec2 | AWS-EC2身份验证后端的装载路径. |
| spring.cloud.vault.aws-ec2identity-document | [http://169.254.169.254/latest/dynamic/instance-identity/pkcs7](http://169.254.169.254/latest/dynamic/instance-identity/pkcs7) | AWS-EC2 PKCS7身份证件的URL. |
| spring.cloud.vault.aws-ec2.nonce | | Nonce用于AWS-EC2身份验证.空nonce默认为nonce生成. |
| spring.cloud.vault.aws-ec2.role | |角色名称，可选. |
| spring.cloud.vault.aws-iam.aws-path | aws | AWS身份验证后端的装载路径. |
| spring.cloud.vault.aws-iam.role | |角色名称，可选.如果未设置，则默认为友好的IAM名称. |
| spring.cloud.vault.aws-iam.server-name | |用于在登录请求标头中设置{@code X-Vault-AWS-IAM-Server-ID}标头的服务器的名称. |
| spring.cloud.vault.aws.access-key-property | cloud.aws.credentials.accessKey |获取的访问密钥的目标属性. |
| spring.cloud.vault.aws.backend | aws | aws后端路径. |
| spring.cloud.vault.aws.enabled | false |启用aws后端使用. |
| spring.cloud.vault.aws.role | |凭据的角色名称. |
| spring.cloud.vault.aws.secret-key-property | cloud.aws.credentials.secretKey |获取的密钥的目标属性. |
| spring.cloud.vault.cassandra.backend | cassandra | Cassandra后端路径. |
| spring.cloud.vault.cassandra.enabled | false |启用cassandra后端使用. |
| spring.cloud.vault.cassandra.password-property | spring.data.cassandra.password |获取密码的目标属性. |
| spring.cloud.vault.cassandra.role | |凭据的角色名称. |
| spring.cloud.vault.cassandra.username-property | spring.data.cassandra.username |获取的用户名的目标属性. |
| spring.cloud.vault.config.lifecycle.enabled | true |启用生命周期管理. |
| spring.cloud.vault.config.order | 0 |用于设置{@link org.springframework.core.env.PropertySource}优先级.这对于将Vault用作其他属性源的覆盖非常有用. @see org.springframework.core.PriorityOrdered |
| spring.cloud.vault.connection-timeout | 5000 |连接超时; |
| spring.cloud.vault.consul.backend | consul |Consul后端路径. |
| spring.cloud.vault.consul.enabled | false |启用consul后端使用. |
| spring.cloud.vault.consul.role | |凭据的角色名称. |
| spring.cloud.vault.consul.token-property | spring.cloud.consul.token |获取令牌的目标属性. |
| spring.cloud.vault.database.backend | database |数据库后端路径. |
| spring.cloud.vault.database.enabled | false |启用数据库后端使用情况. |
| spring.cloud.vault.database.password-property | spring.datasource.password |获取密码的目标属性. |
| spring.cloud.vault.database.role | |凭据的角色名称. |
| spring.cloud.vault.database.username-property | spring.datasource.username |获取的用户名的目标属性. |
| spring.cloud.vault.discovery.enabled | false |标记以指示已启用Vault服务器发现（将通过发现查找Vault服务器URL）. |
| spring.cloud.vault.discovery.service-id | vault |用于查找Vault的服务ID. |
| spring.cloud.vault.enabled | true |启用Vault配置服务器. |
| spring.cloud.vault.fail-fast | false |如果无法从Vault获取数据，则快速失败. |
| spring.cloud.vault.generic.application-name | application |要用于上下文的应用程序名称. |
| spring.cloud.vault.generic.backend | secret |默认后端的名称. |
| spring.cloud.vault.generic.default-context | application |默认上下文的名称. |
| spring.cloud.vault.generic.enabled | true |启用通用后端. |
| spring.cloud.vault.generic.profile-separator | / | Profile-separator用于组合应用程序名称和配置文件. |
| spring.cloud.vault.host | localhost | Vault服务器主机.|
| spring.cloud.vault.kubernetes.kubernetes-path | kubernetes | Kubernetes身份验证后端的挂载路径. |
| spring.cloud.vault.kubernetes.role | |正在尝试登录的角色的名称. |
| spring.cloud.vault.kubernetes.service-account-token-file | /var/run/secrets/kubernetes.io/serviceaccount/token |服务帐户令牌文件的路径. |
| spring.cloud.vault.kv.application-name | application |要用于上下文的应用程序名称. |
| spring.cloud.vault.kv.backend | secret |默认后端的名称. |
| spring.cloud.vault.kv.backend-version | 2 |键值后端版本.目前支持的版本是：<ul> <li>版本1（无版本键值后端）.</ li> <li>版本2（版本化键值后端）.</ li> </ ul> |
| spring.cloud.vault.kv.default-context | application |默认上下文的名称. |
| spring.cloud.vault.kv.enabled | false |启用kev-value后端. |
| spring.cloud.vault.kv.profile-separator | / |用于组合应用程序名称和配置文件的Profile-separator. |
| spring.cloud.vault.mongodb.backend | mongodb | Cassandra后端路径. |
| spring.cloud.vault.mongodb.enabled | false |启用mongodb后端使用. |
| spring.cloud.vault.mongodb.password-property | spring.data.mongodb.password |获取密码的目标属性. |
| spring.cloud.vault.mongodb.role | |凭据的角色名称. |
| spring.cloud.vault.mongodb.username-property | spring.data.mongodb.username |获取的用户名的目标属性. |
| spring.cloud.vault.mysql.backend | mysql | mysql后端路径. |
| spring.cloud.vault.mysql.enabled | false |启用mysql后端用法. |
| spring.cloud.vault.mysql.password-property | spring.datasource.password |获取的用户名的目标属性. |
| spring.cloud.vault.mysql.role | |凭据的角色名称. |
| spring.cloud.vault.mysql.username-property | spring.datasource.username |获取的用户名的目标属性. |
| spring.cloud.vault.port | 8200 | Vault服务器端口. |
| spring.cloud.vault.postgresql.backend | postgresql | postgresql后端路径. |
| spring.cloud.vault.postgresql.enabled | false |启用postgresql后端用法. |
| spring.cloud.vault.postgresql.password-property | spring.datasource.password |获取的用户名的目标属性. |
| spring.cloud.vault.postgresql.role | |凭据的角色名称. |
| spring.cloud.vault.postgresql.username-property | spring.datasource.username |获取的用户名的目标属性. |
| spring.cloud.vault.rabbitmq.backend | rabbitmq | rabbitmq后端路径. |
| spring.cloud.vault.rabbitmq.enabled | false |启用rabbitmq后端使用. |
| spring.cloud.vault.rabbitmq.password-property | spring.rabbitmq.password |获取密码的目标属性. |
| spring.cloud.vault.rabbitmq.role | |凭据的角色名称. |
| spring.cloud.vault.rabbitmq.username-property | spring.rabbitmq.username |获取的用户名的目标属性. |
| spring.cloud.vault.read-timeout | 15000 |读取超时; |
| spring.cloud.vault.scheme | https |协议方案.可以是“http”或“https”. |
| spring.cloud.vault.ssl.cert-auth-path | cert | TLS证书身份验证后端的装载路径. |
| spring.cloud.vault.ssl.key-store | |包含证书和私钥的信任存储. |
| spring.cloud.vault.ssl.key-store-password | |用于访问密钥库的密码. |
| spring.cloud.vault.ssl.trust-store | |包含SSL证书的信任存储. |
| spring.cloud.vault.ssl.trust-store-password | |用于访问信任库的密码. |
| spring.cloud.vault.token | |静态保险库令牌.如果{@link #authentication}是{@code TOKEN}，则必填. |
| spring.cloud.vault.uri | | Vault URI.可以使用方案，主机和端口进行设置. |
| spring.cloud.zookeeper.base-sleep-time-ms | 50 |重试之间等待的初始时间|
| spring.cloud.zookeeper.block-until-connected-unit | |与阻止连接到Zookeeper的时间单位|
| spring.cloud.zookeeper.block-until-connected-wait | 10 |等待时间阻止与Zookeeper的连接
| spring.cloud.zookeeper.connect-string | localhost：2181 |到Zookeeper集群的连接字符串|
| spring.cloud.zookeeper.default-health-endpoint | |将检查以验证依赖项是否存活的缺省运行状况endpoints
| spring.cloud.zookeeper.dependencies | |将别名映射到ZookeeperDependency.从Ribbon角度来看，别名实际上是serviceID，因为Ribbon不能接受serviceID中的嵌套结构
| spring.cloud.zookeeper.dependency-configurations | | |
| spring.cloud.zookeeper.dependency-names | | |
| spring.cloud.zookeeper.discovery.enabled | true | |
| spring.cloud.zookeeper.discovery.initial-status | |此实例的初始状态（默认为{@link StatusConstants＃STATUS_UP}）. |
| spring.cloud.zookeeper.discovery.instance-host | |预定义的主机，服务可以在Zookeeper中注册自己的主机.对应于URI规范中的{code address}. |
| spring.cloud.zookeeper.discovery.instance-id | | Id曾用于注册zookeeper.默认随机UUID. |
| spring.cloud.zookeeper.discovery.instance-port | |在（默认为侦听端口）|下注册服务的端口
| spring.cloud.zookeeper.discovery.instance-ssl-port | |已注册服务的Ssl端口. |
| spring.cloud.zookeeper.discovery.metadata | |获取与此实例关联的元数据名称/值对.此信息将发送到zookeeper，并可供其他实例使用. |
| spring.cloud.zookeeper.discovery.register | true |在zookeeper中注册为服务. |
| spring.cloud.zookeeper.discovery.root | / services | Root Zookeeper文件夹，其中所有实例都已注册
| spring.cloud.zookeeper.discovery.uri-spec | {scheme}：// {address}：{port} |在Zookeeper中注册服务期间要解析的URI规范
| spring.cloud.zookeeper.enabled | true |是否启用了Zookeeper
| spring.cloud.zookeeper.max-retries | 10 |重试次数的最大次数
| spring.cloud.zookeeper.max-sleep-ms | 500 |每次重试时睡眠的最长时间（以毫秒为单位）|
| spring.cloud.zookeeper.prefix | |将应用于所有Zookeeper依赖项路径的公共前缀|
| spring.integration.poller.fixed-delay | 1000 |默认轮询器的固定延迟. |
| spring.integration.poller.max-messages-per-poll | 1 |默认轮询器的每次轮询的最大消息数. |
| spring.sleuth.annotation.enabled | true | |
| spring.sleuth.async.configurer.enabled | true |启用默认的AsyncConfigurer. |
| spring.sleuth.async.enabled | true |启用检测异步相关组件，以便在线程之间传递跟踪信息. |
| spring.sleuth.baggage-keys | |应该在进程外传播的Baggage密钥名称列表.这些键在实际键之前将以 `baggage` 为前缀.设置此属性是为了向后兼容之前的Sleuth版本. @see brave.propagation.ExtraFieldPropagation.FactoryBuilder #addPrefixedFields（String，java.util.Collection）|
| spring.sleuth.enabled | true | |
| spring.sleuth.feign.enabled | true |使用Feign时启用Span信息传播. |
| spring.sleuth.feign.processor.enabled | true |启用在其跟踪表示中包装Feign Context的后处理器. |
| spring.sleuth.http.enabled | true | |
| spring.sleuth.http.legacy.enabled | false | |
| spring.sleuth.hystrix.strategy.enabled | true |启用自定义HystrixConcurrencyStrategy，将所有Callable实例包装到其Sleuth代表中 -  TraceCallable. |
| spring.sleuth.integration.enabled | true |启用Spring Integration侦听工具. |
| spring.sleuth.integration.patterns | [！hystrixStreamOutput *，*] |将与之匹配的模式数组. @see org.springframework.integration.config.GlobalChannelInterceptor＃patterns（）.默认为与Hystrix Stream通道名称不匹配的任何通道名称. |
| spring.sleuth.integration.websockets.enabled | true |为WebSockets启用跟踪. |
| spring.sleuth.keys.http.headers | |如果存在，则应添加为标记的其他Headers.如果标头值是多值的，则标记值将是逗号分隔的单引号列表. |
| spring.sleuth.keys.http.prefix | http. |Headers名称的前缀（如果它们作为标记添加）. |
| spring.sleuth.log.slf4j.enabled | true |启用在日志中打印跟踪信息的{@link Slf4jCurrentTraceContext}. |
| spring.sleuth.messaging.enabled | false | |
| spring.sleuth.messaging.kafka.enabled | false | |
| spring.sleuth.messaging.kafka.remote-service-name | kafka | |
| spring.sleuth.messaging.rabbit.enabled | false | |
| spring.sleuth.messaging.rabbit.remote-service-name | rabbitmq | |
| spring.sleuth.opentracing.enabled | true | |
| spring.sleuth.propagation-keys | |在进程中引用的字段列表与在线上引用的字段列表相同.例如，名称“x-vcap-request-id”将按原样设置，包括前缀. <p>注意：{@ code fieldName}将隐式低级. @see brave.propagation.ExtraFieldPropagation.FactoryBuilder＃addField（String）|
| spring.sleuth.reactor.enabled.enabled | true |当为true启用reactor的检测时
| spring.sleuth.rxjava.schedulers.hook.enabled | true |通过RxJavaSchedulersHook启用对RxJava的支持. |
| spring.sleuth.rxjava.schedulers.ignoredthreads | [HystrixMetricPoller，^ RxComputation.* $] |不对其进行Span采样的线程名称. |
| spring.sleuth.sampler.probability | 0.1 |应采样的请求的概率.例如.应该对1.0  -  100％的请求进行抽样.精度仅为整数（即不支持0.1％的迹线）. |
| spring.sleuth.scheduled.enabled | true |启用{@link org.springframework.scheduling.annotation.Scheduled}的跟踪. |
| spring.sleuth.scheduled.skip-pattern | org.springframework.cloud.netflix.hystrix.stream.HystrixStreamTask |应跳过的类的完全限定名称的模式. |
| spring.sleuth.supports-join | true | True表示跟踪系统支持在客户端和服务器之间共享span ID. |
| spring.sleuth.trace-id128 | false |当为true时，生成128位跟踪ID而不是64位跟踪ID. |
| spring.sleuth.web.additional-skip-pattern | |在跟踪中应跳过的URL的其他模式.这将附加到{@link SleuthWebProperties #skipPattern} |
| spring.sleuth.web.client.enabled | true |启用拦截器注入{@link org.springframework.web.client.RestTemplate} |
| spring.sleuth.web.enabled | true |当为true启用Web应用程序的检测时|
| spring.sleuth.web.exception-throwing-filter-enabled | true |标记以切换记录抛出的异常的过滤器的存在
| spring.sleuth.web.filter-order | |应在其中注册跟踪过滤器的顺序.默认为{@link TraceHttpAutoConfiguration＃TRACING_FILTER_ORDER} |
| spring.sleuth.web.skip-pattern | /api-docs.* | / autoconfig |
| / configprops | / dump | / health |
| / info | /metrics.* | / mappings |
| / trace | /swagger.* |.* \.png |
|.* \.css |.* \.js |.* \.html |
| /favicon.ico | /hystrix.stream | / application /.* |
| /actuator.* | / cloudfoundryapplication |跟踪中应跳过的URL的模式
| spring.sleuth.zuul.enabled | true |使用Zuul时启用Span信息传播. |
| stubrunner.amqp.enabled | false |是否启用对Stub Runner和AMQP的支持. |
| stubrunner.amqp.mockCOnnection | true |是否启用对Stub Runner和AMQP模拟连接工厂的支持. |
| stubrunner.classifier | stubs |默认情况下在常Spring藤的坐标中使用的分类器. |
| stubrunner.cloud.consul.enabled | true |是否在Consul中启用存根注册. |
| stubrunner.cloud.delegate.enabled | true |是否启用DiscoveryClient的Stub Runner实现. |
| stubrunner.cloud.enabled | true |是否为Stub Runner启用Spring Cloud支持. |
| stubrunner.cloud.eureka.enabled | true |是否在Eureka中启用存根注册. |
| stubrunner.cloud.ribbon.enabled | true |是否启用Stub Runner的功能区集成. |
| stubrunner.cloud.stubbed.discovery.enabled | true |是否应为Stub Runner存根服务发现.如果设置为false，则存根将在实际服务发现中注册. |
| stubrunner.cloud.zookeeper.enabled | true |是否在Zookeeper中启用存根注册. |
| stubrunner.consumer-name | |您可以通过为此参数设置值来覆盖此字段的默认{@code spring.application.name}. |
| stubrunner.delete-stubs-after-test | true |如果设置为{@code false}，则在运行测试后不会从临时文件夹中删除存根
| stubrunner.ids | [] |以“常Spring藤”表示法运行的存根的ID（[groupId]：artifactId：[version]：[classifier] [：port]）. {@code groupId}，{@ code classifier}，{@ code version}和{@code port}可以是可选的. |
| stubrunner.ids-to-service-ids | |将基于常Spring藤符号的id映射到应用程序中的serviceId示例“a：b”→“myService”“artifactId”→“myOtherService”|
| stubrunner.integration.enabled | true |是否启用Stub Runner与Spring Integration的集成. |
| stubrunner.mappings-output-folder | |将每个HTTP服务器的映射转储到所选文件夹|
| stubrunner.max-port | 15000 |自动启动的WireMock服务器端口的最大值|
| stubrunner.min-port | 10000 |自动启动的WireMock服务器端口的最小值
| stubrunner.password | |存储库密码|
| stubrunner.properties | |可以传递给自定义的属性的Map{@link org.springframework.cloud.contract.stubrunner.StubDownloaderBuilder} |
| stubrunner.proxy-host | |存储库代理主机|
| stubrunner.proxy-port | |存储库代理端口|
| stubrunner.snapshot-check-skip | false |如果设置为{@code true}，则不会断言下载的存根/ContractJAR是从远程位置还是从本地下载的（仅适用于Maven存储库，而不适用于Git或Pact） ）|
| stubrunner.stream.enabled | true |是否启用Stub Runner与Spring Cloud Stream的集成. |
| stubrunner.stubs-mode | |选择存根应该来自哪里|
| stubrunner.stubs-per-consumer | false |此特定使用者的存根只能在HTTP服务器存根中注册. |
| stubrunner.username | |存储库用户名|
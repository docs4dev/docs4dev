## 42.寻址实例

应用程序的每个实例都有一个服务ID，其值可以使用 `spring.cloud.bus.id` 设置，其值应该是以冒号分隔的标识符列表，从最不具体到最具体.默认值是从环境构造为 `spring.application.name` 和 `server.port` （或 `spring.application.index` ，如果设置）的组合. ID的默认值以 `app:index:id` 的形式构造，其中：

-  `app` 是 `vcap.application.name` ，如果存在，或 `spring.application.name` 

-  `index` 是 `vcap.application.instance_index` ，如果存在， `spring.application.index` ， `local.server.port` ， `server.port` 或 `0` （按此顺序）.

-  `id` 是 `vcap.application.instance_id` （如果存在）或随机值.

HTTPendpoints接受“目标”路径参数，例如 `/bus-refresh/customers:9000` ，其中 `destination` 是服务ID.如果ID由总线上的实例拥有，它将处理该消息，并且所有其他实例都会忽略它.

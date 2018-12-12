## 42. Addressing an Instance

Each instance of the application has a service ID, whose value can be set with  `spring.cloud.bus.id`  and whose value is expected to be a colon-separated list of identifiers, in order from least specific to most specific. The default value is constructed from the environment as a combination of the  `spring.application.name`  and  `server.port`  (or  `spring.application.index` , if set). The default value of the ID is constructed in the form of  `app:index:id` , where:

-  `app`  is the  `vcap.application.name` , if it exists, or  `spring.application.name` 

-  `index`  is the  `vcap.application.instance_index` , if it exists,  `spring.application.index` ,  `local.server.port` ,  `server.port` , or  `0`  (in that order).

-  `id`  is the  `vcap.application.instance_id` , if it exists, or a random value.

The HTTP endpoints accept a “destination” path parameter, such as  `/bus-refresh/customers:9000` , where  `destination`  is a service ID. If the ID is owned by an instance on the bus, it processes the message, and all other instances ignore it.

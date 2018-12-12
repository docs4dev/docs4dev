## 43.解决服务的所有实例

“destination”参数在Spring  `PathMatcher` 中使用（路径分隔符为冒号 -   `:` ），以确定实例是否处理该消息.使用前面的示例， `/bus-env/customers:**` 定位“客户”服务的所有实例，而不管其余的服务ID.

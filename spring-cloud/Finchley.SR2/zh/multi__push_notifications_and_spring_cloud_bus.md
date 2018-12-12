## 9.推送通知和Spring Cloud Bus

许多源代码存储库提供程序（例如Github，Gitlab，Gitea，Gitee，Gogs或Bitbucket）通过webhook通知您存储库中的更改.您可以通过提供商的用户界面将webhook配置为URL以及您感兴趣的一组事件.例如，[Github](https://developer.github.com/v3/activity/events/types/#pushevent)使用POST给webhook，其中JSON正文包含一个提交列表和一个Headers（ `X-Github-Event` ）设置为 `push` .如果在 `spring-cloud-config-monitor` 库中添加依赖项并在Config Server中激活Spring Cloud Bus，则会启用 `/monitor` endpoints.

激活webhook后，Config Server会发送 `RefreshRemoteApplicationEvent` 针对它认为可能已更改的应用程序.可以制定变化检测策略.但是，默认情况下，它会查找与应用程序名称匹配的文件中的更改（例如， `foo.properties` 针对 `foo` 应用程序，而 `application.properties` 针对所有应用程序）.要覆盖行为时使用的策略是 `PropertyPathNotificationExtractor` ，它接受请求标头和正文作为参数，并返回已更改的文件路径列表.

默认配置与Github，Gitlab，Gitea，Gitee，Gogs或Bitbucket开箱即用.除了来自Github，Gitlab，Gitee或Bitbucket的JSON通知之外，您还可以通过使用 `path={name}` 模式中的表单编码正文参数POST到 `/monitor` 来触发更改通知.这样做会广播到匹配 `{name}` 模式的应用程序（可以包含通配符）.

> 仅当在配置服务器和客户端应用程序中激活 `spring-cloud-bus` 时才会传输 `RefreshRemoteApplicationEvent` .

> 默认配置还检测本地git存储库中的文件系统更改.在这种情况下，不使用webhook.但是，只要编辑配置文件，就会广播刷新.


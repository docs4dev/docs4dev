## 7.提供纯文本

您的应用程序可能需要根据其环境定制的通用纯文本配置文件，而不是使用 `Environment` 抽象（或YAML或属性格式中的其中一种替代表示）. Config Server通过 `/{name}/{profile}/{label}/{path}` 处的附加endpoints提供这些endpoints，其中 `name` ， `profile` 和 `label` 与常规环境endpoints具有相同的含义，但 `path` 是文件名（例如 `log.xml` ）.此endpoints的源文件的位置与环境endpoints的方式相同.相同的搜索路径用于属性和YAML文件.但是，不是聚合所有匹配的资源，而是仅返回要匹配的第一个.

找到资源后，通过使用提供的应用程序名称，配置文件和标签的有效 `Environment` 来解析正常格式（ `${…}` ）的占位符.通过这种方式，资源endpoints与环境endpoints紧密集成.请考虑以下GIT或SVN存储库示例：

```java
application.yml
nginx.conf
```

其中 `nginx.conf` 看起来像这样：

```java
server {
listen              80;
server_name         ${nginx.server.name};
}
```

和 `application.yml` 这样：

```java
nginx:
server:
name: example.com
---
spring:
profiles: development
nginx:
server:
name: develop.com
```

`/foo/default/master/nginx.conf` 资源可能如下：

```java
server {
listen              80;
server_name         example.com;
}
```

和 `/foo/development/master/nginx.conf` 这样：

```java
server {
listen              80;
server_name         develop.com;
}
```

> 与环境配置的源文件一样， `profile` 用于解析文件名.因此，如果您需要特定于配置文件的文件， `/*/development/*/logback.xml` 可以通过名为 `logback-development.xml` 的文件解析（优先于 `logback.xml` ）.

> 如果您不想提供 `label` 并让服务器使用默认标签，则可以提供 `useDefaultLabel` 请求参数.因此，前面的 `default` 配置文件示例可能是 `/foo/default/nginx.conf?useDefaultLabel` .


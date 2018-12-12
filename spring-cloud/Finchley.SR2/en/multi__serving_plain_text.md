## 7. Serving Plain Text

Instead of using the  `Environment`  abstraction (or one of the alternative representations of it in YAML or properties format), your applications might need generic plain-text configuration files that are tailored to their environment. The Config Server provides these through an additional endpoint at  `/{name}/{profile}/{label}/{path}` , where  `name` ,  `profile` , and  `label`  have the same meaning as the regular environment endpoint, but  `path`  is a file name (such as  `log.xml` ). The source files for this endpoint are located in the same way as for the environment endpoints. The same search path is used for properties and YAML files. However, instead of aggregating all matching resources, only the first one to match is returned.

After a resource is located, placeholders in the normal format ( `${…}` ) are resolved by using the effective  `Environment`  for the supplied application name, profile, and label. In this way, the resource endpoint is tightly integrated with the environment endpoints. Consider the following example for a GIT or SVN repository:

```java
application.yml
nginx.conf
```

where  `nginx.conf`  looks like this:

```java
server {
listen              80;
server_name         ${nginx.server.name};
}
```

and  `application.yml`  like this:

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

The  `/foo/default/master/nginx.conf`  resource might be as follows:

```java
server {
listen              80;
server_name         example.com;
}
```

and  `/foo/development/master/nginx.conf`  like this:

```java
server {
listen              80;
server_name         develop.com;
}
```

> As with the source files for environment configuration, the  `profile`  is used to resolve the file name. So, if you want a profile-specific file,  `/*/development/*/logback.xml`  can be resolved by a file called  `logback-development.xml`  (in preference to  `logback.xml` ).

> If you do not want to supply the  `label`  and let the server use the default label, you can supply a  `useDefaultLabel`  request parameter. So, the preceding example for the  `default`  profile could be  `/foo/default/nginx.conf?useDefaultLabel` .


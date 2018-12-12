## 55. Instrumentation

Spring Cloud Sleuth automatically instruments all your Spring applications, so you should not have to do anything to activate it. The instrumentation is added by using a variety of technologies according to the stack that is available. For example, for a servlet web application, we use a  `Filter` , and, for Spring Integration, we use  `ChannelInterceptors` .

You can customize the keys used in span tags. To limit the volume of span data, an HTTP request is, by default, tagged only with a handful of metadata, such as the status code, the host, and the URL. You can add request headers by configuring  `spring.sleuth.keys.http.headers`  (a list of header names).

> Tags are collected and exported only if there is a  `Sampler`  that allows it. By default, there is no such  `Sampler` , to ensure that there is no danger of accidentally collecting too much data without configuring something).


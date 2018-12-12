# Part II. Spring Cloud Config

**Finchley.SR2** 

Spring Cloud Config为分布式系统中的外部化配置提供服务器端和客户端支持.使用Config Server，您可以在所有环境中管理应用程序的外部属性.客户端和服务器上的概念与Spring  `Environment` 和 `PropertySource` 抽象相同，因此它们非常适合Spring应用程序，但可以与任何语言运行的任何应用程序一起使用.当应用程序通过部署管道从开发到测试再到生产环境时，您可以管理这些环境之间的配置，并确保应用程序具有迁移时需要运行的所有内容.服务器存储后端的默认实现使用git，因此它可以轻松支持配置环境的标签版本，并且可以访问各种用于管理内容的工具.添加替代实现并使用Spring配置插入它们很容易.


# Part XV. Spring Cloud Vault

©2016-2018原作者.

> 本文件的复印件可供您自己使用并分发给他人，前提是您不对此类复印件收取任何费用，并且进一步规定每份复印件均包含此版权声明，无论是以印刷品还是以电子方式分发.

Spring Cloud Vault Config为分布式系统中的外部化配置提供客户端支持.使用[HashiCorp’s Vault](https://www.vaultproject.io)，您可以在所有环境中为应用程序管理外部机密属性. Vault可以管理静态和动态机密，例如远程应用程序/资源的用户名/密码，并为外部服务提供凭据，如MySQL，PostgreSQL，Apache Cassandra，MongoDB，Consul，AWS等.


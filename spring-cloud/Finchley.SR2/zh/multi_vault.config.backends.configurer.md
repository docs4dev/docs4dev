## 106.配置PropertySourceLocator行为

Spring Cloud Vault使用基于属性的配置为通用和已发现的秘密后端创建 `PropertySource` s.

发现的后端提供 `VaultSecretBackendDescriptor`  beans来描述使用秘密后端的配置状态为 `PropertySource` .需要 `SecretBackendMetadataFactory` 来创建包含路径，名称和属性转换配置的 `SecretBackendMetadata` 对象.

`SecretBackendMetadata` 用于支持特定的 `PropertySource` .

您可以注册任意数量的bean实现 `VaultConfigurer` 进行自定义.如果Spring Cloud Vault发现至少一个 `VaultConfigurer`  bean，则禁用默认通用和已发现的后端注册.但是，您可以使用 `SecretBackendConfigurer.registerDefaultGenericSecretBackends()` 和 `SecretBackendConfigurer.registerDefaultDiscoveredSecretBackends()` 启用默认注册.

```java
public class CustomizationBean implements VaultConfigurer {

@Override
public void addSecretBackends(SecretBackendConfigurer configurer) {

configurer.add("secret/my-application");

configurer.registerDefaultGenericSecretBackends(false);
configurer.registerDefaultDiscoveredSecretBackends(true);
}
}
```

> 在引导上下文中需要进行所有自定义.在应用程序的 `org.springframework.cloud.bootstrap.BootstrapConfiguration` 处将配置类添加到 `META-INF/spring.factories` .


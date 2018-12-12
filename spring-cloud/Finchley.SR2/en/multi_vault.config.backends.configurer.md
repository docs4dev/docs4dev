## 106. Configure PropertySourceLocator behavior

Spring Cloud Vault uses property-based configuration to create  `PropertySource` s for generic and discovered secret backends.

Discovered backends provide  `VaultSecretBackendDescriptor`  beans to describe the configuration state to use secret backend as  `PropertySource` . A  `SecretBackendMetadataFactory`  is required to create a  `SecretBackendMetadata`  object which contains path, name and property transformation configuration.

`SecretBackendMetadata`  is used to back a particular  `PropertySource` .

You can register an arbitrary number of beans implementing  `VaultConfigurer`  for customization. Default generic and discovered backend registration is disabled if Spring Cloud Vault discovers at least one  `VaultConfigurer`  bean. You can however enable default registration with  `SecretBackendConfigurer.registerDefaultGenericSecretBackends()`  and  `SecretBackendConfigurer.registerDefaultDiscoveredSecretBackends()` .

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

> All customization is required to happen in the bootstrap context. Add your configuration classes to  `META-INF/spring.factories`  at  `org.springframework.cloud.bootstrap.BootstrapConfiguration`  in your application.


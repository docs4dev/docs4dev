## 110. Lease lifecycle management (renewal and revocation)

With every secret, Vault creates a lease: metadata containing information such as a time duration, renewability, and more.

Vault promises that the data will be valid for the given duration, or Time To Live (TTL). Once the lease is expired, Vault can revoke the data, and the consumer of the secret can no longer be certain that it is valid.

Spring Cloud Vault maintains a lease lifecycle beyond the creation of login tokens and secrets. That said, login tokens and secrets associated with a lease are scheduled for renewal just before the lease expires until terminal expiry. Application shutdown revokes obtained login tokens and renewable leases.

Secret service and database backends (such as MongoDB or MySQL) usually generate a renewable lease so generated credentials will be disabled on application shutdown.

> Static tokens are not renewed or revoked.

Lease renewal and revocation is enabled by default and can be disabled by setting  `spring.cloud.vault.config.lifecycle.enabled`  to  `false` . This is not recommended as leases can expire and Spring Cloud Vault cannot longer access Vault or services using generated credentials and valid credentials remain active after application shutdown.

```java
spring.cloud.vault:
config.lifecycle.enabled: true
```

See also: [Vault Documentation: Lease, Renew, and Revoke](https://www.vaultproject.io/docs/concepts/lease.html)

## 82.加密和解密

Spring Cloud CLI附带“加密”和“解密”命令.两者都以相同的形式接受带有指定为强制性“--key”的键的参数，例如

```java
$ spring encrypt mysecret --key foo
682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
$ spring decrypt --key foo 682bc583f4641835fa2db009355293665d2647dade3375c0ee201de2a49f7bda
mysecret
```

要在文件中使用密钥（例如，用于加密的RSA公钥），用“@”前面加上密钥值，并提供文件路径，例如，

```java
$ spring encrypt mysecret --key @${HOME}/.ssh/id_rsa.pub
AQAjPgt3eFZQXwt8tsHAVv/QHiY5sI2dRcR+...
```

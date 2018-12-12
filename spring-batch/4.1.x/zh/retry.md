## 1.重试

XML Java

为了使处理更加健壮且不易出现故障，有时可以自动重试失败的操作，以防后续尝试成功 . 易受间歇性故障影响的错误通常是短暂的 . 示例包括远程调用因网络故障或数据库更新中的 `DeadlockLoserDataAccessException` 而失败的Web服务 . 


### 1.1. RetryTemplate


> 

从2.2.0开始，重试功能从Spring Batch中撤出 . 它现在是新库的一部分，[Spring Retry](https://github.com/spring-projects/spring-retry) . 

要自动执行重试操作，Spring Batch具有 `RetryOperations` 策略 .   `RetryOperations` 的以下接口定义：


```java
public interface RetryOperations {

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback) throws E;

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback)
        throws E;

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RetryState retryState)
        throws E, ExhaustedRetryException;

    <T, E extends Throwable> T execute(RetryCallback<T, E> retryCallback, RecoveryCallback<T> recoveryCallback,
        RetryState retryState) throws E;

}
```


基本回调是一个简单的接口，允许您插入一些要重试的业务逻辑，如以下接口定义所示：


```java
public interface RetryCallback<T, E extends Throwable> {

    T doWithRetry(RetryContext context) throws E;

}
```


回调运行，如果失败（通过抛出 `Exception` ），它将被重试，直到它成功或实现中止 .   `RetryOperations` 接口中有许多重载的 `execute` 方法 . 当所有重试尝试都用完并处理重试状态时，这些方法处理各种用于恢复的用例，这允许客户端和实现在调用之间存储信息（我们将在本章后面详细介绍） . 

 `RetryOperations` 最简单的通用实现是 `RetryTemplate`  . 它可以使用如下：


```java
RetryTemplate template = new RetryTemplate();

TimeoutRetryPolicy policy = new TimeoutRetryPolicy();
policy.setTimeout(30000L);

template.setRetryPolicy(policy);

Foo result = template.execute(new RetryCallback<Foo>() {

    public Foo doWithRetry(RetryContext context) {
        // Do stuff that might fail, e.g. webservice operation
        return result;
    }

});
```


在前面的示例中，我们进行Web服务调用并将结果返回给用户 . 如果该调用失败，则重试该调用直到达到超时 . 


#### 1.1.1. RetryContext

 `RetryCallback` 的方法参数是 `RetryContext`  . 许多回调忽略了上下文，但是，如果需要，它可以用作属性包来存储迭代持续时间内的数据 . 

如果在同一个线程中正在进行嵌套重试，则 `RetryContext` 具有父上下文 . 父上下文偶尔用于存储需要在 `execute` 调用之间共享的数据 . 


#### 1.1.2. RecoveryCallback

重试耗尽时， `RetryOperations` 可以将控制权传递给另一个名为 `RecoveryCallback` 的回调 . 要使用此功能，客户端将回调一起传递给相同的方法，如以下示例所示：


```java
Foo foo = template.execute(new RetryCallback<Foo>() {
    public Foo doWithRetry(RetryContext context) {
        // business logic here
    },
  new RecoveryCallback<Foo>() {
    Foo recover(RetryContext context) throws Exception {
          // recover logic here
    }
});
```


如果在模板决定中止之前业务逻辑没有成功，则客户端有机会通过恢复回调进行一些备用处理 . 


#### 1.1.3.无状态重试

在最简单的情况下，重试只是一个while循环 .   `RetryTemplate` 可以继续尝试，直到它成功或失败 .   `RetryContext` 包含一些状态来确定是重试还是中止，但是这个状态在堆栈上并且不需要将它存储在全局任何地方，所以我们称之为无状态重试 . 无状态和有状态重试之间的区别包含在 `RetryPolicy` 的实现中（ `RetryTemplate` 可以处理两者） . 在无状态重试中，重试回调始终在失败时在其所在的同一线程中执行 . 


#### 1.1.4.有状态重试

如果失败导致事务资源变得无效，则需要考虑一些特殊情况 . 这不适用于简单的远程调用，因为没有事务性资源（通常），但它有时适用于数据库更新，尤其是在使用Hibernate时 . 在这种情况下，重新抛出立即调用失败的异常才有意义，这样事务就可以回滚，我们可以启动一个新的有效事务 . 

在涉及事务的情况下，无状态重试不够好，因为重新抛出和回滚必然涉及离开 `RetryOperations.execute()` 方法并可能丢失堆栈上的上下文 . 为了避免丢失它，我们必须引入一个存储策略来将其从堆栈中取出并将其（至少）放入堆存储中 . 为此，Spring Batch提供了一个名为 `RetryContextCache` 的存储策略，可以将其注入 `RetryTemplate`  . 使用简单的 `Map` ， `RetryContextCache` 的默认实现位于内存中 . 在集群环境中使用多个进程的高级用法也可能考虑使用某种类型的集群高速缓存实现 `RetryContextCache` （但是，即使在集群环境中，这可能也是过度的） . 

 `RetryOperations` 的部分责任是识别失败的操作，当它们返回新的执行时（通常包含在新的事务中） . 为了实现这一点，Spring Batch提供了 `RetryState` 抽象 . 这与 `RetryOperations` 接口中的特殊 `execute` 方法结合使用 . 

识别失败操作的方式是通过多次重试调用来识别状态 . 识别在状态下，用户可以提供 `RetryState` 对象，该对象负责返回标识该项目的唯一键 . 标识符用作 `RetryContextCache` 接口中的键 . 


> 

在 `RetryState` 返回的密钥中执行 `Object.equals()` 和 `Object.hashCode()` 时要非常小心 . 最好的建议是使用业务密钥来识别项目 . 对于JMS消息，可以使用消息ID . 

当重试耗尽时，还可以选择以不同的方式处理失败的项目，而不是调用 `RetryCallback` （现在假定可能失败） . 就像无状态情况一样，此选项由 `RecoveryCallback` 提供，可以通过将其传递给 `RetryOperations` 的 `execute` 方法来提供 . 

重试与否的决定实际上是委托给常规的 `RetryPolicy` ，因此可以在那里注入关于限制和超时的常见问题（本章稍后将对此进行介绍） . 


### 1.2.重试政策

在 `RetryTemplate` 内， `execute` 方法中重试或失败的决定由 `RetryPolicy` 确定， `RetryPolicy` 也是 `RetryContext` 的工厂 .   `RetryTemplate` 有责任使用当前策略创建 `RetryContext` 并在每次尝试时将其传递给 `RetryCallback`  . 回调失败后， `RetryTemplate` 必须调用 `RetryPolicy` 以要求它更新其状态（存储在 `RetryContext` 中），然后询问策略是否可以进行另一次尝试 . 如果无法进行另一次尝试（例如达到限制或检测到超时），则策略还负责处理耗尽状态 . 简单实现抛出 `RetryExhaustedException` ，这会导致回滚任何封闭事务 . 更复杂的实现可能会尝试采取一些恢复操作，在这种情况下，事务可以保持不变 . 


> 

故障本质上是可重试的或不可重试的 . 如果总是从业务逻辑中抛出相同的异常，那么重试它就没有用了 . 所以不要重试所有异常类型 . 相反，尝试只关注那些您希望可以重试的异常 . 通过更积极地重试业务逻辑通常不会有害，但这是浪费，因为如果失败是确定性的，那么您花时间重试事先知道的事情是致命的 . 

Spring Batch提供了一些简单的无状态 `RetryPolicy` 通用实现，例如 `SimpleRetryPolicy` 和 `TimeoutRetryPolicy` （在前面的例子中使用） . 

 `SimpleRetryPolicy` 允许在任何命名的异常类型列表上重试，最多固定次数 . 它还有一个永远不会重试的"fatal"异常列表，这个列表会覆盖可重试列表，以便它可以用来更好地控制重试行为，如下例所示：


```java
SimpleRetryPolicy policy = new SimpleRetryPolicy();
// Set the max retry attempts
policy.setMaxAttempts(5);
// Retry on all exceptions (this is the default)
policy.setRetryableExceptions(new Class[] {Exception.class});
// ... but never retry IllegalStateException
policy.setFatalExceptions(new Class[] {IllegalStateException.class});

// Use the policy...
RetryTemplate template = new RetryTemplate();
template.setRetryPolicy(policy);
template.execute(new RetryCallback<Foo>() {
    public Foo doWithRetry(RetryContext context) {
        // business logic here
    }
});
```


还有一个名为 `ExceptionClassifierRetryPolicy` 的更灵活的实现，它允许用户通过 `ExceptionClassifier` 抽象为任意一组异常类型配置不同的重试行为 . 该策略通过调用分类器将异常转换为委托 `RetryPolicy` 来工作 . 例如，通过将一个异常类型映射到不同的策略，可以在失败之前多次重试一个异常类型 . 

用户可能需要实施自己的重试策略以进行更多自定义决策 . 例如，当有一个众所周知的，特定于解决方案的异常分类为可重试且不可重试时，自定义重试策略是有意义的 . 


### 1.3.退避政策

在瞬态故障后重试时，通常会在再次尝试之前等待一段时间，因为通常故障是由某些问题引起的，只能通过等待来解决 . 如果 `RetryCallback` 失败， `RetryTemplate` 可以根据 `BackoffPolicy` 暂停执行 . 

以下代码显示 `BackOffPolicy` 接口的接口定义：


```java
public interface BackoffPolicy {

    BackOffContext start(RetryContext context);

    void backOff(BackOffContext backOffContext)
        throws BackOffInterruptedException;

}
```


 `BackoffPolicy` 可以以任何方式自由实现backOff .  Spring Batch开箱即用的策略都使用 `Object.wait()`  . 一个常见的用例是以指数级增加的等待时间进行退避，以避免两次重试进入锁定步骤并且都失败（这是从以太网中吸取的教训） . 为此，Spring Batch提供 `ExponentialBackoffPolicy`  . 


### 1.4.听众

通常，能够在多个不同的重试中接收针对横切关注点的额外回调是有用的 . 为此，Spring Batch提供 `RetryListener` 接口 .   `RetryTemplate` 允许用户注册 `RetryListeners` ，并且在迭代期间可以使用 `RetryContext` 和 `Throwable` 进行回调 . 

以下代码显示 `RetryListener` 的接口定义：


```java
public interface RetryListener {

    <T, E extends Throwable> boolean open(RetryContext context, RetryCallback<T, E> callback);

    <T, E extends Throwable> void onError(RetryContext context, RetryCallback<T, E> callback, Throwable throwable);

    <T, E extends Throwable> void close(RetryContext context, RetryCallback<T, E> callback, Throwable throwable);
}
```


在最简单的情况下， `open` 和 `close` 回调在整个重试之前和之后， `onError` 适用于各个 `RetryCallback` 调用 .   `close` 方法也可能会收到 `Throwable`  . 如果出现错误，则它是 `RetryCallback` 抛出的最后一个错误 . 

请注意，当有多个侦听器时，它们位于列表中，因此存在顺序 . 在这种情况下， `open` 以相同的顺序调用 `onError` 和 `close` 以相反顺序调用 . 


### 1.5.声明性重试

有时，有一些业务处理，您知道每次发生时都要重试 . 典型的例子是远程服务调用 .  Spring Batch提供了一个AOP拦截器，它为了这个目的在 `RetryOperations` 实现中包装了一个方法调用 .   `RetryOperationsInterceptor` 执行截获的方法，并根据提供的 `RetryTemplate` 中的 `RetryPolicy` 重试失败 . 

以下示例显示了一个声明性重试，它使用Spring AOP命名空间重试对名为 `remoteCall` 的方法的服务调用（有关如何配置AOP拦截器的更多详细信息，请参阅Spring用户指南）：


```xml
<aop:config>
    <aop:pointcut id="transactional"
        expression="execution(* com..*Service.remoteCall(..))" />
    <aop:advisor pointcut-ref="transactional"
        advice-ref="retryAdvice" order="-1"/>
</aop:config>

<bean id="retryAdvice"
    class="org.springframework.batch.retry.interceptor.RetryOperationsInterceptor"/>
```


以下示例显示了声明性重试，该声明性重试使用java配置重试对名为 `remoteCall` 的方法的服务调用（有关如何配置AOP拦截器的更多详细信息，请参阅Spring用户指南）：


```java
@Bean
public MyService myService() {
        ProxyFactory factory = new ProxyFactory(RepeatOperations.class.getClassLoader());
        factory.setInterfaces(MyService.class);
        factory.setTarget(new MyService());

        MyService service = (MyService) factory.getProxy();
        JdkRegexpMethodPointcut pointcut = new JdkRegexpMethodPointcut();
        pointcut.setPatterns(".*remoteCall.*");

        RetryOperationsInterceptor interceptor = new RetryOperationsInterceptor();

        ((Advised) service).addAdvisor(new DefaultPointcutAdvisor(pointcut, interceptor));

        return service;
}
```


前面的示例在拦截器中使用默认的 `RetryTemplate`  . 要更改策略或侦听器，可以将 `RetryTemplate` 的实例注入拦截器 .
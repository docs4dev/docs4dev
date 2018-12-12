## 1.重复

XML Java


### 1.1. RepeatTemplate

批处理是关于重复操作，可以是简单的优化，也可以是作业的一部分 . 为了制定策略并概括重复并为迭代器框架提供相应的数量，Spring Batch具有 `RepeatOperations` 接口 .   `RepeatOperations` 接口具有以下定义：


```java
public interface RepeatOperations {

    RepeatStatus iterate(RepeatCallback callback) throws RepeatException;

}
```


回调是一个接口，如下面的定义所示，它允许您插入一些要重复的业务逻辑：


```java
public interface RepeatCallback {

    RepeatStatus doInIteration(RepeatContext context) throws Exception;

}
```


重复执行回调，直到实现确定迭代应该结束 . 这些接口中的返回值是枚举，可以是 `RepeatStatus.CONTINUABLE` 或 `RepeatStatus.FINISHED`  .   `RepeatStatus` 枚举向重复操作的调用者传达关于是否还有其他工作要做的信息 . 一般来说， `RepeatOperations` 的实现应检查 `RepeatStatus` 并将其用作结束迭代的决策的一部分 . 任何希望向调用者发出信号表明没有更多工作要做的回调都可以返回 `RepeatStatus.FINISHED`  . 

 `RepeatOperations` 的最简单的通用实现是 `RepeatTemplate` ，如以下示例所示：


```java
RepeatTemplate template = new RepeatTemplate();

template.setCompletionPolicy(new SimpleCompletionPolicy(2));

template.iterate(new RepeatCallback() {

    public RepeatStatus doInIteration(RepeatContext context) {
        // Do stuff in batch...
        return RepeatStatus.CONTINUABLE;
    }

});
```


在前面的示例中，我们返回 `RepeatStatus.CONTINUABLE` ，以表明还有更多工作要做 . 回调也可以返回 `RepeatStatus.FINISHED` ，向呼叫者发出信号，表示没有其他工作要做 . 一些迭代可以通过回调中正在进行的工作固有的考虑来终止 . 就回调而言，其他的实际上是无限循环，并且完成决策被委托给外部策略，如前面示例中所示的情况 . 


#### 1.1.1. RepeatContext

 `RepeatCallback` 的方法参数是 `RepeatContext`  . 许多回调忽略了上下文 . 但是，如果需要，它可以用作属性包来存储迭代持续时间内的瞬态数据 . 返回 `iterate` 方法后，上下文不再存在 . 

如果正在进行嵌套迭代，则 `RepeatContext` 具有父上下文 . 父上下文偶尔用于存储需要在 `iterate` 的调用之间共享的数据 . 例如，如果您想计算迭代中事件的出现次数并在后续调用中记住它，就会出现这种情况 . 


#### 1.1.2. RepeatStatus

 `RepeatStatus` 是Spring Batch用于指示处理是否已完成的枚举 . 它有两个可能的 `RepeatStatus` 值，如下表所示：

||
|值|描述|
| ---- | ---- |
|可持续|那里还有更多工作要做 .  |
|完成|不应再发生重复 .  |

通过使用 `RepeatStatus` 中的 `and()` 方法， `RepeatStatus` 值也可以与逻辑AND运算组合使用 . 这样做的结果是在可持续标志上进行逻辑AND . 换句话说，如果任一状态为 `FINISHED` ，则结果为 `FINISHED`  . 


### 1.2.完成政策

在 `RepeatTemplate` 内， `iterate` 方法中的循环终止由 `CompletionPolicy` 确定， `CompletionPolicy` 也是 `RepeatContext` 的工厂 .   `RepeatTemplate` 有责任使用当前策略创建 `RepeatContext` 并在迭代的每个阶段将其传递给 `RepeatCallback`  . 在回调完成 `doInIteration` 之后， `RepeatTemplate` 必须调用 `CompletionPolicy` 以要求它更新其状态（将存储在 `RepeatContext` 中） . 然后，如果迭代完成，它会询问策略 . 

Spring Batch提供了 `CompletionPolicy` 的一些简单的通用实现 .   `SimpleCompletionPolicy` 允许执行最多固定次数（ `RepeatStatus.FINISHED` 强制在任何时候提前完成） . 

用户可能需要实施自己的完成策略以执行更复杂的决策 . 例如，在使用在线系统时阻止批处理作业执行的批处理窗口需要自定义策略 . 


### 1.3.异常处理

如果在 `RepeatCallback` 内部抛出异常，则 `RepeatTemplate` 会查询 `ExceptionHandler` ，它可以决定是否重新抛出异常 . 

以下清单显示了 `ExceptionHandler` 接口定义：


```java
public interface ExceptionHandler {

    void handleException(RepeatContext context, Throwable throwable)
        throws Throwable;

}
```


常见的用例是计算给定类型的异常数，并在达到限制时失败 . 为此，Spring Batch提供 `SimpleLimitExceptionHandler` 和稍微灵活的 `RethrowOnThresholdExceptionHandler`  .   `SimpleLimitExceptionHandler` 具有limit属性和应与当前异常进行比较的异常类型 . 还计算所提供类型的所有子类 . 在达到限制之前，将忽略给定类型的异常，然后重新抛出它们 . 其他类型的例外总是被重新抛出 . 

 `SimpleLimitExceptionHandler` 的一个重要的可选属性是名为 `useParent` 的布尔标志 . 默认情况下为 `false` ，因此限制仅在当前 `RepeatContext` 中计算 . 设置为 `true` 时，在嵌套迭代中（例如，步骤中的一组块），跨越同级上下文保持限制 . 


### 1.4.听众

通常，能够在多个不同的迭代中接收横切关注点的额外回调是有用的 . 为此，Spring Batch提供 `RepeatListener` 接口 .   `RepeatTemplate` 允许用户注册 `RepeatListener` 实现，并且在迭代期间可以使用 `RepeatContext` 和 `RepeatStatus` 给出回调 . 

 `RepeatListener` 接口具有以下定义：


```java
public interface RepeatListener {
    void before(RepeatContext context);
    void after(RepeatContext context, RepeatStatus result);
    void open(RepeatContext context);
    void onError(RepeatContext context, Throwable e);
    void close(RepeatContext context);
}
```


 `open` 和 `close` 回调在整个迭代之前和之后出现 .   `before` ， `after` 和 `onError` 适用于各个 `RepeatCallback` 次来电 . 

请注意，当有多个侦听器时，它们位于列表中，因此存在顺序 . 在这种情况下， `open` 和 `before` 以相同的顺序调用，而 `after` ， `onError` 和 `close` 以相反的顺序调用 . 


### 1.5.并行处理

 `RepeatOperations` 的实现不限于顺序执行回调 . 一些实现能够并行执行其回调非常重要 . 为此，Spring Batch提供 `TaskExecutorRepeatTemplate` ，它使用Spring  `TaskExecutor` 策略来运行 `RepeatCallback`  . 默认情况下使用 `SynchronousTaskExecutor` ，它具有在同一个线程中执行整个迭代的效果（与普通 `RepeatTemplate` 相同） . 


### 1.6.声明性迭代

有时会有一些业务处理，您知道每次发生时都要重复这些业务处理 . 典型的例子是消息管道的优化 . 处理一批消息（如果它们频繁到达）比为每条消息承担单独事务的成本更有效 .  Spring Batch提供了一个AOP拦截器，它为了这个目的而在 `RepeatOperations` 对象中包装一个方法调用 .   `RepeatOperationsInterceptor` 执行截获的方法，并根据提供的 `RepeatTemplate` 中的 `CompletionPolicy` 重复 . 

以下示例显示使用Spring AOP命名空间重复对名为 `processMessage` 的方法的服务调用的声明性迭代（有关如何配置AOP拦截器的更多详细信息，请参阅Spring用户指南）：


```xml
<aop:config>
    <aop:pointcut id="transactional"
        expression="execution(* com..*Service.processMessage(..))" />
    <aop:advisor pointcut-ref="transactional"
        advice-ref="retryAdvice" order="-1"/>
</aop:config>

<bean id="retryAdvice" class="org.spr...RepeatOperationsInterceptor"/>
```


以下示例演示如何使用java配置重复对名为 `processMessage` 的方法的服务调用（有关如何配置AOP拦截器的更多详细信息，请参阅Spring用户指南）：


```java
@Bean
public MyService myService() {
        ProxyFactory factory = new ProxyFactory(RepeatOperations.class.getClassLoader());
        factory.setInterfaces(MyService.class);
        factory.setTarget(new MyService());

        MyService service = (MyService) factory.getProxy();
        JdkRegexpMethodPointcut pointcut = new JdkRegexpMethodPointcut();
        pointcut.setPatterns(".*processMessage.*");

        RepeatOperationsInterceptor interceptor = new RepeatOperationsInterceptor();

        ((Advised) service).addAdvisor(new DefaultPointcutAdvisor(pointcut, interceptor));

        return service;
}
```


前面的示例在拦截器内使用默认的 `RepeatTemplate`  . 要更改策略，侦听器和其他详细信息，可以将 `RepeatTemplate` 的实例注入拦截器 . 

如果截获的方法返回 `void` ，则拦截器始终返回 `RepeatStatus.CONTINUABLE` （因此如果 `CompletionPolicy` 没有有限的终点，则存在无限循环的危险） . 否则，它返回 `RepeatStatus.CONTINUABLE` 直到截取方法的返回值为 `null` ，此时返回 `RepeatStatus.FINISHED`  . 因此，目标方法中的业务逻辑可以通过返回 `null` 或抛出由提供的 `RepeatTemplate` 中的 `ExceptionHandler` 重新抛出的异常来表示没有更多工作要做 .